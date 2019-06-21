---
layout: post
title: 'Enyo Daily #6 - Events - Part 1'
date: 2011-07-27 09:31 -0700
---

<p><p>I should probably talk about the design goals addressed by events but that's not a topic I'm up for tonight.  Instead, I'll jump into how events are handled in enyo and defer design talk for another day.[[MORE]]</p>
<p>There are two kinds of events in enyo:  DOM and custom.  DOM events are fired as a result of user interaction such as clicking a control, changing a value in a field, or resizing the window.  Custom events are those defined on an enyo kind that are triggered at the discretion of the component.</p>
<p>Custom events are simpler so we'll start with those.  These events are declared in the kind definition inside the "events" property.  Enyo expects events to be prefixed by "on" and will both warn you and add the prefix itself if it discovers an event without it.</p>
<blockquote>
<p>{<br>  name:"ComponentName",<br>  events:{<br>    onEventName:""<br>  }<br>}</p>
</blockquote>
<p>During the component creation process, enyo will automatically add a function to the component which will "fire" the event.  By default, the function will match the event name with "on" replaced with "do" (doEventName in the above example).  This is actually customizable by specifying an object instead of an empty string as the value following the event name.  For example, if you prefered fireEventName instead of doEventName, use this syntax:</p>
<blockquote>
<p>{<br>  name:"ComponentName",<br>  events:{<br>    onEventName:{caller:"fireEventName"}<br>  }<br>}</p>
</blockquote>
<p>You might be wondering why require an object literal to specify the caller if it's just a string; why not just pass "fireEventName" as the value and skip the object.  The reason is that you can also specify a default handler name for the method.  If the value following the event is a string, it is treated as the default handler name.  If it's an object, it must be included as the value of the "value" property of that object.</p>
<blockquote>
<p>{<br>  name:"ComponentName",<br>  events:{<br>    onEventName:{caller:"fireEventName", value:"handleEventName"},<br>    onAnotherEvent:"handleAnotherEvent"<br>  }<br>}</p>
</blockquote>
<p>If you're using a component that publishes an event, you can react to it by declaring the name of the handler method in the component declaration.</p>
<blockquote>
<p>components:[<br>  {kind:"ComponentName", onEventName:"myEventHandler"}<br>],<br>myEventHandler:function(source) {<br>  // do something<br>}</p>
</blockquote>
<p>If the kind specified a default handler (as illustrated above) and the owning object has a method of the same name, the declaration can be omitted.</p>
<blockquote>
<p>components:[<br>  {kind:"ComponentName", onEventName:"myEventHandler"}<br>],<br>myEventHandler:function(source) {<br>  // do something<br>},<br>handleAnotherEvent:function(source) {<br>  // will be found automatically since it matches the default handler name<br>}</p>
</blockquote>
<p>Finally, here's a complete example.  I've included some DOM event stuff that I'll cover in a future topic.</p>
<blockquote>
<p>var _Example = {<br>    name:"com.technisode.example.App",<br>    kind:"Control",<br>    components:[<br>        {kind:"EventSender", name:"eventSender", onSend:"sent"},<br>        {kind:"Button", onclick:"clicked"}<br>    ],<br>    captureDomEvent:function(e) {<br>        enyo.log("A DOM event occured", e.type);<br>        <br>        // returning true would indicated the event is captured and prevent the bubble phase<br>        // thereby preventing the declared handlers (clicked in this case) from being called<br>        <br>        // returning false (or no explicit return) lets things continue<br>        return false;<br>    },<br>    clicked:function() {<br>        // trigger my custom events<br>        this.$.eventSender.go();<br>        <br>        // toggles event handler between send and secondSent ... just because ...<br>        this.$.eventSender.onSend = (this.$.eventSender.onSend === "sent") ? "secondSent" : "sent";<br>    },<br>    sent:function(source, one, two, three) {<br>        enyo.log("sent", one, two, three)<br>    },<br>    handleOnAlert:function(source, obj) {<br>        enyo.log("alerted", enyo.json.stringify(obj));<br>    },<br>    secondSent:function(source, one, two, three) {<br>        enyo.log("secondSent handles onSend now", one, two, three)<br>    }<br>}<br><br>var _EventSender = {<br>    name:"EventSender",<br>    kind:"Component",<br>    events:{ <br>        onSend:"handleOnSend",<br>        onAlert:{value:"handleOnAlert", caller:"sendAlert"}<br>    },<br>    go:function() {<br>        this.doSend(1,2,3);    // dispatchIndirectly<br>        this.sendAlert({a:1, b:2});<br>    }<br>}<br><br>enyo.kind(_EventSender);<br>enyo.kind(_Example);</p>
</blockquote></p>