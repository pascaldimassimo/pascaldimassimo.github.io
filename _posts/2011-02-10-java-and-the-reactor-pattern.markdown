---
layout: post
title: 'Java and the Reactor pattern'
---

The <a href="http://en.wikipedia.org/wiki/Reactor_pattern">Reactor pattern</a> is a common design pattern to provide nonblocking I/O.﻿ Instead of having multiple threads that are blocked waiting for IO to complete on a connection, you assign a single thread that is responsible to monitor all the connections. When all the IO operations are completed for a connection, that thread can fire up an event so that another thread starts processing the data coming from the connection. This approach works well when you have to handle a lot of connections, because you are not force to dedicate a thread for each connection, which might consume lot of resources if the number of connections is high.

Since Java 1.4, the <a href="http://download.oracle.com/javase/6/docs/api/java/nio/channels/Selector.html">Selector class</a> provides an implementation of this pattern. You start by registering connections to a Selector instance. Then you call the <em>select</em> method of the Selector to get a list of all the connections that are ready to do IO operations, like read and write. A single thread can be assigned the responsibility of polling the selector and send notifications when a connection is complete. See this <a href="http://www.ibm.com/developerworks/java/library/j-javaio/">nice tutorial</a> for more details on how to use the Selector class.

To familiarize myself with the Selector class, I wrote <a href="https://github.com/pascaldimassimo/nio-crawler">nio-crawler</a>. It uses a single thread to fetch links from the web, using the Selector class. When a page is fully downloaded, it is passed to a handler thread that parses the HTTP and the HTML to get the new links to follow. The handler threads never block on IO, so they never sit idle (as long as there is data to parse, of course).

Comments are welcome!
