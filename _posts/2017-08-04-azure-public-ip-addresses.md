---
title: Azure Public IP Addresses
date: 2017-08-04 13:42
categories: [Technical-Howto]
tags: [azure-network, ftp]
---

Ever created a VM in Azure and noticed that the VM only knows about its private IP address even though it might have a public IP as well?  
This behavior is actually by design, which might surprise people with traditional networking background.

The VM in Azure should be oblivious of its public IP. The public IP is injected to traffic traveling outside the VNet by Azure. You can conceptually think of this as a transparent NAT between your VM and the Internet, translating between private and public IPs. For the sake of simplicity I'll call this translation layer - 'Azure hidden NLB'.

Consider the following scenario: A client with private IP 102.168.7.4 is talking to a server under 40.68.244.64.

Client:
![client](/images/2017-08-04-azure-public-ip-addresses_1.png)

Server:
![server](/images/2017-08-04-azure-public-ip-addresses_2.png)

What's going on here?  
The client sends packets from 192.168.7.4 (it's own IP) to the server under 40.68.244.64 (server's public IP), but from the server's point of view, it sees packets arriving from 52.233.140.16 (the client's public IP) destined into 172.16.1.4 (it's own private IP). Azure hidden NLB has mangled with the traffic so that it will all work seamlesly.

One thing to keep in mind is that Azure knows which public IP is associated with which NIC, and when the outbound packet hits the Azure hidden NLB, the IP for translation must match the NIC to IP configuration in Azure. (following there's an example for this)

You can read more on the subject here: [https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections)

## Why is this important to know?
Here are some example where this behavior has actually hit me:

### Story 1: iptables forwarding
I had a scenario where I had a VM that does simple packet forwarding using iptables.  So packets were going through this middleware routing VM and leaving its NIC with the source IP of the client VM that originated the packet.  

![forwarding worked](/images/2017-08-04-azure-public-ip-addresses_3.png)

This worked fine as long as we were using private IPs, but as soon as we tried to talk to the server using its public IP, or talk to a different server that's outside of Azure, it stopped working.

![forwarding worked](/images/2017-08-04-azure-public-ip-addresses_4.png)

The reason is that caveat I mentioned before about matching the NIC as well.  
When we used a public IP, traffic started leaving the VNET and going through the Azure hidden NLB. Since packets were leaving the routing VM's NIC with a source IP that didn't match it's private IP, the Azure hidden NLB got confused and didn't translate to public IP, so traffic didn't reach it's destination.
When using private IPs, packets were not leaving the VNET so didn't even go through this Azure hidden NLB and therefore it worked.

### Story 2: FTP server
Another scenario where this knowledge was useful was where I had an environment with an FTP server in Azure.  
You might know about FTP flow in [Passive mode](https://stackoverflow.com/questions/1699145/what-is-the-difference-between-active-and-passive-ftp) which advertises the server's address to the client, so that the client will initiate the connection.   
Which address the FTP server might advertise? Well, it only knows about its private address, and that's what it will use by default. This of course will break the FTP transfer as the packet will be unrouteable outside the VNET.  
A simple fix was to override the advertised address to the public one using the FTP software configuration settings.

### Monitoring
Obviously this behavior makes any kind of tracing and monitoring network traffic un-trevial, as apparent from the TCP dump I showed earlier. It's not impossible, you just need to be aware of how things work.

### Firewalling
Another point to consider is if you'd expect external traffic arriving at a designated NIC so that you can apply filtering rules on it. The simple workaround is to apply the rules on the same single NIC but rewrite their matching logic to be more specific to catch only external traffic. Alternatively you could have multiple NICs.



(I'd like to thank [Shay Shahak](https://www.linkedin.com/in/shay-shahak-35a46213/) for helping me write this post)