---
layout: post
title: 'Enyo Daily #21 - Enyo Style Tag'
date: 2012-05-06 20:52 -0700
---

<p>Had a random idea to implement a &lt;style&gt; tag in Enyo for global runtime CSS manipulation.  I'm thinking it might be handy to resize a bunch of elements as a result of window resize, for example.  I wrote a quick example in enyo v1 and ported the couple differences for v2.</p>
<p>You can either set the entire CSS via its published css property or set individual blocks via the set(classSpec, styleObject) method.</p>
<p>Thoughts?</p>
<p><em>Edit:</em> Make a quick update to enable removing CSS from a selector by passing a null.  See the example below for removing the border from the children of .app.</p>
<p>Here it is live:</p>
<p><iframe frameborder="0" height="400px" src="http://jsfiddle.net/ryanjduffy/KAT2Z/2/embedded/"></iframe></p>