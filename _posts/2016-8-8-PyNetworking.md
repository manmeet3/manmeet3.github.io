---
title: "Python Network Programming"
layout: post
categories: Blog
comments: true
---


This is a quick and to the point overview of network programming in python. The reason behind writing this is to get myself up to speed in network programming and for others who might be looking for a succinct and clutter-less tutorial on the subject as well.


<div align="center">
This post will be broken down into 5 stages: <br/>
I. Networking Fundamentals<br/>
II. Client Programming<br/>
III. Internet Data Handling<br/>
IV. Web Programming<br/>
V. Advanced Networking
<br/>
</div>

## I. Networking Fundamentals
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

### TCP Sockets

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
>>> c,a = s.accept()   	   # c is new socket created for this connection while a is the network/port addr of client as a tuple
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


Socket programming is often a mess: huge number of options, many corner cases and many failure modes/reliability issues.

**Partial Reads/Writes**  happen where *recv()* length is a maximum limit, but it can be the case that less data than expected is received. In TCP, the data stream is continuous and thus no concept of records or fragments exists. A client sending 2 packets of data (in UDP terms) to a server through TCP, if interrupted at an arbitrary point before finishing, could be at any position within those two packets. 
For most non-specialized applications, you can send all data using *sendall()* or in complete syntax *socket.sendall(data)*. You would not use this if networking is mixed in with other kinds of processing (ie. screen updates, multitasking, etc.)

**Data Reassembly** is done in a loop as shown in the example above. You basically initiate a forever loop, get data in chunks and append the fragments to (generally) an array. If a null is received through socket.recv(data) command, it means the end of sent data.

Another point is to note the .join used in the case of a string. You should lean towards using .join where possible as += is slow (allocating a new string or other DS thats an addition of the two).


**Timeouts** are used to prevent deadlocks in case a receiver is waiting indefinitely for the sender to send data. In such casses, timesouts can be used to break connection if no response is received and raise a *timeout* exception. Timeouts can be disabled using: S.settimeout(None). 

**Non-blocking Sockets**
Instead of disabling timeouts, we can also set sockets to a non-blocking state using :s.setblocking(False). Future send(),recv() operations will raise an exception if the operation would have blocked. 


The difference between blocking and non-blocking sockets is that in blocking sockets, once a recv() command is issued, the control is not returned to the program until either data is received or an error (possibly timeout) occurs. This means that any other operation must wait. In non-blocking sockets on the other hand, you don't have to wait for an operation to complete, and an exception is raised in the case that a connection cannot be established. This is an invaluable tool if you need to switch between many different connected sockets, and want to ensure that none of them cause the program to lock up.

*Socket Options*: Sockets have a large number of parameteres that can be set using *s.setsockopt()*


As examples: <br>
a) Blocking I/O  <br>
Wait until a new connection is established
{% highlight py %}
>>> from socket import *
>>> s = socket(AF_INET, SOCK_STREAM)
>>> s.bind(("",15000))
>>> s.listen(5)
>>> c,a = s.accept()		#function waits indefinitely until a new connection arrives--blocking I/O in action
>>> s.send("Hello?")		# typed in case of c, to show that pending messages can be received immediately
{% endhighlight %}
In this code, the accept() function waits indefinitely until a new connection arrives. This is blockin I/O in action.


b) Blocking I/O with a timeout <br>
In this case the settimeout() method sets a time in seconds to wait before calling a non-established connection quits.
{% highlight py %}
>>> from socket import *
>>> s = socket(AF_INET, SOCK_STREAM)	# client socket
>>> s.connect(("localhost",15000))     # connect to the server above
>>> c.settimeout(15)
>>> c.recv(8192)				#... wait 15 seconds. See what happens ...	

{% endhighlight %}
c) Non-blocking I/O   <br>
In this case, operations immediately raise an exception if the operation can't be completely. To the code in part b, add:
{% highlight py %}
>>> c.setblocking(False)
>>> c.recv(8192)
Traceback (most recent call last):
  File "", line 1, in 
socket.error: [Errno 35] Resource temporarily unavailable
>>> c.recv(8192)
'Hello?'				# Pending string received and no error thrown. All because the data was ready to be received upon opening connection
{% endhighlight %}

<h3> UDP + other supported socket types </h3>

