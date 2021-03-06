---
layout: post
title: 'Enyo Daily #20 - Frankenstein and Enyo'
date: 2012-05-18 20:53 -0700
---

<p><p>CSS transitions are a really easy way to add basic animation to an enyo app.  It works on (seemingly) any CSS property and takes JavaScript out of the picture.   With Pirate's Dice (which is still not fully functional ...), I'm using them to zoom the background when starting a new game.  I'm also trying to support multiple resolutions (phones in portrait and landscape, tablets, and desktop) using media queries.[[MORE]]</p>

<p>To accomplish this, I'm scaling the background image differently for smaller screens than larger.</p>

<pre><code>

.animate {
    -webkit-transition:all 500ms ease-out;
}

/* portrait */

.app {
    background:#3e140c url(../image/bg.png) no-repeat 50% 100%;
    -webkit-background-size:auto 100%;
}

.app.new-game {
    -webkit-background-size:auto 200%;
}

/* landscape */
.app {
    -webkit-background-size:100% auto;
}

.app.new-game {
    -webkit-background-size:200% auto;
}
</code></pre>
<p>The consequence of this was that changing orientations of the phone would cause WebKit to animate the width from 100% to auto along with the height from auto to 100%.  This turns out to be a rather jarring experience.  To avoid this, I added a sprinkle of script to the resize handler that removes the animate class when resize starts and readds it once its complete.  enyo.job does the work of ensuring reanimate only runs once.  In other words, it won't be constantly added and readded as the window is resized such as on a desktop browser.</p>

<pre><code class="javascript">
resizeHandler:function() {
    this.removeClass(“animate”);
    enyo.job(“frankenstein”, enyo.bind(this, “reanimate”), 200);
},
reanimate:function() {
    this.addClass(“animate”);
}
</code></pre></p>