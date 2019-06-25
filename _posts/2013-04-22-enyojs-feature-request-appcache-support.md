---
layout: post
title: 'EnyoJS Feature Request: appCache Support'
date: 2013-04-22 20:37 -0700
---

Obviously you can add appCache to existing apps but there are a couple things I think the framework could add here. First, let's get the appcache events wired into enyo's event model.

<!--more-->

```
if(window.applicationCache) {
  enyo.dispatcher.listen(window.application, "updateReady");
  // there are other cache states. updateReady seems to be the most useful one
}
```

Second, let's enhance bootplate to include an appcache manifest in the index.html. Open to naming preferences. Also, provide the default manifest file. Here's a first cut

```
CACHE MANIFEST

# these will be cached
CACHE:
index.html
build/enyo.js
build/app.js
build/enyo.css
build/app.css
assets/*

# these resources will always be pulled from the network
NETWORK:
# /api

# these resources will be loaded if the requested fails
FALLBACK:
# tryFirst.html useOnFail.html
```

Third, let's update the build scripts to (optionally) auto-update root/*.appcache with a build time so the manifest is reloaded whenever the app is built. Perhaps a hash of the contents of the build folder instead?