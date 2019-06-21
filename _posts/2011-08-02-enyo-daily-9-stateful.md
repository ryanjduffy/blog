---
layout: post
title: 'Enyo Daily #9 - Stateful'
date: 2011-08-02 09:29 -0700
---

<p><p>I came across this interesting kind while investigating the Picker control.  It's pretty basic -- 1 property and 1 method (+1 protected) -- but can be a useful base class to streamline the styling of a control.</p>
[[MORE]]
<p>Stateful's single method is setState() which expects attribute as a string and its state as a boolean.  This method in turn adds or removes a css class with the attribute name as the suffix.  The prefix for the class name is set using the single published property of Stateful -- cssNamespace.  The default cssNamespace is "enyo" but I'd recommend changing it to something unique to your application.</p>
<p>While you could create a Stateful kind directly, it's designed to be used as a base class.  For example, CustomButton (which is one of the base kinds for the Picker control), publishes several additional properties; each of the changed handlers for these properties call Stateful.setState to style the control appropriate for the change in property value.</p>
<p>An example will help illustrate.  For a further example, take a look at the source for CustomButton.</p>
<p><strong>CSS</strong></p>
<blockquote>
<p>.extras-active {<br>  background-color:red;<br>}</p>
</blockquote>
<p><strong>JavaScript</strong></p>
<blockquote>
<p>enyo.kind({<br>  name:"StatefulButton",<br>  kind:"Stateful",<br>  cssNamespace:"extras",<br>  published:{<br>    active:false<br>  },<br>  activeChanged:function() {<br>    this.setState("active", this.active);<br>  },<br>  clickHandler:function() {<br>      this.setActive(!this.active);<br>  }<br>});<br><br>enyo.kind({<br>  name:"com.technisode.example.App",<br>  kind:"VFlexBox",<br>  components:[<br>    {kind:"StatefulButton", active:true, content:"Button 1"},<br>    {kind:"StatefulButton", active:false, content:"Button 2"},<br>    {kind:"StatefulButton", active:false, content:"Button 3"}<br>  ]<br>});</p>
</blockquote></p>