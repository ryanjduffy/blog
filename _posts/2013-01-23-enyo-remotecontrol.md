---
layout: post
title: enyo.RemoteControl
date: 2013-01-23 20:42 -0700
---

<p>Wanted to share a preview of a new kind I've been working on. The idea stemmed from a recent patch I submitted to fix remote loading of js into enyo. A prime use case for that capbility is to deliver a mobile website in which the entire site could be integrated into a single enyo app without loading the entire source on load. Each view could be managed by a parent enyo.Panels and loaded as requested.

To support this idea, I created a new kind called enyo.RemoteControl. In short, it has a load() method that retrieves a remote JS file and attempts to instantiate the specified kind as a child component. To configure what to load, the kind has 2 design-time parameters: href to specify the URL and remoteKind to specify the "kind" to create. It also supports fit which tells the RemoteControl to apply the enyo-fit class to the new child. Finally, it provides a scrim and spinner which displays while the script is fetched and the kind is rendered. This can be overriden through the scrimKind member.

```
enyo.kind({
	name:"enyo.RemoteControlScrim",
	kind:"onyx.Scrim",
	classes:"onyx-scrim-translucent",
	style:"text-align:center",
	showing:false,
	components:[
		{kind:"onyx.Spinner", style:"position:absolute;top:50%;margin-top:-30px;display:inline-block"}
	]
});

enyo.kind({
	name: "enyo.RemoteControl",
	kind: "Control",
	loadState: 0,
	scrimKind:"enyo.RemoteControlScrim",
	published:{
		animate:true
	},
	events:{
		onLoad:"",
		onLoadFailed:""
	},
	components:[
		{name:"ani", kind:"Animator", startValue:65, endValue:1, onStep:"aniStep", onStop:"aniStop", onEnd:"aniStop"}
	],
	getScrim:function() {
		// defer scrim creation ...
		return this.$.scrim || this.createComponent({name:"scrim", kind:this.scrimKind || "Control", owner:this}).render();
	},
	refresh: function() {
		this.setLoadState(0);
		this.load();
	},
	setLoadState:function(state) {
		this.loadState = state;
		if(this.loadState == 1) {
			this.getScrim().show();
		} else if(this.loadState == 2) {	// what about errors?
			if(this.animate) {
				this.$.ani.play();
			} else {
				this.aniStop();
			}
		}
	},
	load: function() {
		if(this.href &amp;&amp; this.loadState == 0) {
			this.setLoadState(1);
			//enyo.job(this.id+"_load", enyo.bind(this, function() {
				enyo.load([this.href], enyo.bind(this, "loaded"));
			//}), 2000);
		}
	},
	loaded: function(block) {
		// should be null unless my pull is changed to always created the failed member
		if(!block.failed || block.failed.length === 0) {
			this.setLoadState(2);
			this.remote = this.createComponent({kind:this.remoteKind, classes:(this.fit) ? "enyo-fit" : ""}).render();

			this.doLoad({remote:this.remote});
		} else {
			this.setLoadState(-1);
			this.doLoadFailed({failed:block.failed});
		}
	},
	aniStep:function(source) {
		this.getScrim().applyStyle("opacity", source.value/100)
	},
	aniStop:function(source) {
		this.getScrim().hide();
	}
});
```

To see it in action, check out <a href="http://jsfiddle.net/ryanjduffy/EL9j4/">this fiddle</a> which can remotely load several of the enyo samples as well as allowing you to provide your own details.</p>