<b>UDP:</b> Datagrams are packets of data sent as discrete entities. They have no concept of a connection, reliability or ordering. Datagrams can be lost or arrive in random order. Although one important thing to take note is that the lack of security checks means that it is fast and lightweight, hence used in video games, skype, etc.

A simple UDP Server can look like:
{% highlight py %}
>>> from socket import *
>>> s = socket(AF_INET, SOCK_DGRAM)
>>> while True:
...  data,addr = s.recvfrom(maxsize)		# Sent Data
...  resp = "String to send"
...  s.sendto(resp,addr)				# Important to not that data is sent and received in the same loop
{% endhighlight %}

Sending a datagram to a server:
{% highlight py %}
>>> from socket import *
>>> s = socket(AF_INET,SOCK_DGRAM)
>>> msg = "Hello World"
>>> s.sendto(msg,("server.com",10000))
>>> data, addr = s.recvfrom(maxsize)
data, addr = s.recvfrom(maxsize) 		# Optional to wait for a response where data is the destination variable and addr is address to receive from
{% endhighlight %}


Essentially the difference between UDP and TCP is that TCP's continuous data stream exchanges files and data sequentially in the order it is requested and sent where as UDP has no concept of this and begins to perform a send or receive operation as soon as it is executed.

*Unix domain sockets* are sometimes used for fast inter-process-communication or pipes between processes

{% highlight py %}
>>> s = socket(AF_UNIX, SOCK_STREAM | SOCK_DGRAM)	# creation of socket
>>> s.bind("/tmp/foo")		# Server binding
>>> c.connect("/tmp/foo")		# Client connection 
				# Rest of the programming interface the same
{% endhighlight %}

**Sockets and Concurrency**
For a server handing multiple clients, each client gets its own socket on server. To manage multiple clients:
Server must always be ready to accept new connections
and allow each client to operate independently. There are many options here, the three we'll discuss are <br>

1) Threaded Server: Each client handled by a unique thread
{% highlight py %}
>>> import threading
>>> from socket import *
>>> 
>>> def handle_client(c):	# defined function to be called by each new thread later
>>>   # handle each client here
>>>   c.close()
>>>   return
>>> 
>>> s = socket(AF_INET,SOCK_STREAM)
>>> s.bind(("",9000))
>>> s.listen(5)
>>> while True:	# typical create and bind socket and run in a loop
>>> c,a = s.accept()
>>> t = threading.Thread(target=handle_client, args=(c,))	# call function defined above, using as a parameter
{% endhighlight %}

2) Forking Server (UNIX): Each client handled by a subprocess
{% highlight py %}
>>> import os
>>> from socket import *
>>>
>>> s = socket(AF_INET,SOCK_STREAM)
>>> s.bind(("",9000))
>>> s.listen(5)
>>> while True:	
>>>   c,a = s.accept()
>>> 
>>>   if os.fork() == 0:
>>>       # Child process. Manage client here
>>>       c.close()
>>>       os._exit(0)
>>>   else:
      # Parent process. Clean up and go back to wait for more connections
>>> c.close()
{% endhighlight %}

3) Asynchronous Server: Server handles all clients in an event loop

{% highlight py %}
>>> import select
>>> from socket import *
>>>
>>> s = socket(AF_INET,SOCK_STREAM)
>>> clients = [] # List of all active client sockets
>>> while True:
>>>   # Look for activity on the sockets
>>>   input,output,err = select.select(s+clients, clients, clients)
>>>   # process all sockets ready for output
>>>   for o in output: # do something for each
>>> 
{% endhighlight %}

A few **Utility Functions**:

{% highlight py %}
>>> socket.gethostname() # As stated -- returns a hostname like 'foo.bar.com'
>>> socket.gethostbyname("www.python.org") # Returns ip address of the remote machine
>>> socket.gethostbyaddr("82.93.246.218")  # Fetches information on remote ip like its hostname and other available data.
{% endhighlight %}


## II. Client Programming
**Overview** - Python has library modules for interacting with a variety of standard internet services: HTTP, FTP, SMTP, NNTP, XML-RPC, etc. In this section, we'll look at how some of these library modules work while focusing mainly on the web (HTTP).

**urllib** module is a high level module that allows clients to connect to a variety 

