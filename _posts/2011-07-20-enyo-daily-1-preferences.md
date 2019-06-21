---
layout: post
title: 'Enyo Daily #1 - Preferences'
date: 2011-07-20 09:35 -0700
---

<p>I'm planning to try a new thing here.  I'm relatively active in the <a href="https://developer.palm.com/distribution/index.php" target="_blank">Palm Dev Forums</a> and have pulled together a list of topics that seem to be relatively common.  So, I'm hoping to do a daily (maybe, perhaps every few days ...) post on how to solve those common issues.[[MORE]]</p>
<p>To start, I'm going to cover how to store preferences using the system service.  The <a href="https://developer.palm.com/content/api/dev-guide/enyo/tutorial.html" target="_blank">enyo tutorial</a> covers this briefly but unfortunately inaccurately right now.</p>
<p>The low level approach is to call the system service directly.</p>
<blockquote>
<p>// define the service in the components block<br>{name: "preferencesService", kind: "enyo.SystemService"},</p>
<p>// elsewhere, in create() for example, call getPreferences passing<br>// the desired keys<br>create:function() {<br>    this.inherited(arguments);<br>    this.$.preferencesService.call({<br>        keys: ["pref1", "pref2"]<br>    },{<br>        method: "getPreferences",<br>        onSuccess: "gotPreferences",<br>        onFailure: "gotPreferencesFailure"<br>    });<br>}</p>
<p>// setting the preferences uses a similar call.  you'd execute this<br>// on button click, property change, or view change, for example<br>buttonClicked:function(source, event) {<br>    this.$.preferencesService.call({<br>        "pref1":"value of pref 1",<br>        "pref2":"value of pref 2"<br>    },{<br>        method: "setPreferences",<br>        onSuccess: "setPreferences",<br>        onFailure: "setPreferencesFailure"<br>    });<br>}</p>
</blockquote>
<p>That works perfectly fine but is a little verbose for my taste.  There's also an (undocumented) PreferencesService kind you can use.  With that component, you're insulated from the underlying system service and instead call one of its two methods:</p>
<blockquote>
<p>// passing an object of name/value pairs<br>updatePreferences(inPreferences)</p>
<p>// passing an array of key names<br>fetchPreferences(inKeys, inSubscribe)</p>
</blockquote>
<p>It also has an onFetchPreferences event to which you can listen to be notified when preferences have been loaded.</p>
<p>Finally, I've created a new kind as part of my <a href="http://github.com/tiqtech/enyo-extras" target="_blank">enyo-extras repo</a> called AutoPreferencesService.  This component works by inspecting the published properties of its sub-kind and automatically loading those preferences and auto-wiring change events to update preferences.  This allows you to create a app or feature-specific preferences interface that's more semantic while still insulating the app from directly interacting with the system service.</p>
<blockquote>
<p>// create a kind inheriting from AutoPreferencesService<br>enyo.kind({<br>    name:"myApp.ExamplePrefs",<br>    kind:"extras.AutoPreferencesService",<br>    published:{  // declare your preferences and default values<br>        pref1:"",<br>        pref2:""<br>    }<br>});</p>
<p>// add the new kind to you apps components block<br>{kind:"myApp.ExamplePrefs", onLoad:"prefsReady", name:"prefs"}</p>
<p>// add onLoad callback handler<br>prefsReady:function() {<br>    var pref1 = this.$.prefs.getPref1();<br>    // do something useful with pref1<br>}</p>
<p>// update preference as needed<br>saveButtonClicked:function() {<br>   this.$.prefs.setPref1("new value");<br>}</p>
</blockquote>
<p>This will be my new default way to deal with preferences.  What do you think?  Which approach is your preference or are you dealing with preferences a different way entirely?</p>