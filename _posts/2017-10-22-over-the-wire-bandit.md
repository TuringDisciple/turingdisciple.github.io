---
layout: post
title: "Over the Wire: Bandit"
---

This is my first experience with wargames. I'll be going into to some detail of my answers but I am assuming anyone reading is fairly familiar with Linux and the terminal. 

### Level 0 -> Level 1 and Level 1 -> Level 2
Login into bandit 0, then just cat the readme. 
{% highlight bash %}
bandit0@bandit:~$ ssh bandit1@bandit.labs.overthewire.org -p 2220
bandit1@bandit:~$ cat ./-
{% endhighlight %}

### Level 2 -> Level 3
The filename with the password has spaces making things a little bit more difficult

{% highlight bash %}
bandit2@bandit:~$ ls
spaces in this filename
bandit2@bandit:~$ cat spaces%in%this%filename

{% endhighlight %}

### Level 3 -> Level 4
The password is stored in a hidden file.
{% highlight bash %}
bandit3@bandit:~/inhere$ ls -a
bandit3@bandit:~/inhere$ cat .hidden
 
{% endhighlight %}

### Level 4 -> Level 5
Introducing, the file command!
{% highlight bash %}
bandit4@bandit:~$ ls 
inhere
bandit4@bandit:~$ cd    
bandit4@bandit:~$ cd inhere/
bandit4@bandit:~/inhere$ ls
-file00  -file01  -file02  -file03  -file04  -file05  -file06  -file07  -file08  -file09
bandit4@bandit:~/inhere$ file ./-*
./-file00: Non-ISO extended-ASCII text, with CR line terminators, with escape sequences
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
bandit4@bandit:~/inhere$ cat ./-file07
{% endhighlight %}

### Level 5 -> Level 6
Introducing the find command!
{% highlight bash %}
bandit5@bandit:~$ cd inhere/
bandit5@bandit:~/inhere$ cat `find ./ -size 1033c`

{% endhighlight %}

### Level 6 -> Level 7
Bit more fiddly.

{% highlight bash %}
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password 

{% endhighlight %}

### Level 7 -> Level 8
This is a simple grep request 
{% highlight bash %}
bandit7@bandit:~$ ls
data.txt
bandit7@bandit:~$ cat data.txt | grep millionth
millionth	x
{% endhighlight %}

### Level 8 -> Level 9 
We can sort lines by occurence as follows
{% highlight bash %}
bandit8@bandit:~$ cat data.txt | sort | uniq -u
{% endhighlight %}

### Level 9 -> Level 10
Similar to 7, but we use strings to obtain all readable input characters
{% highlight bash %}
bandit9@bandit:~$ strings data.txt | grep '='
|========== the
,]=NB
@k<=
"m=g	
========== password
=r-3
========== is
mu=v.
<= V57
i=Hk>$B
========== xxxxxxxxxxxxxxxxxxx
S1N=
PbgQ=Zp
=M Q
x3X}=
{% endhighlight %}

#### Level 10 -> Level 11
The base64 command has a decode option 
{% highlight bash %}
bandit10@bandit:~$ ls
data.txt
bandit10@bandit:~$ base64 -d data.txt 
The password is xxxxxxxxxxxxxxxxxxxx
{% endhighlight %}

### Level 11 -> Level 12
We can use the tr command here
{% highlight bash %}
bandit11@bandit:~$ ls
data.txt
bandit11@bandit:~$ cat data.txt | tr a-zA-Z n-za-mN-ZA-M
The password is xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{% endhighlight %}

