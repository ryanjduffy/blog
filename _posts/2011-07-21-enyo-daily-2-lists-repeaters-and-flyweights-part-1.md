---
layout: post
title: 'Enyo Daily #2 - Lists, Repeaters, and Flyweights - Part 1'
date: 2011-07-21 09:34 -0700
---

<p><p>The new approach for list components in enyo has proven to be a difficult topic to fully understand.  I don’t pretend to have a complete grasp but rummaging through the source code has helped me immensely.  Today, I want to cover the magic that is the Flyweight.</p>
[[MORE]]
<p>The Flyweight is appropriately described by the enyo API docs as “a control designed to be rendered multiple times.”  It stands to reason why this kind of a control would be good for lists.  The Flyweight enables you to define a template for a list item and the list renders the Flyweight multiple times – once for each list item.  If you’ve used the VirtualList or VirtualRepeater controls, you’ll know that you don’t have to define a Flyweight to get things running; those controls hide the Flyweight.  Nonetheless, it’s handy to understand how the Flyweight works because its design directly affects the design of the higher level controls.</p>
<p>The most important feature of Flyweights is that unlike most enyo controls where the control has a single top-level node, it may have multiple nodes for a single enyo control.  This is where things begin to get murky for devs.  Consider the following code using the VirtualRepeater.  VirtualRepeater, as the name inplies, will create a node containing the controls declared in its component block for each successful return from onSetupRow.  It accomplishes this by employing the Flyweight.</p>
<p>So, if in setupRow() I configure the repeater to have 10 rows, how would I later modify the text of the fifth row, for example?</p>
<blockquote>
<p>{<br>        kind:”VirtualRepeater”,<br>        name:”myRepeater”,<br>        components:[<br>                {name:”text”},<br>                {name:”button”,kind:”Button”, onclick:”buttonClicked”}<br>        ],<br>        onSetupRow:”setupRow”<br>}</p>
</blockquote>
<p>As always, there are several ways to solve the problem but the one I’ll use relates to setting the “focus” of the Flyweight.  Because the Flyweight has a single component that represents multiple instances in the DOM, you have to tell it on which instance you wish to operate.  VirtualRepeater exposes a method to do such a thing:  controlsToRow(inRowIndex).  Under the covers, this method calls setNode() on the Flyweight to “focus” it on the right instance.  After that, you can reference the components as you would any other in enyo.</p>
<blockquote>
<p>highlightRow:function(rowIndex) {<br>        this.$.myRepeater.controlsToRow(rowIndex);<br>        this.$.text.setClassName(“highlighted-row”);  // invented class for illustration<br>}</p>
</blockquote>
<p>The Flyweight has a nice feature that will focus on an instance in response to an event that occurs on control of that instance. Using our example above, when buttonClicked is called, the instance containing the button will automatically have focus so no additional calls are necessary:</p>
<blockquote>
<p>buttonClicked:function(source, event) {<br>        // no need to call controlsToRow beforehand<br>        this.$.text.setContent(“I’ve been clicked!”);<br>}</p>
</blockquote>
<p>There is a little extra going on in VirtualList and VirtualRepeater to make this work correctly.  While Flyweight will automatically link itself to an instance's node on event and you can explicitly focus an instance using setNode(), this only ties the Flyweight to the node, not the enyo Component.  The List and Repeater components employ a StateManager component to save and restore the state of components when the Flyweight's focus changes.  Take a look in the source of those components as well as the RowServer for detail</p>
<p>To wrap up, here’s a complete example using Flyweight directly to illustrate:</p>
<blockquote>
<p> var _Example = {<br>    name:"com.technisode.example.App",<br>    kind:"Control",<br>    components:[<br>        {kind:"Flyweight", name:"fw", layoutKind:"HFlexLayout", components:[<br>            {kind:"Control", name:"text"},<br>            {kind:"Button", name:"button", onclick:"buttonClicked"}<br>        ]}<br>    ],<br>    buttonClicked:function() {<br>        // the node is automatically mapped to the right instance on event<br>        // but the enyo-managed data (e.g. this.$.text.getContent()) isn't.<br>        // see enyo.StateManger for a mechanism to save/restore component state<br>        console.log("clicked",this.$.text.node.innerText);<br>    },<br>    getInnerHtml:function() {<br>        var h = [];<br>        <br>        // make a bunch of instances of the flyweight<br>        for(var i=0;i&lt;10;i++) {<br>            this.$.text.setContent("I'm instance "+i);<br>            this.$.button.setCaption("Button #"+i)<br>            h.push(this.getChildContent());<br>        }<br>        <br>        // see comment in enyo.RowServer.generateRow()<br>        this.$.fw.needsNode = true;<br>        <br>        return h.join("");<br>    }<br>};<br><br>enyo.kind(_Example);</p>
</blockquote></p>