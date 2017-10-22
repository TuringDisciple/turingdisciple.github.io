---
layout: post
title: "Networking: Basic Socket programming in Python"
---
Prerequisites
-------------
Familiarity with Python syntax is needed and familiarity with the concept of a protocol within networking.

So what are sockets
====================

In the world of networks, sockets are the basis for communication between networks over TCP(transmission control protocol). TCP being one of the main protocols of the Internet protocol suite, TCP provides a method for streaming data between applications running on hosts communicating in a network.

A network socket is an endpoint for sending and recieving data over a single within a network. Sockets provide a the ability to communicate with other sockets, therefore providing a method for exchanging between hosts. In this tutorial we will just implement a generic client server model in Python using the [socket library](https://docs.python.org/2/library/socket.html). A few languages offer libraries to access the power of sockets, so feel free to explore them.

Our basic protocol
==================

<p align="center">
	<img src="/assets/simple-socket-communication.png">
</p>

So what is going on the above diagram. In the basic sense, some client is trying to connect to some server so that data exchange can occur. In more specific term we talk seperately about the server and client.

### Server
1. In step we initialise a socket to communicate over. In the Socket library, this initialiser can take arguments specifying whether we use TCP or UDP, and whether we use some sort of loopback(indicating that we'll only be connecting to our own machine)
2. We bind the server socket to an address indicating the connection.
3. The server "listens" to the socket meaning we are waiting for a connections to be made to the socket (our client in this case)
4. We accept data placed on the socket by the client.
5. We process the data and then send back a response to the client, indicating we are ready to recieve more data from client. At this point, the server can decide to close the connection if no data is actually recieved
6. We close the connection to the socket

### Client
The client follows a similar pattern to the server, but doesn't need to listen to the socket for connections
1. Initialise socket
2. The client can decide to optionally bind to this socket. This is optional as we don't need to really pay attention to the state of the socket we just need to attempt to make a connection to the server.
3. N/A
4. We attempt to connect to the server
5. We send data to the server to be processed, and we wait for the servers response. To continue
6. Close connection when client decides they're finished communicating with the server.

As we can see, communication is fairly simple. In our implementation of this protocol, we will not however be connecting to some remote server, but rather just connecting to our local machine. This is acheived using a [loopback](https://en.wikipedia.org/wiki/Loopback). In most situations we wouldn't be doing this, but if we wanted to processed to communicate within a machine, this would be the way to go.

Now for some code
=================

The code is split into client and server scripts. We'll look at the client code first.

### client.py

{% highlight python %}
import socket

def Main():
    hostIp       = '127.0.0.1' 
    hostPort     = 5001
    clientSocket = socket.socket() 
    clientSocket.connect((hostIp, hostPort)) #Docs specify information required for setting up connection to server

    message = raw_input('> ')

    while message != 'quit':
        clientSocket.send(message.encode()) #Established connection to server, sending message to be processed

        serverResponse = clientSocket.recv(1024) .decode()
        print "Server response: %s" % serverResponse

        message = raw_input('> ')

    clientSocket.close()

if __name__ == '__main__':
    Main()
{% endhighlight %}

We can see clearly how the client machine is setup for connections as specified above. However there are some caveats

These lines
{% highlight python %}
hostIp   = '127.0.0.1'
hostPort = 5001
{% endhighlight %}
specify some important info. The hostIp is the IP address of the server we intend to connect to. In this case we use what is called the loopback address, this basically means that the client will attempt to make a connection to our machine. hostPort indicates the port number, in this case it's basically random. What should be noted is that port numbers make it possible for different clients to access the same server and thereby share resources.

{% highlight python %}
while message != 'quit':
        clientSocket.send(message.encode()) #Established connection to server, sending message to be processed

        serverResponse = clientSocket.recv(1024) .decode()
        print "Server response: %s" % serverResponse
{% endhighlight %}
Here we have the core of the client logic. After taking in the clients data, in this case is just a something from stdin, we send the message over the connection to the server to processed. The encode() method allows for the data to be formatted appriately for data transfer. We then wait to recieve the server response and output it to standard out. 1024 indicates the buffer size, which is the maximum amount of data to be recieved at once.

### server.py

{% highlight python %}
import socket

def Main():
    hostIp   = '127.0.0.1'
    hostPort = 5001 

    serverSocket = socket.socket()
    serverSocket.bind((hostIp, hostPort))
    
    print "Listening for a connection"
    serverSocket.listen(1) 
    connection, address = serverSocket.accept() 
   
    print "Connection to %s made" % str(address)
    
    while True:
        recievedFromClient = connection.recv(1024).decode()
        if not recievedFromClient: 
            break
        print "Recieved from client '%s'" % recievedFromClient

        serverResponse = raw_input('> ')
        connection.send(serverResponse.encode())

    connection.close()


if __name__ == '__main__':
    Main()
{% endhighlight %}

The server is a little bit more complex so lets have a look.

{% highlight python %}
serverSocket.listen(1)
connection, address = serverSocket.accept() 
{% endhighlight %}
We have to check listen to the socket for connections being made. The argument (backlog argument) specifies the maximum number of queued connections. If we allowed our server to allow multiple client connections, we would wait change this value accordingly. When a connection has been made, we accept the connection. The connection value is a new socket object that is used to send and recieve data on the connection. The address value is the address bound to the socket on the other end of the connection.

{% highlight python %}
while True:
        recievedFromClient = connection.recv(1024).decode()
        if not recievedFromClient:
            break
        print "Recieved from client '%s'" % recievedFromClient

        serverResponse = raw_input('> ')
        connection.send(serverResponse.encode())
{% endhighlight %}
The server logic is similar to the client. We recieve some data from the client. But we need to check if the connection is closed. In the case that the client connection is closed, we recieve a none value and we therefore have to stop waiting for client messages. To do this we break the loop. If there was data sent by the client, the server must process it approriately. In this case, we can just send back a message read from the stdin. However we can see how this sort of exchange can be used when we want to make queries for data to a server, or webpages, or we want the server to perform some sort of processing of our data and return it back to us modified. This is what is happening in more complicated protocols. 


Conclusion
==========

I have implemented a basic protocol to highlight the basics of socket programming. Next time I'll probably try and implement TCP/IP, UDP or maybe SSH. 


~Nashe
