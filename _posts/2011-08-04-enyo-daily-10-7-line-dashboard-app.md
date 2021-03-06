---
layout: post
title: 'Enyo Daily #10 - 7 Line Dashboard App'
date: 2011-08-04 09:28 -0700
---

<p>Took a couple days off from this series due to a hackathon event I attended.  After writing code for that many hours, I wasn't up to inventing and composing a blog post.  I did have a chance to build an app using jQuery Mobile so perhaps I'll post about that experience sometime in the future.</p>
<p>Back on topic, I have been intending to investigate dashboards in Enyo for a while but hadn't found the time.  I'll go deeper in a future post but wanted to share a novelty app I created in 7 lines of code.</p>
<p>The app doesn't do much, as you might expect in 7 lines of javascript, but it is functional.  To be fair, I'm not counting the boilerplate HTML markup so 7 lines might be a little generous.  Regardless, the app launches directly into a dashboard and increments a counter every time it's clicked.  Exciting, eh?</p>
<p>Without further ado, here it is!</p>
<blockquote>
<p>&lt;!doctype html&gt;<br>&lt;html&gt;<br>&lt;head&gt;<br>    &lt;title&gt;Dashboard&lt;/title&gt;<br>    &lt;script src="/dev/enyo/1.0/framework/enyo.js" type="text/javascript"&gt;&lt;/script&gt;<br>&lt;/head&gt;<br>&lt;body&gt;<br>&lt;script type="text/javascript"&gt;<br>    (function () {<br>        var i = 0, d = enyo.create({kind:"Dashboard", name:"counter"});<br>        d.clickHandler = function() {<br>            d.setLayers([{title:"Click Count",text:"I've been clicked " + (i++) + " times"}]);<br>        }<br>        d.clickHandler();<br>    })();<br>&lt;/script&gt;<br>&lt;/body&gt;<br>&lt;/html&gt;</p>
</blockquote>
<p>If you try this at home, be sure to add "noWindow":true to your appinfo.json.</p>