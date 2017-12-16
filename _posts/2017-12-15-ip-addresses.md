---
layout: post
title: "Soft Intro to IP addressing"
---

### Introduction
In this post (long overdue), I am going to go over the basics of IP addressing, what IPV4 is, IPV6, IP address classes and subnetting. So let the fun begin!

### IP Addresses: What is?
Most likely if you're reading this post, you've heard of the term "IP address". But what does that mean? Well assuming that you were curious enough, you would be able to screen that 

>" IP addresses are numerical values, given to devices within a network, that are used within the internet protocol"

And that pretty much sums it up. Looks like we're done here....


Ok maybe we should go a bit more in depth into this idea. So first question to ask, based on this basic explanation, what does this number look like?

Well an example IP address would be '127.0.0.1'(in fact this address is a reserved address called the [loopback address](https://superuser.com/a/255839))
. The number actually would look more like this to a computer
'01111111000000000000000000000001'. IP addresses are actually 32 bit values split into 4 octets for readibility(128 bit value in IPv6). That means each octet takes values between 0-255 if you know anything about binary.

So networks use these addresses to map to devices. And in a local network, that seems simple enough. A few things to note. IPv4 only gives us 2^32 addresses (over 4 billion). In the early days of the internet, this was more than enough as there was only a few devices anyways. But [looking here](https://www.statista.com/statistics/471264/iot-number-of-connected-devices-worldwide/), that's peanuts. So with this in mind, how come we haven't run out of addresses. Well we have technically, IP addresses are resused all the time!!!!! That's insane, there must be a reason that networks don't get confused and I get access to information that is meant for me. And there is, in particular there's more going on logically that maps IP addresses to devices. But first...

### Subnet Masks and IP address classes
IP addresses belong to classes based on the first value of their octet. Below is an image showing the different IP address classes

<p align="center">
	<img src="/assets/IP-address-classes.jpg"/>
</p>

Notice how 127 is omitted as this is used for loopback. Classes A, B and C are used by network adminstrators but D and E are reserved. But these are class definitions, a more important thing is relating this to the concept of subnetting. 

<p align="center">
	<img src="/assets/subnetting.jpg"/>
</p>

What does the diagram mean? Essentially a subnet mask is a number of bits (can be variable as we'll see later) that tell us which part of an IP address indicates the network we're on and which part indicates the host, within that network, we wish to connect to.

This is used using boolean logic. So for example.

Suppose we have a class B device, with subnet mask 255.255.0.0 and IP address 128.1.8.0
If we showed these number in binary we'd get

SM = 11111111.11111111.00000000.00000000

IP = 10000000.00000001.00001000.00000000

When we perform a boolean and operation between these two we get

SM & IP = 10000000.00000001.0000000.00000000 == 128.1.0.0

This value indicates the network which we are on. If we want to obtain the host, we look at the bits indicated by the below logic

!SM & IP = 0.0.8.0

Thinking about this further we see that in a class B network, with subnet mask 255.255.0.0, their are only 16,384 different networks available (2^16) and within each of those networks there can only be 65,536 different hosts.

Similarly in a class A network with subnet mask 255.0.0.0. There are 127 different networks available, band 2^24 different devices that can be within those networks.


### CIDR
Now the above class system is obsolete. The reasons for this is that as the internet expanded, this sort of class of system failed to scale. The class system of the address space was replaces with classless inter domain routing (or CIDR for short). There's quite a lot to say on CIDR so I suggest reading more [here](https://whatismyipaddress.com/cidr)

### Private vs Public 
Not all devices need to be connected on the internet, so that is why private IP addressing exists. In a case where all devices on a network need to only communicate within that network, you would use a private IP address space to do this. There are 3 reserved IPv4 network ranges. Below is a figure comparing private and public IP addresses.

<p align="center">
	<img src="/assets/public-private-ip.png"/>
</p>


### IPv6
So we've come this far and we have only been dealing with IPv4. So what is IPv6? Well this is a different version of the IP. One of the main differences is that IPv6 provides 128 bit IP addresses. This means 2^128 different IP addresses are available. This is seen as enough to support us until the forseeable future, but IPv6 is still slow in it's uptake. More on IPv6 is [here](https://en.wikipedia.org/wiki/IPv6).I suggest reading up, there's a lot going on there.

### MAC addresses
Another type of address that is referenced is called a MAC address. These are the addresses of the network interface cards within devices. They are completely unique to each device worldwide and are written in hexadecimal format. More [here](https://en.wikipedia.org/wiki/MAC_address)


### Conclusion
I hope all this was useful, I certianly enjoyed writing this long overdue post. 

~Nashe

