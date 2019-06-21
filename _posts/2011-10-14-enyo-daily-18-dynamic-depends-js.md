---
layout: post
title: 'Enyo Daily #18 - Dynamic depends.js '
date: 2011-10-14 20:59 -0700
---

<p><p><em>While it's probably not fair to call this "daily" any longer, I'm going to stick with it anyway.</em></p>
<p>You may have heard that <a href="http://www.precentral.net/bing-maps-update-brings-enyo-support-webos-masses">enyo has found its way onto the legacy</a> devices via a <a href="http://blog.palm.com/palm/2011/10/a-new-view-of-the-world.html">new Maps application</a> in the App Catalog.  To test things out, I tried to deploy <a href="http://bit.ly/scorekeeper">Score Keeper</a> onto my Pre2.  It deploys successfully but there are definitely some bugs to sort out.  One way I intend to clean up the UI is by including a phone-specific stylesheet through a dynamic depends.js file.</p>
[[MORE]]
<p>The normal depends.js file looks something like this:</p>
<blockquote>
<p>enyo.depends(<br>    "controls.js",<br>    "app.js",<br>    "app.css"<br>);</p>
</blockquote>
<p>This tells enyo to load up controls.js, app.js, and app.css in the same directory as this depends.js file.  But depends.js is just a regular javascript file so there's nothing preventing you from doing something a little more robust here to select which files you want to include.  Also, since the framework is the one including depends.js in the first place, all the framework functions are available during execution of depends.js.</p>
<p>I elected to use a closure to keep the global namespace clear and base my custom CSS inclusion on the screen width.  If you're aware of a better condition to check, please share in the comments.  Inside the function, I retrieve the device info and build an array of the common files.  Next, I check the device width and push the extra css if appropriate.  Finally, I call enyo.depends (via Function.apply since I'm working with an array) and that's all!</p>
<blockquote>
<p>(function() {<br>    var di = enyo.fetchDeviceInfo();<br>    <br>    var paths = ["extras.js",<br>                 "views.js",<br>                 "app.css"];<br>    <br>    if(di &amp;&amp; di.screenWidth &lt; 500) {<br>        paths.push("app-phone.css");<br>    }<br>    <br>    enyo.depends.apply(enyo, paths);<br>})();</p>
</blockquote></p>