### Level 12 -> Level 13
Worst! Level! Ever!
{% highlight bash %}
bandit12@bandit:~$ file data.txt 
data.txt: ASCII text
bandit12@bandit:~$ xxd -r data.txt > data1 
bandit12@bandit:~$ file data1        
data1: gzip compressed data, was "data2.bin", from Unix, last modified: Thu Sep 28 14:04:06 2017, max compression
bandit12@bandit:~$ bzip2 -d data1
bzip2: Can't guess original name for data1 -- using data1.out
bzip2: data1 is not a bzip2 file.
bandit12@bandit:~$ zcat data1 > dataNew
bandit12@bandit:~$ file dataNew 
dataNew: bzip2 compressed data, block size = 900k
bandit12@bandit:~$ bzip2 -d dataNew
bzip2: Can't guess original name for dataNew -- using dataNew.out
bandit12@bandit:~$ file dataNew.out 
dataNew.out: gzip compressed data, was "data4.bin", from Unix, last modified: Thu Sep 28 14:04:06 2017, max compression
bandit12@bandit:~$ zcat dataNew.out > dataNewer
bandit12@bandit:~$ file dataNewer
dataNewer: POSIX tar archive (GNU)
bandit12@bandit:~$ tar -xvf dataNewer
data5.bin
bandit12@bandit:~$ tar -xvf data5.bin
data6.bin
bandit12@bandit:~$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
bandit12@bandit:~$ bzip2 -d data6.bin 
bzip2: Can't guess original name for data6.bin -- using data6.bin.out
bandit12@bandit:~$ file data6.bin.out 
data6.bin.out: POSIX tar archive (GNU)
bandit12@bandit:~$ tar -xvf data6.bin.out
data8.bin
bandit12@bandit:~$ file data8.bin 
data8.bin: gzip compressed data, was "data9.bin", from Unix, last modified: Thu Sep 28 14:04:06 2017, max compression
bandit12@bandit:~$ zcat data8.bin > dataNew3  
bandit12@bandit:~$ file dataNew3
dataNew3: ASCII text
bandit12@bandit:~$ cat dataNew3
The password is xxxxxxxxxxxxxxxxxxxxxxxx
{% endhighlight %}

### Level 13 -> Level 14
This level instroduces us to ssh with keys. 

{% highlight bash %}
bandit13@bandit:~$ ls
sshkey.private
{% endhighlight %}

We see that the private key is within the current directory. It's pretty simple from here.

{% highlight bash %}
bandit13@bandit:~$ ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
The authenticity of host '[bandit.labs.overthewire.org]:2220 ([0.0.0.0]:2220)' can't be established.
ECDSA key fingerprint is ee:4c:8c:e7:57:2c:bc:63:24:b8:e6:23:27:63:72:9f.
Are you sure you want to continue connecting (yes/no)? yes

...

bandit14@bandit:~$ 
{% endhighlight %}

The manual says that the -i command specifies the file from which the private key for public key authenication is read.

### Level 14 -> Level 15
Had a little bit of trouble finding the password for this level, but remembering previous levels, the password was found as follows

{% highlight bash %}
bandit14@bandit:/$ cat etc/bandit_pass/bandit14  
{% endhighlight %}

We then telnet to the specified port and server

{% highlight bash %}
bandit14@bandit:/$ telnet localhost 30000
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
xxxxxxxxxxxxxxxxxxxxxxxxxxxx
Correct!

Connection closed by foreign host.
bandit14@bandit:/$ 
{% endhighlight %}

### Level 15 -> Level 16
The answer is basically spelled out for us, just have to read the openssl manual

{% highlight bash %}
bandit15@bandit:~$ cat /etc/bandit_pass/bandit15
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
bandit15@bandit:~$ openssl s_client -connect localhost:30001 -ign_eof
CONNECTED(00000003)

...

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Correct!
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{% endhighlight %}

### Level 16 -> Level 17
In this level we have to scan ports which match the credentials
- Port is in the range 31000 to 32000
- There's a server listening 
- This server speaks SSL
According to the details only one server matches these credentials 

{% highlight bash %}
bandit16@bandit:~$ cat /etc/bandit_pass/bandit16
xxxxxxxxxxxxxxxxxxxxxxxxxxxx
bandit16@bandit:~$ nmap -p 31000-32000 localhost -sV

