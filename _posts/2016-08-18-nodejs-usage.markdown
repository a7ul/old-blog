---
layout: post
title: When should I use Node JS for backend ?
date: '2016-08-18 10:00:37 +0530'
categories: web
---

# When should I consider Node JS for my backend project ?

Javascript is one of my favorite languages. Its easy to learn and get started with. So, When I learnt that I could write server side code also in JS, I have to admit, I was very excited!

So, I began writing all my backend projects in Node JS without even giving a second thought. After few projects, I realized that while Node JS fits in really well for most of my projects, it does not perform nicely for some. After some research, I finally found the answer to my question: **"When exactly should I use Node JS ?"**.

To understand this we need to see how Node works !

## Understanding the way a Node app runs.

First of all, Node JS is not a server. In the sense, Node JS doesn't serve anything directly. One needs to create an application server instance using other JS libraries. Hence, it basically is Javascript with added features to support majority of the core backend needs like access to filesystem, OS system calls ,etc. The magic happens at the V8 JavaScript Engine which runs the node program.

### Process model

Node JS follows event based programming. This is the absolute core of Node. It's a single threaded program that acts upon callbacks and never blocks on the main thread.

Hence, instead of spawning a new thread for every new user, node loops through events of all the users in a single thread itself. This avoids thread creation overhead. So, ideally the main thread only listens for events and nothing else. It never gets blocked because of the way node forces you to write code. The code you write should always be asynchronous and callbacks driven.

Take a look at the diagram below:

<div style="text-align:center">
  <img src="/assets/EventLoop/eventloop.png" style="width: 80%;display: inline;">
</div>

<br>

Events can be any action, like HTTP request or even callback from any I/O operation for which the application was waiting. Node has an event loop which processes every event that comes up in the event queue. Hence, whenever an event occurs, the event is processed and the associated callback is called.

Now, there are two types of applications based on how they behave.

1. **CPU intensive** - These applications require large amount of cpu cycles for their operations. For example : Heavy mathematical calculations, data crunching , Image processing ,etc.
2. **I/O intensive** - These applications perform mostly Input/Output operations. For example: Database access, file reading and writing, responding to HTTP requests and so on.

CPU intensive applications are best suited for multi threaded systems, obviously because they will be performed by multiple cores simultaneously. But turns out node doesn't work that way. Node based apps are single threaded. Hence, a single thread will need to perform all the cpu intensive jobs which will make it very slow. This is the reason if you have ever tried building image processing apps on node, you would have been pretty disappointed at the speed.

While, incase of I/O operations, they are essentially performed not by the main application itself. For example, database operations are performed by the DB, file read and write is carried out by the OS itself. The main application just provides instructions on what needs to be performed and then waits for the callback. Node JS is best suited for this purpose and is faster than multicore , multi threaded applications primarily because of the processing model it follows. And as it turns out most of the web applications are structured this way.

### Final verdict!

Due to the way NodeJS works we can come to the conclusion that :

- **Potentially I/O intensive application** - Node JS might be the best choice  &nbsp; &nbsp; <span style="font-size:25px;color:green">✔</span>


- **Potentially CPU intensive application** - Node JS will be very slow  &nbsp; &nbsp; <span style="font-size:25px;color:red">✘</span>

  Go for multithreaded systems like Java.
