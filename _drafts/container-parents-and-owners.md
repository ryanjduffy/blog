---
layout: post
title: Containers, Parents, and Owners
categories: []
tags: []
published: True

---

## Containers and Parents

The container of a control is the nearest ancestor from the external perspective.

The parent of a control is the nearest ancestor from the internal perspective.

Similar to node distribution in the shadow DOM, a control (C) may be defined as the descendant of a
complex, componsite control (Container) which may cause it to be distributed into a private inner
child (Chrome).

* From the creator's perspective, Container is the `parent` of C.
* From the DOM's perspective, Chrome is the `parent` of C.
* From the Container's perspective, it contains C and determimes where it is distributed.

So ... Container is the `container` of C and Chrome is the `parent` of C. The consequence of all of
this is that you can retrieve the controls contained by another using `getControls()`.

> To exclude the private inner children, like Chrome, use `getClientControls()` which will only
> return controls with a falsey `isChrome`

## Owners

Ownership acts independent of containers and parents. Ownership is primarily concerned with which
controls can interact directly with others. Ownership is reflected in the `this.$` hash for a
control which will contain a reference to each control `this` owns. A control should only access
properties and methods on controls it owns.

Ownership defaults to:

* `undefined` when created directly via the kind's constructor, or

  ```
  // doesn't have an owner
  var c = new Control();
  ```

* the control that created it.

  ```
  // owned by this and accessible at this.$.c1
  this.createComponent({name: 'c1', kind: Control});

  // owned by this.$.c1 and not accessible
  this.$.c1.createComponent({name: 'c2', kind: Control});
  ```

When programmatically creating kinds, ownership can be specified to override the default behavior.

```
// owned by this.$.c1 and accessible at this.$.c3
this.$.c1.createComponent({name: 'c3', kind: Control});
```

