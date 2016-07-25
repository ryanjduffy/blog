---
layout: post
title: Exploring Mixins
categories: []
tags: [Enyo, Research]
published: True

---

## Why Mixins?

My primary driver for digging deeper into mixins was to flatten the inheritance hierarchy. Enyo v2.6
requires a 4-deep prototype to render a `<div>` (CoreObject -> Component -> UiComponent -> Control)
which is a bit silly, imo. As you go deeper into the framework, particularly in the UI libraries,
things do not get better (e.g. moonstone/Button is 8 deep).

Mixins provide an opportunity to add features to controls without (necessarily) adding depth to the
prototype chain.

## Work to do