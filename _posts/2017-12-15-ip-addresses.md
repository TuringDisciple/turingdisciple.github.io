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

