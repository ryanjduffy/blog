---
layout: post
title: 'Enyo Daily #19 - Local Scrims'
date: 2012-02-22 20:53 -0700
---

<p><p>There are two ways to create a scrim in enyo: globally through enyo.scrim or through the enyo.Scrim kind.  In either case, the scrim "fades" the entire view (usually in order to show a modal dialog or a spinner).  Another use case I'm currently experimenting with in Pirate's Dice is a local scrim -- one that only fades a particular portion of the app.</p>
<p>[[MORE]]In my case, I'm using a scrim to disable the bid controls while other players are active.  I've tried a few different approaches (including drawers, dialogs, and hiding the controls) but a scrim seems to be a nice and simple way to handle it.</p>
<p>Making a scrim local is quite easy and mostly relies on a quick CSS customization.  First, create an instance of enyo.Scrim as a child of the control you wish to fade.</p>

<blockquote>
<p>enyo.kind({<br><span> </span>name:"ex.Hideable",<br><span> </span>kind:"Control",<br><span> </span>className:"extras-hideable",<br><span> </span>components:[<br><span> </span>{name:"count", content:"Click Count: 0"},<br><span> </span>{kind:"Button", onclick:"clicked"},<br><span> </span>{name:"scrim", kind:"Scrim"}<br><span> </span>],<br>});</p>
</blockquote>
<p>Notice I declared a className for the Control.  Next, I'll use that class name in css to limit the scrim to the bounds of my Hideable control.  By default, a scrim covers the screen by settings its position attribute to fixed.  To make it local, simply set the position of the parent control to relative and the scrim to absolute and you're set!</p>

<blockquote>
<p>.extras-hideable {<br><span> </span>position:relative;<br>}<br>.extras-hideable .enyo-scrim {<br><span> </span>position:absolute;<br>}</p>
</blockquote>

<p>Finally, to show or hide the scrim, call the appropriate method on the scrim instance.  To illustrate, here's a silly example but it get's the point across.  Clicking on the button increments the count and changes the label.  Once the scrim is active (by clicking anywhere else in the control), clicking the button is blocked by the scrim.</p>
<blockquote>
<p>enyo.kind({<br><span> </span>name:"ex.Hideable",<br><span> </span>kind:"Control",<br><span> </span>className:"extras-hideable",<br><span> </span>components:[<br><span> </span>{name:"count", content:"Click Count: 0"},<br><span> </span>{kind:"Button", onclick:"clicked"},<br><span> </span>{name:"scrim", kind:"Scrim"}<br><span> </span>],<br><span> </span>create:function() {<br><span> </span>this.inherited(arguments);<br><span> </span>this.showing = false;<br><span> </span>this.count = 0;<br><span> </span>},<br><span> </span>toggle:function() {<br><span> </span>this.$.scrim[this.showing ? "hide" : "show"]();<br><span> </span>this.showing = !this.showing;<br><span> </span>},<br><span> </span>clicked:function(source, event) {<br><span> </span>this.count++;<br><span> </span>this.$.count.setContent("Click Count: " + this.count);<br><span> <br></span><span> </span>event.stopPropagation();</p>
<p><span> </span>}<br>});<br><br>enyo.kind({<br><span> </span> name:"ex.App",<br><span> </span> kind:"VFlexBox",<br><span> </span> components:[<br><span> </span>{kind:"ex.Hideable", flex:1, onclick:"toggle"},<br><span> </span>{kind:"ex.Hideable", flex:1, onclick:"toggle"},<br><span> </span>{kind:"ex.Hideable", flex:1, onclick:"toggle"},<br><span> </span>{kind:"ex.Hideable", flex:1, onclick:"toggle"}<br><span> </span>],<br><span> </span>lastRow:null,<br><span> </span>toggle:function(source) {<br><span> </span>if(source === this.lastRow) return;<br><span> </span>if(this.lastRow) {<br><span> </span>this.lastRow.toggle();<br><span> </span>}<br><span> </span>source.toggle();<br><span> </span>this.lastRow = source;<br><span> </span>},<br>});<br><br>new ex.App().renderInto(document.body);</p>
</blockquote></p>