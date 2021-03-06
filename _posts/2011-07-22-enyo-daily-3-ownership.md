---
layout: post
title: 'Enyo Daily #3 - Ownership'
date: 2011-07-22 09:33 -0700
---

<p>Ownership in Enyo rivals function binding in Mojo for the topic most likely to trip someone up when they start development.  The developer guide has a really good <a href="https://developer.palm.com/content/api/dev-guide/enyo/more-on-ownership-and-containment.html">overview on ownership and containment</a> but I wanted to comment on a couple additional items.[[MORE]]</p>
<p>From a practical perspective, ownership affects 2 things.  First, every control has a component hash (this.$) that contains references to all other components it owns.  The owned components can be referenced by their names on the hash.  Second, when Enyo auto-wires events for a control, it looks for the declared method on the owning object.</p>
<p>There are 2 primary ways in which a component ownership can be established.  The most common approach is declaratively in the components array in a kind definition.  Any component declared in this way is owned by the top level component.  The second approach is programattically via the createComponent or createComponents methods.  These methods are identical except the earlier expects a single object whereas the latter expects an array as its first parameter.  With these methods, if you do not explicitly declare the owner (via the owner member of the passed object), it will default to the caller.</p>
<blockquote>
<p>enyo.kind({<br>  name:"myControl",<br>  components:[<br>    {kind:"Button", name:"myButton", onclick:"clicked"},<br>    {kind:"Control", name:"myText", content:"Some text content"}<br>  ],<br>  create:function() {<br>    this.inherited(arguments);<br><br>    this.createComponent({kind:"Button", name:"newButton", "clicked"});<br><br>    // this onclick handler will not execute<br>    this.$.myText.createComponent({kind:"Button", name:"anotherNewButton", onclick:"clicked"}); <br><br>    this.$.myText.createComponent({kind:"Button", name:"finalNewButton", owner:this, onclick:"clicked"});<br>  },<br>  clicked:function() { /* do something */}<br>})</p>
</blockquote>
<p>In the above example:</p>
<ul><li>Both myButton and myText are owned by myControl using the declarative method</li>
<li>newButton is owned by myControl by default (no owner specified and creatComponent call on myControl)</li>
<li>anotherNewButton is owned by myText by default</li>
<li>finalNewButton is owned by myControl (because owner:this is specified)</li>
</ul><p>All the onclick handlers would be mapped to myControl.clicked() except anotherButton because it is owned by myText.</p>
<p>As a general rule, any components you create should be owned by the top level kind.  In other words, always pass owner:this as part of object to createComponent.  If you find yourself with a reason that a child of your kind should be the owner of the component, you probably would be better off encapsulating the child and the new component into a completely separate kind.  More on that in another post.</p>
<p>A final note on containment:  Containment is primarily concerned with DOM rendering.  If you are working with (big C) Components (meaning they directly inherit from Component, not Control or its sub-kinds), containment doesn't appear to matter.  For Controls, containment directly affects the DOM tree.  So, when you're working on your app's CSS, you'll be able to reference the containment hierarchy to define your selectors.</p>