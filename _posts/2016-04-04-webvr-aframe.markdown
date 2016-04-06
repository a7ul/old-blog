---
layout: post
title: 'When VR and Web come together ! - Web VR and AFrame'
date: '2016-04-05 10:00:37 +0530'
categories: blog
---

**WebVR** is an experimental Javascript API that provides access to Virtual Reality devices, such as the Oculus Rift or Google Cardboard, in your browser. The API is in `Editor's Draft` stage as of today.

# Supported Browsers:
---------------------
- Firefox nightly builds with an Oculus Rift enabler installed
- experimental builds of Chrome.

**AFrame** enables to use markup to create VR experiences that work across desktop, iOS, Android, and the Oculus Rift. Hence instead of trying to make the VR interfaces available as a library , Aframe tries to make the interfaces available as first class citizens of the web.
 AFrame is kindof a wrapper on three.js and WebGL. It helps avoiding the complex APIs of WebGL while writing VR apps.


So, Lets get started !

# Initial setup
----------------
Easiest way to setup the project is to include the following scripts in the head of the document.

{% highlight html %}

<!-- Production Version, Minified -->
<script src="https://aframe.io/releases/0.2.0/aframe.min.js"></script>
<!-- Development Version, Uncompressed with Source Maps -->
<script src="https://aframe.io/releases/0.2.0/aframe.js"></script>

{% endhighlight %}


But if you are like me , you would like to setup a webpack project and include the scripts via npm modules

to use aframe do

`npm i aframe --save`

and then do

`require('aframe')`

This will add the polyfill for a-frame to work.

Now lets do some hello world ....

# Hello World !
----------------
Most of the code we write will be HTML markups

To start with we need to create a blank scene
{% highlight html %}
<a-scene>
</a-scene>
{% endhighlight %}

This will create a blank scene with camera and lights. And also it will setup basic keyboard (W,A,S,D)and mouse controls to look around the scene.

Now lets add a box

{% highlight html %}
<a-scene>
  <a-box color="#E45334" width="2" height="2" depth="2"></a-box>
</a-scene>
{% endhighlight %}

<iframe style="width: 100%; height: 500px" src="https://embed.plnkr.co/30486SppK0q1WWBxU0QS/" frameborder="0" allowfullscren="allowfullscren"></iframe>

##### Try pressing W A S D keys to navigate in the scene.

<br/>
**Transformation** of an entity is achieved by attributes position,rotation and scale similar to that of WebGL.

each of these attributes accept a string in the format "X Y Z".

For example position="25 10 12" means position of the entity is 25units on x-axis , 10units on y-axis and 12units on z-axis

{% highlight html %}
<a-scene>
  <a-box color="#E45334" width="2" height="2" depth="2" position="0 1 1" scale="0.5 1 0.5" rotation="40 20 20"></a-box>
</a-scene>
{% endhighlight %}

<iframe style="width: 100%; height: 500px" src="https://embed.plnkr.co/8eoDsRpiqbXKdgbt8LJQ?p=preview/" frameborder="0" allowfullscren="allowfullscren"></iframe>

# Testing the scene
-------------

To test the scene using google cardboard.
Load up your deployed app/example on a mobile browser.

I am trying it out on an android phone.
Make sure you have installed the google cardboard app on your phone.

Then click on the VR goggles icon on the bottom right.
The scene should load up in the cardboard viewer.


I hope the Web VR kicks off really well. This could potentially lead to really neat user interfaces in the web.


# More details
--------

Check out `http://aframe.io` and `https://aframe.io/faq/` for more info on how to get the most out of Aframe.