Starting Nmap 6.40 ( http://nmap.org ) at 2017-10-22 15:42 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00057s latency).
Other addresses for localhost (not scanned): 127.0.0.1
Not shown: 996 closed ports
PORT      STATE SERVICE VERSION
31046/tcp open  echo
31518/tcp open  msdtc   Microsoft Distributed Transaction Coordinator (error)
31691/tcp open  echo
31790/tcp open  msdtc   Microsoft Distributed Transaction Coordinator (error)
31960/tcp open  echo
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.48 seconds
bandit16@bandit:~$ openssl s_client -connect localhost:31790 -ign_eof 
CONNECTED(00000003)
depth=0 CN = 8f75dc271013
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = 8f75dc271013
verify return:1
---
Certificate chain
 0 s:/CN=8f75dc271013
   i:/CN=8f75dc271013
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICvjCCAaagAwIBAgIJALADbwWQ0u9aMA0GCSqGSIb3DQEBCwUAMBcxFTATBgNV
BAMTDDhmNzVkYzI3MTAxMzAeFw0xNzA5MTYwNzAyMjRaFw0yNzA5MTQwNzAyMjRa
MBcxFTATBgNVBAMTDDhmNzVkYzI3MTAxMzCCASIwDQYJKoZIhvcNAQEBBQADggEP
ADCCAQoCggEBALmjBUTlmjROJUssm+rAlFADFfzrz+xCH0qUXryou5/wW8pnQ6nG
HbdeRIBwTVGFiDIKRbFdWQU4BbrfjEhyGn9d7eh/3GV09ZdvLDYRoLmJ4tDF8CiC
wGl9GufcWr3zeaNYa8CwVdtWam8umhMICrsv7B5iV9RdSQfudUtVbr26SBVyuhBm
m0t7Su6rLCrrGtshdIihjk4k67bBMpSNAOduhpp79UgIPKcwJUhRJHTcji3m/IQ8
O9zNS25oL8KhMn7e/Xe70kztstq0ShMsx8feutONnGulUOlaEMMqW+HSWgnVeG/r
mU9Nzwn++4qxe16OvvmXAzctH2RlDx7XbcsCAwEAAaMNMAswCQYDVR0TBAIwADAN
BgkqhkiG9w0BAQsFAAOCAQEADHODX5CcMLI5fdumzly5FAVg5Yc22eDGNhmyhi/N
kDhP6QYw+HW5nWEYapc9m/ZQGEEoxr+wj6qeEhscxRxpuEIcunZsLKcoAmToyXeO
ANMslQugRcGqN57Pt0h5VuctLMa3ickeVPFvV6gxJSHBNRK1iN8nrfsy+zR+stzI
xcjIuakDDxMKFtb/1TMKf4/EsimSQLS0WXLjbxfQ/J510O4/Of0tmZI0ZIG+cKmM
V5hAOtuuAk6jREfWYJQ3DB+phv7PO9s2FpofVJss5PK4NWDS7UQOv359ZOJ85ZpJ
ihGxDqV7IAHJZNM9lvFXz/+EOn1oTGW9V8bAwt34OVYoPw==
-----END CERTIFICATE-----
subject=/CN=8f75dc271013
issuer=/CN=8f75dc271013
---
No client certificate CA names sent
---
SSL handshake has read 1682 bytes and written 637 bytes
---
New, TLSv1/SSLv3, Cipher is DHE-RSA-AES256-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : DHE-RSA-AES256-SHA
    Session-ID: B8FE8CF276F42F7F3B5C4CA141163834B59545DFC0347C29B8DC734FC80905CF
    Session-ID-ctx: 
    Master-Key: F60DB3D5FFADCDFEA5AF728C18D9276F467143557D4BC6DF9C3C39685F17AD2C14EFD1BEA0C8FBB7B89912A95D46C7CC
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1508687222
    Timeout   : 300 (sec)
    Verify return code: 18 (self signed certificate)
---
xxxxxxxxxxxxxxxxxxxxxxxxxxxx
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----

read:errno=0
{% endhighlight %}

We obtained an RSA key, so we're going to have to use ssh to connect to next level

{% highlight bash %}
bandit16@bandit:~$ vim sshprivate.private 
bandit16@bandit:~$ chmod 600 sshprivate.private 
bandit16@bandit:~$ ssh -i sshprivate.private bandit17@localhost
{% endhighlight %}

### Level 17 -> Level 18
The power of the diff command 
{% highlight bash %}
bandit17@bandit:~$ diff passwords.new passwords.old   
42c42
< xxxxxxxxxxxxx
---
> xxxxxxxxxxxxxx
bandit17@bandit:~$ 
{% endhighlight %}

### Level 18 -> Level 19
When we try to ssh we get logged out immediately 
{% highlight bash %}
bandit17@bandit:~$ ssh bandit18@localhost           

...

Byebye !
Connection to localhost closed.
bandit17@bandit:~$ ssh bandit18@localhost cat readme

...

bandit18@localhost's password: 
xxxxxxxxxxxxxxxx
{% endhighlight %}

### Level 19 -> Level 20 
We get a SUID binary. Running it gives us some useful information
{% highlight bash %}
bandit19@bandit:~$ ls -l
total 8
-rwsr-x--- 1 bandit20 bandit19 7378 Sep 28 14:04 bandit20-do
bandit19@bandit:~$ ./bandit20-do 
Run a command as another user.
  Example: ./bandit20-do id
bandit19@bandit:~$ ./bandit20-do cat /etc//bandit_pass/bandit20 

{% endhighlight %}

### Level 20 -> Level 21
I opened two shells. In the first shell I type. Due to changes, port forwarding need to be implemented

{% highlight bash %}
NashMacbook:~ nashe$ ssh -l bandit20 -p 2220 -L 1234:localhost:22 bandit.labs.overthewire.org
{% endhighlight %}

On the second I type 
{% highlight bash %}
NashMacbook:~ nashe$ ssh localhost -p 1234
{% endhighlight %}

{% highlight bash %}
bandit20@bandit:~$ nc -l 55555
{% endhighlight %}

In the second shell I type 

{% highlight bash %}
bandit20@bandit:~$ ls
suconnect
bandit20@bandit:~$ ./suconnect 55555
{% endhighlight %}

Answer outputs to first shell  

### Level 21 -> Level 22 
We can check the cron directory and gain some info from that

{% highlight bash %}
bandit21@bandit:~$ cd /etc/cron.d
bandit21@bandit:/etc/cron.d$ ls -la
total 32
drwxr-xr-x   2 root root 4096 Sep 28 14:04 .
drwxr-xr-x 119 root root 4096 Oct 22 18:57 ..
-rw-r--r--   1 root root  102 Feb  9  2013 .placeholder
-rw-r--r--   1 root root  355 May 25  2013 cron-apt
-rw-r--r--   1 root root  120 Sep 28 14:04 cronjob_bandit22
-rw-r--r--   1 root root  122 Sep 28 14:04 cronjob_bandit23
-rw-r--r--   1 root root  120 Sep 28 14:04 cronjob_bandit24
-rw-r--r--   1 root root  510 Aug  4 20:03 php5 
bandit21@bandit:/etc/cron.d$ cat cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null 
bandit21@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
bandit21@bandit:/etc/cron.d$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
{% endhighlight %}

### Level 22 -> Level 23
Similar to the last level, we have to take a look at the cron jobs

{% highlight bash %}
bandit22@bandit:~$ cd /etc/cron.d
bandit22@bandit:/etc/cron.d$ ls -la
total 32
drwxr-xr-x   2 root root 4096 Sep 28 14:04 .
drwxr-xr-x 119 root root 4096 Oct 22 18:57 ..
-rw-r--r--   1 root root  102 Feb  9  2013 .placeholder
-rw-r--r--   1 root root  355 May 25  2013 cron-apt
-rw-r--r--   1 root root  120 Sep 28 14:04 cronjob_bandit22
-rw-r--r--   1 root root  122 Sep 28 14:04 cronjob_bandit23
-rw-r--r--   1 root root  120 Sep 28 14:04 cronjob_bandit24
-rw-r--r--   1 root root  510 Aug  4 20:03 php5
bandit22@bandit:/etc/cron.d$ cat cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
bandit22@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
bandit22@bandit:/etc/cron.d$ /usr/bin/cronjob_bandit23.sh
Copying passwordfile /etc/bandit_pass/bandit22 to /tmp/8169b67bd894ddbb4412f91573b38db3
bandit22@bandit:/etc/cron.d$ cat /tmp/8169b67bd894ddbb4412f91573b38db3
x
{% endhighlight %}
This isn't the answer, I misread the shell script, it actually outputs the password to bandit22 to that file.
The whoami set's to bandit22 when we call the script.

{% highlight bash %}
bandit22@bandit:/etc/cron.d$ echo I am user bandit23 | md5sum | cut -d '' -f 1
8ca319486bfbbc3663ea0fbe81326349  -
bandit22@bandit:/etc/cron.d$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
x
{% endhighlight %}

That's a bingo! 

### Level 23 -> Level 24
We get to write our first shell script which is cool!

{% highlight bash %}
bandit23@bandit:~$ cd /etc/cron.d
bandit23@bandit:/etc/cron.d$ ls
cron-apt  cronjob_bandit22  cronjob_bandit23  cronjob_bandit24  php5
bandit23@bandit:/etc/cron.d$ cat cronjob_bandit24
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
bandit23@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
	echo "Handling $i"
	timeout -s 9 60 ./$i
	rm -f ./$i
    fi
done
{% endhighlight %}

Let's edit run the script

{% highlight bash %}
bandit23@bandit:/var/spool/bandit24$ /usr/bin/cronjob_bandit24.sh 
/usr/bin/cronjob_bandit24.sh: line 5: cd: /var/spool/bandit23: No such file or directory

...

bandit23@bandit:~$ vim script.sh
bandit23@bandit:~$ cat script.sh 
#!/bin/bash
myname=$(whoami)

cat /etc/bandit_pass/myname > /tmp/myname
bandit23@bandit:~$ chmod -R 777 /tmp/bandit24
bandit23@bandit:~$ vim script.sh
bandit23@bandit:~$ chmod 777 script.sh 
bandit23@bandit:~$ cp script.sh /var/spool/bandit24/
bandit23@bandit:~$ cat /tmp/bandit24
x
{% endhighlight %}

At this point we get should get the password but the level appears to be broken

### Level 24 -> Level 25
I decided to write a python script to obtain the answer 

{% highlight python %}
import socket 

pin = 0
password = 'UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ'

clientSocket = socket.socket() # Default AF_INet and SOCK_STREAMM
clientSocket.connect(('localhost', 30002))
clientSocket.recv(1024)
while pin in range(0, 10000):
	m = password + ' ' + str(pin) + '\n'
	print m 
	clientSocket.sendall(m)
	data = clientSocket.recv(1024)
	print data
	pin += 1

# Connection will drop when right answer found
{% endhighlight %}

{% highlight bash %}
...

Wrong! Please enter the correct pincode. Try again.

UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5583

Wrong! Please enter the correct pincode. Try again.

UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5584

Wrong! Please enter the correct pincode. Try again.

UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5585

Wrong! Please enter the correct pincode. Try again.

UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5586

Wrong! Please enter the correct pincode. Try again.

UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5587

Wrong! Please enter the correct pincode. Try again.

UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5588

Correct!
The password of user bandit25 is xxxxxxxxxxxxxxxxxxxxx
Exiting.

UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5589


UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5590

Traceback (most recent call last):
  File "brute.py", line 12, in <module>
    clientSocket.sendall(m)
  File "/usr/lib/python2.7/socket.py", line 224, in meth
    return getattr(self._sock,name)(*args)
socket.error: [Errno 32] Broken pipe
{% endhighlight %}

### Level 25 -> Level 26 and Level 26 -> Level 27 
Lets first find the password, which apparently should be easy

{% highlight bash %}
bandit25@bandit:~$ ls
bandit26.sshkey
bandit25@bandit:~$ ssh -i bandit26.sshkey bandit26@localhost

...

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

  _                     _ _ _   ___   __  
 | |                   | (_) | |__ \ / /  
 | |__   __ _ _ __   __| |_| |_   ) / /_  
 | '_ \ / _` | '_ \ / _` | | __| / / '_ \ 
 | |_) | (_| | | | | (_| | | |_ / /| (_) |
 |_.__/ \__,_|_| |_|\__,_|_|\__|____\___/ 
Connection to localhost closed.


{% endhighlight %}

This part is actually pretty cool

{% highlight bash %}
bandit25@bandit:~$ cat /etc/passwd | grep bandit26
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
bandit25@bandit:~$ cat /usr/bin/showtext
#!/bin/sh

export TERM=linux

more ~/text.txt
exit 0
{% endhighlight %}

Now if we reduce terminal size to only allow 5 lines, we can ssh into bandit26, type vim and then type :r /etc/bandit_pass/bandit26 and vim will give us the password.

### Conclusion 

This whole wargame was really fun, it tested my abilities and I learnt a bunch of linux commands in the process. 

~Nashe 
