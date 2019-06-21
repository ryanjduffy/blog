---
layout: post
title: 'Enyo Daily #23 - Memorable Panels'
date: 2012-07-07 20:49 -0700
---

<p>This is very much beta code but wanted to share how to add history to the Enyo2 Panels control.  For flavor, I even added html5 history support so if you're using a Panels controls as the main view controller (like Pane in Enyo1), you get quick back button support.</p>

<p>Here's the code as well as a quick example (<a href="http://fiddle.jshell.net/ryanjduffy/LnMmL/show/light/">full screen</a> and in <a href="http://jsfiddle.net/ryanjduffy/LnMmL/embedded/result/">jsfiddle</a>):</p>



<pre><code>enyo.kind({

    name:"extras.Panels",

    kind:"enyo.Panels",

    published:{

        history:""

    },

    create:function() {

        this.historyChanged();

        this.inherited(arguments);

        this.createComponent({kind:"Signals", onBack:"back", onpopstate:"statePopped"});

        enyo.dispatcher.listen(window, "popstate");

    },

    historyChanged:function() {

        this.history = this.history || [];

    },

    back:function() {

        this.history.pop(); // current

        if(this.history.length &gt; 0) {

            var last = this.history.pop();

            this.setIndex(last);

        }

    },

    indexChanged:function() {

        this.inherited(arguments);

        if(this.history[this.history.length-1] !== this.index) {

            this.pushState();

        }

    },

    pushState:function() {

        if(window.history.pushState) {

            var c = this.getControls();

            window.history.pushState({index:this.index}, "", "#"+c[this.index].name);

        }

        this.history.push(this.index);

    },

    statePopped:function(source, event) {

        if(event.state) {

            this.back();

        }

    }

});</code></pre>
<p>If you're looking to support other browsers, you might check out <a href="https://github.com/balupton/history.js">history.js</a> which appears to be a nice polyfill for the history API.</p>