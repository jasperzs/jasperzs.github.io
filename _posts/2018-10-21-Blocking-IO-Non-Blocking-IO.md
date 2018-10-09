---
layout:     post
title:      Blocking I/O vs. Non-Blocking I/O
subtitle:   Understanding I/O
date:       2018-10-21
author:     Mr.Humorous 🥘
header-img: img/webApp/header.jpg
catalog: true
tags:
    - WebApp
    - Blocking I/O
    - Non-Blocking I/O
---

## 1. Client Server Application Overview
![Client Server App Overview]({{ site.url }}/img/inputoutput/clientServerApplication.png)

To make a connection, the client and the server both bind to a socket on their respective ends.

The server waits listening to its socket for client to make a request for a connection.
Once a connection is made, the server and the client both read and write data to the sockets that are bound to that connection.

A socket is <ins>__"IP address + port number + transport protocol(e.g. TCP & UDP)"__</ins>.
TCP layer uses server's port number which is bound to the socket to identify which application to send data to.

## 2. Blocking I/O
When a client makes a request to connect with the server, the thread that handles that connection is blocked until there is some data to read or the data is fully written.
Until the relevant operation is complete that thread can do nothing else but wait. Now to fulfill concurrent requests we need to allocate a new thread for each client connection.

+ Here, we have created a new server socket to listen to request connections on a specific port and now the server is bound to this port.
![Server Bound To Socket]({{ site.url }}/img/inputoutput/blockingio_1.png)
```java
ServerSocket serverSocket = new ServerSocket(portNumber);
```

+ Next, when we call the accept() method, server starts to wait for the client to make a connection and when a client makes a request, the server socket accepts the connection from the client and returns a new socket to communicate with the client. Until this new connection is established server socket gets blocked but once the connection is made it goes back to listen to client connections over the original server socket.
![Client Makes A Connection Request]({{ site.url }}/img/inputoutput/blockingio_2.png)
![Client Establishes a Connection]({{ site.url }}/img/inputoutput/blockingio_3.png)
```java
Socket clientSocket = serverSocket.accept();
```

+ Next, we can get the input and output streams from the socket.
```java
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
```

+ Then we read a line of information from input stream and process what the client has sent and writes the response to the client trough the output stream attached to the socket. This happens until the client sends “Done” or type end-of-input character (by pressing __Ctrl-C__).
```java
String request, response;
while ((request = in.readLine()) != null) {
    response = processRequest(request);
    out.println(response);
    if ("Done".equals(request)) {
      break;
    }
}
```

+ Now the important thing to remember here that this code works for one connection at a time. To handle multiple concurrent users we need to allocation a new thread for each client socket.
![Client Makes A Connection Request]({{ site.url }}/img/inputoutput/blockingio_4.png)
```java
while (listening) {
    accept a connection;
    create a thread to deal with the client;
}
```

There are few drawbacks to this approach.
+ Each thread requires a stack of memory allocated to it and with the increase number of connections, spawning multiple threads and switching between them will become cumbersome.
+ At any give point in time there can be multiple threads just waiting for the client requests and that is just a waste of resources.
Therefore, this blocking I/O approach is not ideal if you have to cater to a large number of clients, but to a small to a moderate number of clients this would do just fine.

Full source code of blocking I/O is <a href="https://github.com/arukshani/JavaIOAndNIO/tree/master/src/com/ruk/blocking/io">here</a>

## 3. Non-blocking I/O
With non-blocking I/O, we can use a single thread to handle multiple concurrent connections. Few terms to understand:
+ In NIO based systems, instead of writing data onto output streams and reading data from input streams, we read and write data from “buffers”. You can think of the “buffer” as a temporary storage place and there are different types of Java NIO buffer classes (eg:- ByteBuffer , CharBuffer , ShortBuffer etc..) available for us to use, even though most network programs use ByteBuffer exclusively.
+ “Channel” is the medium that transports bulk of data into and out of buffers and it can be viewed as an endpoint for communication. (For example if we take “SocketChannel” class, it reads from and writes to TCP sockets. But the data must be encoded in ByteBuffer objects for reading and writing.)
+ Then we need to understand a concept called “Readiness Selection” which basically means “the ability to choose a socket that will not block when data is read or written”. Let’s explore this a little bit more.

