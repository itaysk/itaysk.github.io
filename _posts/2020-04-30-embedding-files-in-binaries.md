---
title: Embedding files in binaries
date: 2020-04-30 21:45
categories: [Technical-Howto]
tags: [go, linux]
---

When building [Tracee](https://github.com/aquasecurity/tracee), I wanted to embed the ebpf program file into the compiled binary artifact. This file is essentially an asset that is required by the program, which will look for it at runtime in a predefined path. I wanted to simplify the distribution of Tracee, and ship a single binary that includes this asset file as an embedded resource.  
I have researched a few options for embedding embedding resources into a standalone binary artifact and wanted to share my learnings here. I will not attempt to compare tools or recommend any specific one, but just outline the technical approaches for embedding resources in binaries. 

Note that this was done in the context of Tracee, which is based on Go and Linux, so the techniques are very specific to my use case. However, I think once you understand the general approaches, you can find comparable implementations of the same ideas for your environments as well.

## Generating source file with hardcoded data

This is the most common approach I found. There are many tools that implement this for Go.  
A tool will create `.go` source files that include the serialized contents of the files to embed. For example:

```go
var file string = "The contents of the file"
```

or more realistically: 

```go
var file []byte = []byte{0x54, 0x68, 0x65, 0x20, 0x63, 0x6f, 0x6e, 0x74, 0x65, 0x6e, 0x74, 0x73, 0x20, 0x6f, 0x66, 0x20, 0x74, 0x68, 0x65, 0x20, 0x66, 0x69, 0x6c, 0x65}
```

The high level workflow is:
1. You run the tool before build.
2. The tools generate source files with the contents of the files to embed.
3. You build your code which now includes the generated files.
4. At runtime your program has access to the file's contents because it was compiled with your code. 

Usually the tool will also include some convenient API to discover, access, and decode the embedded data.

For educational purposes, here is a nice article walking through how to create such tool from scratch: [The easiest way to embed static files into a binary file in your Golang app (no external dependencies)
](https://dev.to/koddr/the-easiest-way-to-embed-static-files-into-a-binary-file-in-your-golang-app-no-external-dependencies-43pc)

'go-bindata' used to be a popular tool for this approach, but was abandoned, and was since forked many times in attempts to resurrect it. If you [search GitHub for `go-bindata` Go repositories](https://github.com/search?l=Go&q=go-bindata&type=Repositories), you'll see what I mean. I think that this fork is the modern reincarnation: [go-bindata/go-bindata](https://github.com/go-bindata/go-bindata).

There's an issue in the Go GitHub about upstreaming some implementation of this approach. The GitHub Issue also lists some good tools that you can check out. [proposal: cmd/go: support embedding static assets (files) in binaries #35950](https://github.com/golang/go/issues/35950)

### go generate

If you've read so far and think this sounds familiar, you are right. Go has standard code generation facility that we can and should use when generating code. Here is a quick refresher: [go generate](https://medium.com/@ehrt74/go-generate-89b20a27f7f9).

I just wanted to show off how simple it can be to embed a file using a `go generate` one-liner. Do not use this example in your application - be inspired by it. If you liked it, look for a robust tool to generate code for you (discussed previously) and then integrate it with go generate as described in the [Integration](#Integration) section :

{% gist itaysk/f0a05b6cc8685721fd9ada9efa467553 %}

## Go tool linker flags

Another Go-native option that I did not see mentioned at all, is to use the `-X` Go linker flag that can inject string data into your binary. This option is commonly used to inject the version information into the final binary, so that your application will know what version it is, without you having to hardcode it in source and update it.

You can pass linker flags through the `go build` command using the `-ldflags` flag. To read more on go linker flags, including the `-X` flag, see here: [Command link](https://golang.org/cmd/link/).

Here is an example of this approach:

{% gist itaysk/26d87458a5f0530449c26fa2ccc33567 %}

## Add ELF section

ELF is the binary executable format used in Linux. Without diving into it's specification, we know it is composed of sections that describe pieces of information about the executable, and that you can also include your own extra sections. There are tools that can help you do this - check out the [GNU Binutils](https://www.gnu.org/software/binutils/) project.  
In this approach you:

1. Build the Go binary as usual.
2. Add a custom ELF section to the binary with your file data.
3. When executed, the OS will just ignore your custom section.
4. At runtime your code can look for the section and extract the data from it.

Here is an example of this approach:

{% gist itaysk/993329ef49a4fee26fbb57be6bc28e32 %}

## ELF concatenation

It turns out that the ELF spec does not care if you add extra raw data *after* the end of the ELF data. This is different from the previous approach since here the ELF is unaware of our data we have appended to it. 
One could just concatenate the binary information to the ELF binary executable using something as simple as `cat executable file >newbinary`), and then the application code can read the data appendix at runtime. I stopped here because this started to feel more like a CTF challenge then a recommendable way to embed data.

## Self extracting archive

The previous experiment with ELF concatenation has led me to this about self-extracting archives (SFX). A SFX is a compressed archive of your app+data, attached to a small program that just extracts it and executes your app. To me, researching this approach brought up nostalgia to the old school Windows days. This approach is not technically embedding data in the binary, but wrapping the entire application in a new executable. I'm still mentioning it here for learning purposes.  
Technically this is made possible since the compression format (e.g ZIP) can be prepended to, while the executable format can be appended to (as we discussed before). So if we concatenate for example an ELF with a ZIP it happens to be both a valid executable, and a valid compressed archive!

One of the Go native embedding tools that I looked at - [GeertJohan/go.rice](https://github.com/GeertJohan/go.rice) offered this approach, but since this approach works at the files level and not on source level, it doesn't have to be a Go oriented tool so you can use any SFX packager with your Go application, for example:[megastep/makeself](https://github.com/megastep/makeself).

## Integration

So far we have discussed several approaches to embed data in your binary. After you choose one, and find the tools to help you implement it, you probably want to integrate the solution into your build workflow. Let's consider some options:

### Makefile
Makefile is the classic way to automate build tools. A simple make build target can look something like:

```makefile
build:
  mytool-before # e.g. tool that generate file
  go build -myflag myvalue # e.g. linked flags
  mytool-after # e.g. tool that manipulates the binary
```

### GoReleaser
[GoReleaser](https://goreleaser.com/) is a popular tool to build and release Go projects. It is has some customization options that you can use like passing custom flags to the go build command, setting environment variables, calling tools before and after the build, etc.

```yaml
builds:
  - ldflags: "-X main.myfile={.Env.MY_FILE_CONTENTS}}" e.g linker flag
    hooks:
      pre: mytool-before # e.g. tool that generate file
      post: mytool-after # e.g. tool that manipulates the binary
```

### Go generate

We mentioned go generate in previous example but it's worth mentioning again here because go generate is just a calling convention. Any approach that relies on generating source files can be integrated using go generate.  
In your application Go source code include a comment like:

```go
//go:generate tool-generates-code
```

Then, when you run the `go generate` command, it will invoke the tool for you. You can run `go generate` in your makefile, or goreleaser pre hook for example. The benefit of using go generate over calling the tool directly is that it's a universal and consistent way that is common and familiar with Go developers. In addition, having the generation command in the source code, alongside the code that relies on it makes it easier to maintain and is self-documented. When I reviewed tools in the code generation category it was rare to see anyone that advise or demonstrate the use with `go generate`, which is unfortunate in my opinion.

### Custom build command
Some tools (e.g [packr](https://github.com/gobuffalo/packr)) strayed further from conventions by promoting you replace your entire Go build with their own tool which wraps `go build` and adds code generation. Personally I can't relate to this approach but luckily you can also use these tools in "regular" mode where you choose how to integrate them into your workflow.