---
layout: post
title: ViewState Framework
date: 2012-11-14 20:44 -0700
---

<p><b>Update:</b> Latest code available on <a href="https://github.com/tiqtech/enyo-extras/tree/master/source/util">github</a></p>

<p>The view state framework is a relatively simple but extensible mechanism to manage the state of view controls. The definition of view control is loose but is intended to represent any control on which a user might expect to be able to "go back" to a previous state. The framework is not a view controller in MVC fashion but can be adapted to that purpose. I've done just that by extending the enyo.Panels control.

<!--more-->

</p><p>The view state framework has 3 components: enyo.ViewState, an implementation of ViewStateStrategy, and the consumer control.

</p><p>ViewState is the consumer API for the framework and consists of a single property, path, and two events, onSaveState and onRestoreState. Each instance of ViewState must declare a unique path (in the traditional /path/to/me format) but all paths need not be at the same depth. The intent is that you could have a view (path:"/home") that contains additional views (path:"/home/items") and their paths would represent that heirarchy. In addition to the semantic value, this also allows the framework to persist and restore states of any parent paths of the saved path. So, if you tell ViewState to save the state of /home/items, it will also trigger a save of /home via the onSaveState. Similarly, if the state of /home/items is restored, the state of /home is also restored (via onRestoreState).

</p><p>The management of view states is done through ViewStateStrategy instances. The ViewStateStrategy kind is basically an abstract base kind. It implements the API for strategies and provides some useful boilerplate code but doesn't store the state anywhere. The base API has three methods: push, pop, and register. As you might guess, push and pop are responsible for saving and restoring state, respectively. The register method associates a path with an instance of ViewState. This registration enables the magic of cascading save and restore described above.

</p><p>To get things started, I implemented a couple ViewStateStrategy instances that work well in the browser: HashViewStateStrategy and HistoryViewStateStrategy. HashViewStateStrategy leverages the window.location.hash to store state and window.onhashchange to trigger state restoration. To accomplish this, it includes a portion of the state data (the key member) in the hash following the path. It then parses the key out onhashchange and passes it back to the ViewState owner in the onRestoreState event.

</p><p>HistoryViewStateStrategy extends HashViewStateStrategy but uses the history API to store the state. This allows for more state data to be saved but still falls back to the key in the hash in absence of that state. This ensures that bookmarking the URL could still restore the application correctly.

</p><blockquote><div>N.B. I tried a couple alteratives that leveraged the component heirarchy before landing on the register method approach. Event bubbling seemed to be the most promising but introduced a couple problems. One, events would bubble up from a ViewState to ancestor controls but couldn't waterfall down to aunt/uncle (to continue the metaphor) controls. It's easier to explain with a picture but, since it didn't work out, I won't bother. Second, it forced the path heirachy to match the control heirarchy. That's probably safe normally but didn't seem like a necessary constraint.</div></blockquote>

<p>The consumer of ViewState has very little work. It must:
</p><ul><li>create an instance of ViewState and  provide it a unique path,
</li><li>call enyo.ViewState.save when it wishes to save the state, 
</li><li>provide a handler for onRestoreState to update the control with the saved state, and
</li><li>optionally, provide a handler for onSaveState if other ViewState instances have a child path defined
</li></ul><p>That's it!

</p><p>More documentation to come but check out <a href="http://tiqtech.com/gallery/viewstate.html">the example</a> to see it in action.

</p><p>Thoughts/suggestions/questions? Let me know here or on twitter <a href="http://twitter.com/theryanjduffy">@theryanjduffy</a>.</p>