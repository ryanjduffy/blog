---
layout: post
title: 'Enyo Daily #4 - Box Model'
date: 2011-07-25 09:32 -0700
---

<p><p>The flexible box controls and layouts in Enyo are really just convenience wrappers around the standard CSS3 flexible box model.  If you haven't looked into the CSS method before, I suggest you take a few moments to look into it.  <a href="http://ajaxian.com/archives/css-3-flexible-box-model">Ajaxian has a post</a> that gives you a quick intro; Html5 Rocks has <a href="http://www.html5rocks.com/en/tutorials/flexbox/quick/">another good one</a>.  W3C also has the <a href="http://www.w3.org/TR/css3-flexbox/">draft spec</a> for perusal; there's also an <a href="http://www.w3.org/TR/2009/WD-css3-flexbox-20090723/">earlier draft</a> that appears to match WebKit's implementation a little more closely.</p>
[[MORE]]
<p>There are two ways in Enyo to implement a flexible layout:  FlexBox and FlexLayout.  You could also code the CSS yourself which is completely legal so I suppose there are 3 ways ... but let's pretend you don't want to do that right now.  The key difference between the Box and the Layout is that the former is a Control and the latter a Component.  This means that HFlexBox and VFlexBox are renderable DOM nodes whereas HFlexLayout and VFlexLayout are not.</p>
<p>In reality, all of the relevant code is provided by the Layout components.  The HFlexBox and VFlexBox are simply Controls with layoutKind set to their appropriate layout.  I've found that I use the FlexBoxes when I just need a generic control to layout child controls and the FlexLayouts when I'm using a more complex kind.</p>
<p>One key difference between the "usual" implementation of the flexible box model and enyo's is how it handles the flexed components.  Normally, a node that is flexed will be given its natural space and its portion of the remaining.  In enyo, it only receives the left over space.  It accomplishes this by setting the width of the node to 0px as an inline style.  Here's the comment from FlexLayout:</p>
<blockquote>
<p>// we redefine flex to mean 'be exactly the left over space'<br>// as opposed to 'natural size plus the left over space'</p>
</blockquote>
<p>The result of this is that some components might be forced to overlap if the remaining space isn't sufficient to hold them.  Here's a quick example:</p>
<blockquote>
<p>var _Example = {<br>    name:"com.technisode.example.App",<br>    kind:"Control",<br>    components:[<br>        {kind:"HFlexBox", components:[<br>            {flex:1, components:[<br>                // on a 1024x768 screen, this button will be overlapped<br>                // by the next one because it only gets the<br>                // remaining space<br>                {kind:"Button", width:"500px"}<br>            ]},<br>            {kind:"Button", width:"300px"},<br>            {kind:"Button", width:"300px"}<br>        ]}<br>    ]<br>};<br><br>enyo.kind(_Example);</p>
</blockquote></p>