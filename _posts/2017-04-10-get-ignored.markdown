---
layout: post
title: Using getters in Javascript to build switchable app level configurations
date: '2017-04-10 20:00:00 +0700'
categories: web
published: false
comments: true
---

Objects in Javascript are containers for key:value pairs (properties). They are pretty straight forward.
Lets say we have an object  `const t = {test:123}`;
In order to access the value of test we would do something Like
```
const result = t.test;
console.log(result)
```
Similarly to change t.test , we could write
```
t.test = 456
console.log(t);
```


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
this.page.url= http://www.atulr.com/blog-atul/hack/2017/04/10/get-ignored.html;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = GET_IGNORED; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
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
