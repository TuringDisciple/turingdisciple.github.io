---
layout: post
title: "TCP vs UDP, The Final Showdown, Part 2: TCP"
---

### Introduction
Continuing from my post [here](https://nashemncube.github.io/2017/10/27/udp-protocol.html) I'm going to fan favourite for this fight, TCP.
Like the first post, I intend to describe this topic from a high level to the best of my ability. I am not an authority so if I make any mistakes tell me, and I'll cry in the corner and fix them. LE' GO!

### So what is TCP
TCP compared to UDP is a network protocol which is reliable. Unlike in the case of UDP, TCP provides support for checking for duplicate packets, retransmission of lost packets, ordering of sent packets, and dealing with delay.

TCP first works by establishing a connection between two application programs, and maintains it through the entire communication until all messages have been exchanged. TCP also determines how to split data into the packets that will be transmitted over a connection. In the OSI model TCP lies partway between the transport and session layer, as is the case in UDP. Below is an image that shows TCP position within the OSI model.

<p align="center">
	<img src="/assets/osi-tcp.jpg">
</p>

I didn't go into much depth into the OSI (most likely will in a later) but if were to describe how TCP fits in the communication sequence between to application, we can use a simple example. 

In the case of the internet, if I wanted to access a webpage that request for that page will be processed in the application layer through most likely HTTTP. This request will then be sent to the TCP layer which will determine how to split up the data request into packets to be sent, numbers them accordingly. Packets will not necessarily take the same path through the network, but will reach the right destination. These packets then progress through the rest of the OSI stack to be processed. Below is an image of flow of information within the OSI model. Again, I will most likely cover this in more detail in later posts.

<p align="center">
	<img src="/assets/osi-flow.jpg">
</p>

**Side-note: I'm starting to realise describing networks will be easier if I start with the OSI model, the layers, and the protocols within them rather than specific protocols. Onwards and upwards**

So what does a TCP data packet look like.

<p align="center">
	<img src="/assets/tcp-packet.png">
</p>

Immediately we see the answer is beefier than a UDP datagram. The source port and destination port are self explanatory. As are checksum, TCP header length and data as we encountered them in the UDP protocol. So what's everything else in the TCP header:

- **Sequence number and Acknowledgement number:** In the TCP protocol, there's the concept of handshaking. This is used to establish a connectio between two connections, and is part of what makes TCP so reliable. The image shows an example of the handshake. Packets sent with the approriate flags set when determining this. Th

<p align="center">
	<img src="/assets/tcp-3-way-handshake.jpg">
</p>


- **NS, CWR, URG, ACK, PSH, RST, SYN, FIN:** All different flags which have a specific function. Can be better described [here](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#TCP_segment_structure) Similarly you can find better detail on the urgent pointer, window size and the specific role of the sequence and acknowledgement number.

Now some sweet, sweet code.

### tcp-server.py
{% highlight python %}
import socket

HOST_IP = '127.0.0.1' #Localhost
HOST_PORT = 10000

server_address = (HOST_IP, HOST_PORT)

# Creating the TCP/IP socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

print "Initialising server %s\nOn port %s\n" % server_address
s.bind(server_address)
s.listen(1)

while True:
    print "Waiting for connection"
    con, client_address = s.accept()
    try:
        print "Connection made from"

        # Read in the data 16 byte chunks, retransmit.
        while True:
            data = con.recv(16)
            print "Data recieved %s" % data
            if data: # Check data has been sent
                print "Sending data back: '%s'" % data
                con.sendall(data)
            else:
                print "Recieved no data from client"
                break
    finally:
        con.close()
{% endhighlight %}

### tcp-client.py
{% highlight python %}
import socket

HOST_IP = '127.0.0.1' #Localhost
HOST_PORT = 10000

host_address = (HOST_IP, HOST_PORT)

# Initializing a TCP/IP socket.
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Make connection to server
print "Connecting to server %s\n" % str(host_address)
s.connect(host_address)

try:
    m = "Hello world, foo bar baz. Mumbo jumbo jump"
    print "Sending message\n'%s'" % m
    s.sendall(m)

    # We are going to then check the server response.
    # In this case, that means checking the number of packets recieved
    data_recieved = 0
    data_to_recieve = len(m)

    while data_recieved < data_to_recieve:
        data = s.recv(16)
        data_recieved += len(data)
        print "Recieved %s\n" % data
        

finally:
    print "Socket closing \n"
    s.close()
{% endhighlight %}

Sample output

{% highlight bash %}
NashMacbook:tcp nashe$ python tcp-server.py 
Initialising server 127.0.0.1
On port 10000

Waiting for connection
Connection made from
Data recieved Hello world, foo
Sending data back: 'Hello world, foo'
Data recieved  bar baz. Mumbo 
Sending data back: ' bar baz. Mumbo '
Data recieved jumbo jump
Sending data back: 'jumbo jump'
Data recieved 
Recieved no data from client
Waiting for connection

....

NashMacbook:tcp nashe$ python tcp-client.py 
Connecting to server ('127.0.0.1', 10000)

Sending message
'Hello world, foo bar baz. Mumbo jumbo jump'
Recieved Hello world, foo

Recieved  bar baz. Mumbo 

Recieved jumbo jump

Socket closing 
{% endhighlight %}

And we are done. 


### Conclusion 

I think it is obvious who the contender in this battle is, TCP wins it all. 

![Gif failed](https://media.giphy.com/media/aWRWTF27ilPzy/giphy.gif)

Next time I'll probably tackle the OSI model. Hope you enjoyed

~Nashe


