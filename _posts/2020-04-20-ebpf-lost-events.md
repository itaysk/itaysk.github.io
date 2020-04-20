---
title: eBPF lost events
date: 2020-04-20 16:45
categories: [Technical-Howto]
tags: [linux, ebpf, go]
---

Lately I've been working on re-writing [Tracee](https://github.com/aquasecurity/tracee) from Python to Go. Tracee is a system tracing tool based on eBPF, a Linux technology that we used for collecting the events in the kernel (for an introduction to eBPF see [here]()). Tracee is using the [bcc](https://github.com/iovisor/bcc/) project to easily work with eBPF.  
During the migration from Python to Go, we moved from using bcc's python library to the [gobpf](https://github.com/iovisor/gobpf/) library, which was supposed to offer similar functionality to the bcc's Python library. However, during the migration we have noticed a missing functionality in the gobpf library - the ability to handle "lost events". In this post we'll discuss what exactly are those lost events and how we implemented this missing feature in gobpf.

## perf ring buffer

Before we continue, we need to understand how events we collected with eBPF make their way to our program. eBPF runs in the kernel, and there are several mechanisms to communicate information between kernel-space and users-pace but the most common for eBPF, and what 'bcc' is recommending, is the 'perf' subsystem.
perf predates eBPF, and have been used to collect performance measurements from the OS (and from hardware) for a long time. In order to communicate performance data, perf offers a mechanism based on a 'ring buffer'. 

![ring buffer](/images/2020-04-20-ebpf-lost-events_1.png)

A ring buffer (a.k.a circular buffer) is a contiguous memory area that the producer and the consumer can read and write to simultaneously. It's called "ring" because once the buffer has filled, the producer will continue to write data at the beginning of the buffer, so a fixed sized ring buffer could potentially accommodate an infinite stream of events. But what happens when the buffer is filled and the consumer has not (yet) read it's data? Depending on the implementation, the producer will overwrite existing data, or lose the event.

In order to understand what happens in bcc, we will track the flow of code from the lines we wrote in bcc, down the layers of abstraction to the perf ring buffer.

## Tracking the code

In your Python code, you are instructed to instantiate a `BPF` class and:
1. Initialize buffer using the `BPF.open_perf_buffer` method
1. Start receiving events using `BPF.perf_buffer_poll` method

### Section 1 - BPF.open_perf_buffer

