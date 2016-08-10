---
title: "Python Network Programming"
layout: post
categories: Blog
comments: true
---


This is a quick and to the point overview of network programming in python. The reason behind writing this is to get myself up to speed in network programming and for others who might be looking for a succinct and clutter-less tutorial on the subject as well.


<div align="center">
This post will be broken down into 6 stages: <br/>
I. Introduction<br/>
II. Networking Fundamentals<br/>
III. Client Programming<br/>
IV. Internet Data Handling<br/>
V. Web Programming<br/>
VI. Advanced Networking
<br/>
</div>

### Introduction
Network programming is a major use of python. Python standard library has a wide support for network protocols, data encoding/decoding and other utilities you might have used in C/C++. An appealing reason to use python for network programs over C/C++ or Java is that it tends to be substantially easier to code and update in the future due to its readibility.


Shortly, we will learn about: low-level programming with sockets, high-level client modules, how to deal with commons data encodings, simple web programming(HTTP), and a boiled down version of distributed computing. We will do all of this while using only the standard library that python comes with. It is important to keep in mind that much more functionality can be found in third-party modules and python has a very good module base.

The problem in question when we consider network programming is simple: it is communication between 2 computers in the form of sending/receiving bits. 

<img src="{{ site.assets }}/simpleconn.png" alt="HTML5 Icon" style= "padding: 20px; margin: 0 auto;" >  

Ports for common services are preassigned, ie. <br/>
21\_\_\_\_\_FTP<br/>
22\_\_\_\_\_SSH<br/>
23\_\_\_\_\_Telnet<br/>
80\_\_\_\_\_HTTP<br/>
443\_\_\_\_HTTPS<br/>

You can use 'netstat' to view active connections using
{% highlight ruby %}
  # netstat -a
 {% endhighlight %}

 Each endpoint of a network connection is always represented by a host and port #. In python, you write it out as a tuple (host,port), ie. ("www.python.org", 80) or ("205.172.14.5", 443).

 In the above diagram, if you consider one computer to be a client and the other to be a server, the following must be true for data to be sent from one side to another: <br>
 1) Each endpoint is running a program. <br>
 2)The server waits for incoming connections and provide a service.<br>
 3) Clients make connections to the server. <br>

 Most network programs use a <b>request/response model</b> based on messages. A cliend sends a request message (e.g., HTTP)

 {% highlight ruby %}
  GET /index.html HTTP/1.0
 {% endhighlight %}
 and the server responds with

{% highlight ruby %}
HTTP/1.0 200 OK
Content-type: text/html
Content-length: 48823
<HTML>
... {% endhighlight %}

<b> Data Transport</b> consists of two basic types of communication: <br>
1) Streams (TCP): This is where computers establish with each other and read/write data in a continuous stream of bytes (a file). This is used most commonly.<br>
2) Datagrams (UDP): Computers send discrete packets of some bytes to each other. The packets are separate and self-contained entities.

<b> Sockets </b> are programming abstraction for network code. These are the endpoints for a communication channel between two devices. To create a socket in python:
{% highlight ruby %}
import socket
s = socket.socket(addr_family, type)
{% endhighlight %}

*addr_family* can be:<br>
{% highlight ruby %}
socket.AF_INET  # IPv4
socket.AF_INET6  # IPv6
{% endhighlight %}

while *type* can be: <br>
{% highlight ruby %}
socket.SOCK_STREAM  # TCP
socket.SOCK_DGRAM  # UDP
{% endhighlight %}

An example of a TCP client requesting, saving and printing data:
{% highlight ruby %}
>>> from socket import *
>>> s = socket(AF_INET,SOCK_STREAM)  # create socket
>>> s.connect(("www.python.org",80)) # connect to network,host tuple
>>> s.send("GET /index.html HTTP/1.0\r\n\r\n") # send request
>>> chunks = []                      # create an array to put data into
>>> while True:
        chunk = s.recv(16384)        # receive 16384 bytes each iteration
        if not chunk: break          # if null is reached, break
        chunks.append(chunk)         # append each chunk to array
>>> s.close()                        # close connection after null is reached
>>> response = "".join(chunks)       # join individual chunks to create string
>>> print response                   # look at the output
{% endhighlight %}


Server Implementation is a bit trickier. A server is typically run forever in a server-loop and listen for incoming connections on a well-known port number. It also has to address the possibility of servicing multiple clients.


An example of a basic server (listening on a port) for a client is:

Waiting for connection:
{% highlight ruby %}
>>> from socket import *
>>> s = socket(AF_INET, SOCK_STREAM)
>>> s.bind(("",15000))
>>> s.listen(5)
>>> c,a = s.accept()
# executed after cliend sends data
>>> data = c.recv(1024)
>>> data
'Hello World'
>>> c.send("Hello Yourself")
14
>>> c.send("Goodbye")
7
>>> c.close()
>>> c,a = s.accept()		# goes back to listening for more clients
{% endhighlight %}

Making a connection:
{% highlight py %}
>>> from socket import *
>>> s = socket(AF_INET, SOCK_STREAM)
>>> s.connect(("localhost",15000))
>>> c
<socket._socketobject object at 0x10028d910>
>>> a
('127.0.0.1', 62793)
>>> s.send("Hello World")
11
# after Server sends back a reply to hello string
>>> resp = s.recv(1024)
>>> resp
'Hello Yourself'
>>> s.recv(1024)			#wait until next signal from server
'Goodbye' # after the server sends close signal with a goodbye
#server sends close and {% endhighlight %}
