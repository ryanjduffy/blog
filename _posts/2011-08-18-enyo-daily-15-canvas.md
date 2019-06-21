---
layout: post
title: 'Enyo Daily #15 - Canvas'
date: 2011-08-18 09:21 -0700
---

<p><p>Today's been an interesting day for webOS.  If you're a webOS developer (which is probably a safe assumption if you're reading this), you're probably already aware of HP's decision to jettison its hardware division.  As a result, there are in effect no webOS phones or tablets to be released unless they find a suitable hardware partner.  According to the fine folks over at <a href="http://thisismynext.com/2011/08/18/hp-not-walking-away-webos-exclusive-details/" target="_blank">This Is My Next</a>, HP isn't walking away from webOS.  For now, I'll buy into that and continue to create things for a platform I believe deserves to thrive.</p>
<p>That said, I've taken time off from posting due to the birth of my daughter (our third child).  As a result, I haven't taken much time to plan out the next couple posts.  But, I wanted to get something out today so I'll cover how to get started with the HTML5 canvas tag in enyo.</p>
[[MORE]]
<p>Enyo doesn't provide much for developing in canvas; it doesn't really need to.  Canvas in enyo is the same as in normal web development.  There are a couple minor hurdles you have to overcome, however.</p>
<p>First, how do you create a &lt;canvas&gt; tag when all enyo controls are defined in JavaScript?  The undocumented nodeTag property, of course!  DomNode includes this property and uses it to determine the string to pass to document.createElement.  So, to create an enyo canvas control, set the nodeTag property to "canvas"!</p>
<blockquote>
<p>{name:"myCanvasTag", nodeTag:"canvas"}</p>
</blockquote>
<p>Second, how do you get a reference to the drawing context to draw on the canvas.  Like other enyo controls, the DOM node doesn't exist until the control is rendered.  The simplest solution is to override rendered() in the control containing the canvas and call your drawing methods there.</p>
<blockquote>
<p>rendered:function() {<br>    var n = this.$.myCanvasTag.hasNode();<br>    var ctx = n.getContext("2d");<br>    ctx.fillRect(0,0,100,100);<br>}</p>
</blockquote>
<p>Here's a complete example that creates 5 canvas tags using VirtualRepeater.</p>
<blockquote>
<p>var _App = {<br>    name:"com.technisode.example.App",<br>    kind:"Control",<br>    components:[<br>        {name: "repeater", kind:"VirtualRepeater", onSetupRow:"setupRow", components:[<br>            {name: "canvas", nodeTag:"canvas"}<br>        ]}<br>    ],<br>    rendered:function() {<br>        this.inherited(arguments);<br>        <br>        for(var i=0;i&lt;5;i++) {<br>            // this is a protected method so ... palm may not like you using it ...<br>            this.$.repeater.prepareRow(i);<br>            var n = this.$.canvas.hasNode();<br>            n.width = 150;<br>            n.height = 150;<br>            n.getContext("2d").strokeText("I am item " + i, 10, 10);<br>        }<br>    },<br>    setupRow:function(source, index) {<br>        if(index &lt; 5) {<br>            return true;<br>        }<br>    }<br>}<br><br>enyo.kind(_App);</p>
</blockquote></p>