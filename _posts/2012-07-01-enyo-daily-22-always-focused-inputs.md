---
layout: post
title: 'Enyo Daily #22 - Always Focused Inputs '
date: 2012-07-01 20:51 -0700
---

<p><p>In enyo v1, making an input control appear focused was easy: simply set alwaysLooksFocused to true.  In enyo v2, the styling is delegated to an InputDecorator but it doesn't provide this same property.  Fortunately, it's easy to add to a custom subkind.</p>

<pre><code>enyo.kind({

	name:"extras.InputDecorator",

	kind:"onyx.InputDecorator",

	published:{

		alwaysLooksFocused:false,

	},

	create:function() {

		this.inherited(arguments);

		this.alwaysLooksFocusedChanged();

	},

	alwaysLooksFocusedChanged:function() {

		this.addRemoveClass("onyx-focused", this.alwaysLooksFocused);

	},

	receiveBlur:function() {

		if(!this.alwaysLooksFocused) {

			this.inherited(arguments);

		}

	}

});</code></pre></p>