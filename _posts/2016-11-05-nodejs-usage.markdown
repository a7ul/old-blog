---
layout: post
title: When should I use Node JS for backend ?
date: '2016-11-05 10:00:37 +0530'
categories: web
comments: true
---

Javascript is one of my favorite languages. Its easy to learn and get started with. So, When I learnt that I could write server side code also in JS, I have to admit, I was very excited!
<br>

<div style="text-align:center">
  <img src="/blog-atul/assets/event-loop/make_it_node.jpg" style="width: 40%;display: inline;">
</div>
<br>

So, I began writing all my backend projects in Node JS without even giving a second thought. After few projects, I realized that while Node JS fits in really well for most of my projects, it does not perform nicely for some. After some research, I finally found the answer to my question: **"When exactly should I use Node JS ?"**.

To understand this we need to see how Node works !

## Understanding the way a Node app runs.

First of all, Node JS is not a server. In the sense, Node JS doesn't serve anything directly. One needs to create an application server instance using other JS libraries. Hence, it basically is Javascript with added features to support majority of the core backend needs like access to filesystem, OS system calls ,etc. The magic happens at the V8 JavaScript Engine which runs the node program.

### Process model

Node JS follows event based programming. This is the absolute core of Node. It's a single threaded program that acts upon callbacks and never blocks on the main thread.

Hence, instead of spawning a new thread for every new user, node loops through events of all the users in a single thread itself. This avoids thread creation overhead. So, ideally the main thread only listens for events and nothing else. It never gets blocked because of the way node forces you to write code. The code you write should always be asynchronous and callbacks driven.

Take a look at the diagram below:
<br>

<div style="text-align:center">
  <img src="/blog-atul/assets/event-loop/eventloop.png" style="width: 80%;display: inline;">
</div>

<br>

Events can be any action, like HTTP request or even callback from any I/O operation for which the application was waiting. The event loop  processes every event that comes up in the event queue. Hence, whenever an event occurs, the event is processed and the associated callback is called. This is why we hear devs say: *Node JS never blocks !*

Now, there are two types of applications based on how they behave.

1. **CPU intensive** - These applications require large amount of cpu cycles for their operations. For example : Heavy mathematical calculations, data crunching , Image processing ,etc.
2. **I/O intensive** - These applications perform mostly Input/Output operations. For example: Database access, file reading and writing, responding to HTTP requests and so on.

**CPU intensive** applications are best suited for multi threaded systems, obviously because they will be performed by multiple cores simultaneously. But turns out node doesn't work that way. Node based apps are single threaded. Hence, a single thread will need to perform all the cpu intensive jobs which will make it very slow. This is the reason if you have ever tried building image processing apps on node, you would have been pretty disappointed at the speed.
<br>

<div style="text-align:center">
  <img src="/blog-atul/assets/event-loop/node_cpu_intensive.jpg" style="width: 40%;display: inline;">
</div>

<br>

While, **incase of I/O operations**, node works really well.
This is because I/O operations in node are not carried out in the main thread. Most of the I/O is written as non blocking asynchronous. But certain background I/O are insanely difficult to write in asynchronous manner. For those  I/O apis are executed in a separate  threads and processes for DB access and process execution. Also, certain file operations are performed by OS. Internally, Node JS relies on `libev` to provide the event loop, which is supplemented by `libeio` which uses pooled threads to provide asynchronous I/O.

Thus the main even loop only focuses on handling the events and not the associated operations, making the response time very fast.

## Final verdict!

Due to the way NodeJS works we can come to the conclusion that :

- **Potential I/O intensive application** - Node JS might be the best choice  &nbsp; &nbsp; <span style="font-size:25px;color:green">✔</span>


- **Potential CPU intensive application** - Node JS will be very slow  &nbsp; &nbsp; <span style="font-size:25px;color:red">✘</span>

  Go for multithreaded systems like Java.


As most of the web applications are not CPU intensive in nature, **Node still wins** and is my first preference !!!

<br>

<div style="text-align:center">
  <img src="/blog-atul/assets/event-loop/nodejs.gif" style="width: 40%;display: inline;">
</div>

<br>

### Credits
- https://dzone.com/articles/quick-introduction-how-nodejs

- http://blog.thedigitalgroup.com/ujwalap/2015/05/15/introduction-to-how-node-js-works/

<br />
<br />
<hr />
<br />
{% if page.comments %}
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url= http://www.atulr.com/blog-atul/web/2016/11/05/nodejs-usage.html;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = NODEJS_EVENT_LOOP; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = '//atulr.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>

{% endif %}
