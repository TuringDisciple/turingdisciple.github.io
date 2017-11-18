---
layout: post
title: "Building a port scanner in Python"
---

### Introduction
So I started reading the book [Violent Python](https://www.amazon.com/Violent-Python-Cookbook-Penetration-Engineers/dp/1597499579). It's a really good book and I'm enjoying it so far. So I thought I'd do a writeup on one of the first things it teaches you (literally page 1), and that's how to build a port scanner for reconnaissance. Splitting the task into 4 parts, I'm assuming familiarity of the [python socket library](https://docs.python.org/2/library/socket.html) and how basic networking knowledge.

### Step 1: Dealing with command line arguments
We accept the hostname and port from the user via command line using the optparse library
{% highlight python %}
import optparse

parser = optparse.OptionParser('usage %prog -H' + '<targer host> -p <target port>')
parser.add_:option('-H', dest='tgtHost', type='string', help='specify target host')
parser.add_option('-p', dest='tgtPort', type='int', help='specify target port')

(options, args) = parser.parse_args()
tgtHost = options.tgtHost
tgtPort = options.tgtPort

if (tgtHost == None) or (tgtPort == None):
    print parser.usage
    exit(0)
{% endhighlight %}

Not much to be said here, I believe it would be more beneficial to just read up on the libraries functionality independently.

### Step 2: portScan and connScan
Here we create the basis for two functions called connScan and portSca. portScan takes the hostname and target ports as arguments. It attempts to resolve an IP address to a hostname using the gethostbyname method in sockets. It will then print the hostname and enumerate through each individual port attempting to connect using connScan. Connscan takes two arguments: tgtHost and tgtPort. It attempts to create a connection to the target host and port, if successful, connScan prints an open port message, otherwise it prints a closed port message.

{% highlight python %}
import socket
from socket import *
def connScan(tgtHost, tgtPort):
    try:
        connSkt = socket(AF_INET, SOCK_STREAM)
        connSkt.connect((tgtHost, tgtPort))
        print '[+]%d/tcp open' % tgtPort
        connSkt.close()
    except:
        print '[-]%d/tcp closed' % tgtPort

def portScan(tgtHost, tgtPorts):
    try:
        tgtIP = gethostbyname(tgtHost)
    except:
        print "[-] Cannot resolve '%s': Unknown host" % tgtHost
        return

    try:
        tgtName = gethostbyaddr(tgtIP)
        print '\n[+] Scan results for: ' + tgtName[0]
    except:
        print '\n[+] Scan results for: ' + tgtIP
    
    setdefaulttimeout(1)
    for tgtPort in tgtPorts:
        print 'Scanning port ' + tgtPort
        connScan(tgtHost, int(tgtPort))
{% endhighlight %}

### Step 3: Reading the application banner
We now modify the connscan function to so that we can obtain the application banner from the target host. This tells us what application is using that port. AFter discovering the open port we send a string (junk data) to the port and wait for a response. Gathering this response can tell us about the application.

{% highlight python %}
def connScan(tgtHost, tgtPort):
    try:
        connSkt = socket(AF_INET, SOCK_STREAM)
        connSkt.connect((tgtHost, tgtPort))
        connSkt.send("ViolentPython\n")
        results= connSkt.recv(100)
        print '[+]%d/tcp open' % tgtPort
        print '[+] ' + str(results)
        connSkt.close()
    except:
        print '[-]%d/tcp closed' % tgtPort

{% endhighlight %}

### Step 4: Adding multithreading
To help with speed of port connection, we can add threading to the port scanner to allow multiple threads to make connections to ports. However, when it comes to printing to the screen we will want data to not look out of order. By using a [semaphore](https://en.wikipedia.org/wiki/Semaphore_%28programming%29) we can restrict access to the screen for printing by using a lock.

{% highlight python %}
from threading import Semaphore, Thread
screenlock = Semaphore(value=1) 

# We add this code to portScan
#for tgtPort in tgtPorts:
#        t = Thread(target=connScan, args=(tgtHost, int(tgtPort)))
#        t.start()

# We modify connScan as follows. Acquire method obtains the lock if free, else waits. Finally is
# always called to ensure lock release

def connScan(tgtHost, tgtPort):
    try:
        connSkt = socket(AF_INET, SOCK_STREAM)
        connSkt.connect((tgtHost, tgtPort))
        connSkt.send('ViolentPython\r\n')
        results = connSkt.recv(100)
        screenlock.aquire()
        print '[+]%d/tcp open' % tgtPort
        print '[-]%d/tcp closed' % tgtPort
    except:
        screenlock.acquire()
        print '[-]%d/tcp closed' % tgtPort
    finally:
        screenlock.release()
        connSkt.close()
{% endhighlight %}

### Final code: PortScanner.py
This is how the final script looks

{% highlight python %}
import optparse
import socket
from socket import *
from threading import Semaphore, Thread 

screenlock = Semaphore(value=1)

def portScan(tgtHost, tgtPorts):
    try:
        tgtIP = gethostbyname(tgtHost)
    except:
        print "[-] Cannot resolve '%s': Unknown host" % tgtHost
        return

    try:
        tgtName = gethostbyaddr(tgtIP)
        print '\n[+] Scan results for: ' + tgtName[0]
    except:
        print '\n[+] Scan results for: ' + tgtIP
    
    setdefaulttimeout(1)
    for tgtPort in tgtPorts:
        t = Thread(target=connScan, args=(tgtHost, int(tgtPort)))
        t.start()


def connScan(tgtHost, tgtPort):
    try:
        connSkt = socket(AF_INET, SOCK_STREAM)
        connSkt.connect((tgtHost, tgtPort))
        connSkt.send('ViolentPython\r\n')
        results = connSkt.recv(100)
        screenlock.aquire()
        print '[+]%d/tcp open' % tgtPort
        print '[-]%d/tcp closed' % tgtPort
    except:
        screenlock.acquire()
        print '[-]%d/tcp closed' % tgtPort
    finally:
        screenlock.release()
        connSkt.close()

def main():
    parser = optparse.OptionParser("usage%prog " + "-H <target host> -p <target port>")
    parser.add_option('-H', dest='tgtHost', type='string', help='specify target host')
    parser.add_option('-p', dest='tgtPort', type='string', help='specify target port[s] seperated by comma') 
    (options, args) = parser.parse_args()
    tgtHost = options.tgtHost
    tgtPorts=str(options.tgtPort).split(',')
    if(tgtHost == None) or (tgtPorts[0] == None):
        print '[-] You must specify a target host and port[s]'
        exit(0)
    portScan(tgtHost, tgtPorts)
if __name__ == '__main__':
        main()

{% endhighlight %}

Running it produces the following 

{% highlight bash %}
NashMacbook:violent-python nashe$ python PortScanner.py -H 127.0.0.1 -p 4000,8008,21,32

[+] Scan results for: localhost
[-]4000/tcp closed
[-]21/tcp closed
[-]8008/tcp closed
[-]32/tcp closed
{% endhighlight %}

No ports on my machine were open, but for a machine accepting ports you can predict what you would see

### Extension: nmapScanner.py
There exists a library already for using nmap(which produces XML) within python. Below is a script that would do the same as PortScanner.py but using the ever so famous nmap.

{% highlight python %}
import nmap # To install use pip or pip3 if using python 3
import optparse

def nmapScan(tgtHost, tgtPort):
    nmScan = nmap.PortScanner()
    nmScan.scan(tgtHost, tgtPort)
    state = nmScan[tgtHost]['tcp'][int(tgtPort)]['state']
    print " [*] " + tgtHost + " tcp/" + tgtPort + " " + state

def main():
    parser = optparse.OptionParser("usage%prog " + "-H <target host> -p <target port>")
    parser.add_option('-H', dest='tgtHost', type='string', help='specify target host')
    parser.add_option('-p', dest='tgtPort', type='string', help='specify target port[s] seperated by comma') 
    (options, args) = parser.parse_args()
    tgtHost = options.tgtHost
    tgtPorts=str(options.tgtPort).split(',')
    if(tgtHost == None) or (tgtPorts[0] == None):
        print '[-] You must specify a target host and port[s]'
        exit(0)
    for tgtPort in tgtPorts:
        nmapScan(tgtHost, tgtPort)
if __name__ == '__main__':
        main()
{% endhighlight %}

### Conclusion

Not much to say, I anybody is reading this, get Violent Python 'tis da best.

~Nashe
