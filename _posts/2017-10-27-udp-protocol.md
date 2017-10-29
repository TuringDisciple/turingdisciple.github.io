---
layout: post
title: "TCP vs UDP, The Final Showdown, Part 1: UDP"
---

### Introduction
In this post I am going to discuss the UDP protocol, it's use. This is the first part of a series in which I basically discuss TCP and UDP and their differences. 
### Definitions
These definitions are really high level, for a more in depth understanding I'd suggest google. My favourite book on this stuff is [Computer Networking: A top down approach by Kurose & Ross](https://www.amazon.co.uk/Computer-Networking-Top-Down-James-Kurose/dp/0136079679)

- OSI(open systems interconnection) model: A high level model used for describing the telecommunication system and it's function without making reference to the technology involved or structure within.
<p align="center">
	<img src="/assets/osi-model.jpg" alt="OSI model"> 
</p>
- Datagram: Split into two components, header and data payload, datagrams are an entity of data which is used in information transfer in networks. The header of the datagram usually carries information about the source and destination of the data payload. Datagrams don't have a predetermined path and are therefore considered unreliable but are useful in that there's no need for "switches" within a network to have any prior information about the datagram. Example datagram below: 
<p align="center">
	<img src="/assets/datagram-example.jpg" alt="Example datagram">
</p>
- Packets: A packet is similar to a datagram in terms of definition but the difference is mainly that in a reliable communication (TCP/IP) the term packet is used, but in unreliable communication we use the term datagram(UDP/IP). Reliability in this context will be discussed in the differences between the two below.
- IP(internet protocol): Protocol tasked with delivering packets within packet switched networks, only knowing the IP address within the packet header.
- Port: A communication endpoint identifying a specific process or network service. Some ports are reserved by convention. Usually indicated by a port number in reference to a destination or source network address e.g ip 1.2.3.4 at port 80. 

### UDP
UDP stands for the user datagram protocol. It is a transport layer protocol which is used with the IP network protocol(UDP/IP).The protocol doesn't establish an end-to-end connection with two systems which are communication. UDP datagrams are relatively small and are transferred in single IP packets with an 8 byte header and a payload theoretical limit of 65,527(65535-8). bytes as IP packets are. This may be reduced as IP can take up 20 bytes in the header with it's own header information. Below is an example of a UDP datagram

<p align="center">
	<img src="/assets/udp-datagram.png">
</p>

The header in this example contains the source port, destination port, UDP length and UDP checksum which all take up 2 bytes each.
- **Source port:** Used by clients to indicate the session on the local client that the packet came from
- **Destination port:** Used by clients to indicate the service required from a remote server.
- **UDP length:** The total number of bytes occupied by the UDP header information as well as the payload
- **UDP checksum:** Used for error checking of the datagram. In IPV4 this was optional but is mandatory in IPV6. This checks that packets haven't been corrupted in there transfer.

 I referenced UDP as an unreliable protocol. Unreliable in this context means we have no gurantee that data gets delivered, or even if we have duplicates. The protocol doesn't establish an end-to-end connection with two systems which are communicating. That means UDP datagrams may be sent without even having a connection to the end system. UDP doesn't handle lost packet retransmission, datagram duplication or ordering and also protecting against delay within a network. TCP can deal with all these problems. 

In sending packets, clients usually decided their own port and servers determine their port based on well known port numbers. In server replies, the server indicates which port on the client to send datagrams to, keeping the "conversation" relevant to that port. A server will listen for datagram and determine the correct port number to send the datagram to. The source port number is sent out by the server, with the destination port the client selects.

<p align="center">
	<img src="/assets/ports-udp.png">
</p>Let's implement this in some code.

### udp-client.py
{% highlight python %}
import socket

SERVER_HOST = '127.0.0.1' #Loopback IP
SERVER_PORT = 10000

# Creating the UDP socket
#First argument specifies protocol family, second argument specifies UDP
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

message = "Hello world"
s.connect((SERVER_HOST, SERVER_PORT))
s.send(message)
while message != 'q':
    message = raw_input("Send message: ")
    s.send(message)
{% endhighlight %}

### udp-server.py
{% highlight python %}
import socket

SERVER_IP = "127.0.0.1"
SERVER_PORT = 10000

s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.bind((SERVER_IP, SERVER_PORT))

while 1:
  client_data, client_address  = s.recvfrom(1024)
  print "Client message: %s \nClient address %s \n" % (client_data, str(client_address))
{% endhighlight %}

Below is example output
{% highlight bash %}

NashMacbook:udp nashe$ python udp-server.py 
Client message: Hello world 
Client address ('127.0.0.1', 60881) 

Client message: Hello again 
Client address ('127.0.0.1', 60881) 


NashMacbook:udp nashe$ python udp-client.py 
Send message: Hello again
Send message: 

{% endhighlight %}
**Sidenote: Although we specified a specific port in the python code, we can see that a different port was set. This is because the OS kernel determines the port to assign for the socket**  
### Conclusion

That's a little bit about the UDP protocol, and the basics of how it functions. In the next part I plan to discuss the TCP protocol.

Thanks for reading

~Nashe