1. The user program is opening a perf buffer using the `BPF.open_perf_buffer` method which receives a `lost_cb` callback (function pointer): [(source)](https://github.com/iovisor/bcc/blob/98c18b640117b10f923c8697be8bfff8ad39834b/src/python/bcc/table.py#L670)
```python
 def open_perf_buffer(self, callback, page_cnt=8, lost_cb=None)
```
1. `BPF.open_perf_buffer` ends up creating a "reader" using the `perf_reader_new` C function. "reader" is a bcc construct that facilitates reading from a buffer. Appendix 1 walks through this code path.
1. `perf_reader_new` C function is saving the callback in the newly created reader: [(source)](https://github.com/iovisor/bcc/blob/4d61a57b4ebd8b387abe3270609674e57e334148/src/cc/perf_reader.c#L61)
```c
reader->lost_cb = lost_cb;
```

This shows us what the system does with the lost events callback that we provided - it was just registered with the appropriate reader. When is this callback called? Let's move on to look at the other method we mentioned.

### Section 2 - BPF.perf_buffer_poll

1. The user program is calling `BPF.perf_buffer_poll` method to start receiving events. This is using bcc's C function `perf_reader_poll` to read from the previously created "reader": [(source)](https://github.com/iovisor/bcc/blob/2fa54c0bd388898fdda58f30dcfe5a68d6715efc/src/python/bcc/__init__.py#L1392)
```python
lib.perf_reader_poll(len(readers), readers, timeout)
```
1. `perf_reader_poll` is invoking the read function on every reader: [(source)](https://github.com/iovisor/bcc/blob/4d61a57b4ebd8b387abe3270609674e57e334148/src/cc/perf_reader.c#L234)
```c
perf_reader_event_read(readers[i]);
```
1. `perf_reader_event_read` is reading an event. If it's type is `PERF_RECORD_LOST`, it will call our lost events callback: [(source)](https://github.com/iovisor/bcc/blob/4d61a57b4ebd8b387abe3270609674e57e334148/src/cc/perf_reader.c#L205)
```c
if (e->type == PERF_RECORD_LOST) { 
  ... 
  reader->lost_cb(reader->cb_cookie, lost); 
  ... 
}
```
 
So now we know that our lost events callback was triggered when bcc found events of type `PERF_RECORD_LOST`. But we never submitted events of this type. All we did in our eBPF program was use the `perf_submit` function. How did those event got there? Let's look at what happens when we submit events.

### Section 3 - perf_submit
To submit events from our eBPF (C) program, we are instructed to initialize a "table" using the `BPF_PERF_OUTPUT` macro, and then call the `perf_submit` bcc C helper function.

1. The user eBPF program is using `BPF_PERF_OUTPUT` to define a struct. The created struct holds a pointer to the `perf_submit` function: [(source:)](https://github.com/iovisor/bcc/blob/2fa54c0bd388898fdda58f30dcfe5a68d6715efc/src/cc/export/helpers.h#L137)
1. The user eBPF program calls `table.perf_submit()` to submit an event
1. `bpf_perf_event_output` ends up calling `perf_event_output` function from the perf subsystem. Appendix 2 walks through this code path.
1. `perf_event_output` calls `perf_output_begin` function before it actually submits an event. [(source)](https://elixir.bootlin.com/linux/v4.5/source/kernel/events/core.c#L5608)
1. `perf_output_begin` kernel function is the one that creates the "lost events": [(source)](https://elixir.bootlin.com/linux/v4.5/source/kernel/events/ring_buffer.c#L184)
```c
struct {
  struct perf_event_header header;
  u64			 id;
  u64			 lost;
} lost_event;

if (unlikely(have_lost)) 
{ 
  ... 
  lost_event.header.type = PERF_RECORD_LOST; 
  ... 
}
``` 

What is this `have_lost` indicator? Let's dig (final stretch, bear with me):

### Section 4 - Tracking the ring buffer's `have_lost` indicator

If we look at the `perf_output_begin` function from the kernel's perf ring buffer implementation:
1. `have_lost` variable is holding the ring buffer's `lost` field: [(source)](https://elixir.bootlin.com/linux/v4.5/source/kernel/events/ring_buffer.c#L134)
```c
have_lost = local_read(&rb->lost);
```
1. There's a check if there's enough space in the ring buffer (and also the buffer is configured to not overwrite), then we go to `fail`: [(source)](https://elixir.bootlin.com/linux/v4.5/source/kernel/events/ring_buffer.c#L147)
```c
if (!rb->overwrite && unlikely(CIRC_SPACE(head, tail, perf_data_size(rb)) < size))
  goto fail;
``` 
1. Under `fail`, `rb->lost` is being incremented: [(source)](https://elixir.bootlin.com/linux/v4.5/source/kernel/events/ring_buffer.c#L198)
```c
fail: 
  local_inc(&rb->lost);
``` 

That's it! we have found out that "lost events" are events that couldn't be written to the perf ring buffer because there wasn't enough space. When this happens, the bcc "reader" will call the callback that was given to it when it was created.

To recap:

![lost events depiction](/images/2020-04-20-ebpf-lost-events_2.png)

You can follow along the dotted line from end to start to visualize the logical chain.

## Lost events in gobpf

Now that we know what are lost events, let's get back to our original problem that gobpf didn't ask us for a lost callback. Here's the fix that I submitted: [https://github.com/iovisor/gobpf/pull/235](https://github.com/iovisor/gobpf/pull/235). Let's review it:

The change is contained in the `bcc/perf.go` file. This time I will not go through every single step because this change is more readable. The highlights are:
1. change `callbackData` struct to contain a lost channel in addition to the main channel:
```go
callbackDataIndex := registerCallback(&callbackData{
		receiverChan,
		lostChan,
	})
```
2. change the signature of the `InitPerfMap` user facing function making it also accept a channel for lost events:
```go
func InitPerfMap(table *Table, receiverChan chan []byte, lostChan chan uint64) (*PerfMap, error) {
```
3. in the call to the lower level bcc C function `bpf_open_perf_buffer`, pass the registered lost callback:
```go
reader, err := C.bpf_open_perf_buffer(
		(C.perf_reader_raw_cb)(unsafe.Pointer(C.rawCallback)),
		(C.perf_reader_lost_cb)(unsafe.Pointer(C.lostCallback)),
		unsafe.Pointer(uintptr(callbackDataIndex)),
		-1, cpuC, pageCntC)
```

## Further reading:
- [Linux: circular-buffers]https://www.kernel.org/doc/Documentation/circular-buffers.txt


# Appendix

## Appendix 1 - from BPF.open_perf_buffer to perf_reader_new
1.  `BPF.open_perf_buffer` method is calling into bcc's C function `lib.bpf_open_perf_buffer` [(source)](https://github.com/iovisor/bcc/blob/98c18b640117b10f923c8697be8bfff8ad39834b/src/python/bcc/table.py#L705)
1. `bpf_open_perf_buffer` function is creating a reader using `perf_reader_new` function [(source)](https://github.com/iovisor/bcc/blob/2fa54c0bd388898fdda58f30dcfe5a68d6715efc/src/cc/libbpf.c#L1192)

## Appendix 2 - from bpf_perf_event_output to perf_event_output
1. `table.perf_submit()` function is converted to `bpf_perf_event_output()` [(source)](`https://sourcegraph.com/github.com/iovisor/bcc@454b138e6b75a47d4070a4f99c8f2372b383f71e/-/blob/src/cc/frontends/clang/b_frontend_action.cc#L844:15`)
1. `bpf_perf_event_output` is implemented by `BPF_FUNC_perf_event_output` [(source)](https://github.com/iovisor/bcc/blob/2fa54c0bd388898fdda58f30dcfe5a68d6715efc/src/cc/export/helpers.h#L405)
1. `BPF_FUNC_perf_event_output` is an eBPF helper: [(source)](https://elixir.bootlin.com/linux/v4.5/source/include/uapi/linux/bpf.h#L271)
1. `BPF_FUNC_perf_event_output` is creating the `bpf_perf_event_output` prototype: `bpf_perf_event_output_proto`: [(source)](https://elixir.bootlin.com/linux/v4.5/source/kernel/trace/bpf_trace.c#L300)
1. `bpf_perf_event_output_proto` is pointing to the `bpf_perf_event_output` function [(source)](https://elixir.bootlin.com/linux/v4.5/source/kernel/trace/bpf_trace.c#L262)
1.  `bpf_perf_event_output` function is calling the `perf_event_output` function [(source)](https://elixir.bootlin.com/linux/v4.5/source/kernel/trace/bpf_trace.c#L258)