Java NIO has a class called “Selector” that allows a single thread to examine I/O events on multiple channels. That is, this selector can check the readiness of a channel for operations, such as reading and writing. Now remember different channels can be registered with a “Selector” object and you can specify which operations you are interested in observing and a another thing to remember is that each of these channels are assigned a separate “SelectionKey” which serve as a pointer to a channel.

![Client Makes A Connection Request]({{ site.url }}/img/inputoutput/nonblockingio_1.png)

First let's look at the server code.
+ First we need to create a selector to handle multiple channels and more importantly they allow the server to find all the connections that are ready to receive output or send input.
```java
Selector selector = Selector.open();
```
+ Now let’s create a server socket channel in a non blocking manner and this “ServerSocketChannel” class is wholly responsible for accepting new incoming connections.
```java
ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.configureBlocking(false);
```
+ Then we can bind the server socket channel to a particular host and a port.
```java
InetSocketAddress hostAddress = new InetSocketAddress(hostname, portNumber);
serverChannel.bind(hostAddress);
```
+ Now we need to register this server socket channel with the selector and the “SelectionKey.OP_ACCEPT” parameter tells the “selector” to listen to only incoming connections. Basically the second parameter tells what events we are interested in listening for in the monitored channel. In our case, “OP_ACCEPT” tells that the server socket channel is ready to accept a new connection from a client.
```java
serverChannel.register(selector, SelectionKey.OP_ACCEPT);
```
+ Next we will call selector’s select() method to check whether there’s anything ready to be acted on. Call this inside an infinite loop if you want to wait infinitely for new activity. “readyCount” denotes the number of ready channels. If we don’t have any ready channels we can continue to wait.
```java
while (true) {
   int readyCount = selector.select();
   if (readyCount == 0) {
      continue;
   }
   // process selected keys...
}
```
+ Once the selector finds a ready channel, “selectedKeys()” method returns a set of “readyKeys” each representing a ready channel and we can loop through each channel and perform the necessary operations.
```java
// process selected keys...
Set<SelectionKey> readyKeys = selector.selectedKeys();
Iterator iterator = readyKeys.iterator();
while (iterator.hasNext()) {
  SelectionKey key = iterator.next();
  // Remove key from set so we don't process it twice
  iterator.remove();
  // operate on the channel...
}
```
+ Important thing to notice here is, only one thread, the main thread, processes multiple simultaneous connections.
+ Next, let’s see when we get a channel, how we can handle the operational logic. The channel represented by SelectionKey can either be server socket informing that a new connection has been made or a client socket that is ready to read or write data onto the channel.
+ If key is “acceptable” that means client requires a connection.
```java
 // operate on the channel...
 // client requires a connection
if (key.isAcceptable()) {
    ServerSocketChannel server = (ServerSocketChannel)  key.channel();
    // get client socket channel
    SocketChannel client = server.accept();
    // Non Blocking I/O
    client.configureBlocking(false);
    // record it for read/write operations (Here we have used it for read)
    client.register(selector, SelectionKey.OP_READ);
    continue;
}
```
+ If key is “readable” that means server is ready to read data from client.
```java
    // if readable then the server is ready to read
    if (key.isReadable()) {

    SocketChannel client = (SocketChannel) key.channel();

    // Read byte coming from the client
    int BUFFER_SIZE = 1024;
    ByteBuffer buffer = ByteBuffer.allocate(BUFFER_SIZE);
    try {
    client.read(buffer);
    }
    catch (Exception e) {
    // client is no longer active
    e.printStackTrace();
    continue;
    }
```
+ If key is “writable” that means server is ready to write data to client.
```java
if (key.isWritable()) {
  SocketChannel client = (SocketChannel) key.channel();
  // write data to client...
}
```

Now we will write a simple client to connect to the server.
+ First the client has to create a socket channel to connect to the server
```java
SocketAddress address = new InetSocketAddress(hostname, portnumber);
SocketChannel client = SocketChannel.open(address);
```
+ Now instead of asking for socket’s input and output streams, we are going to write data to the channel itself. But as we know now, we need to encode data in “ByteBuffer” objects to write to the channel. So let’s create a ByteBuffer with a 74-byte capacity.
```java
ByteBuffer buffer = ByteBuffer.allocate(74);
```
+ Fill the buffer with client message and write to the channel.
```java
buffer.put(msg.getBytes());
buffer.flip();
client.write(buffer);
```

Full source code of non-blocking I/O is <a href="https://github.com/arukshani/JavaIOAndNIO/tree/master/src/com/ruk/nio">here</a>
