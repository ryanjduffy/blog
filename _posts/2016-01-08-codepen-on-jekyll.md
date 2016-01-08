---
layout: post
title: CodePen on Jekyll
categories: []
tags: [Jekyll, CodePen]
published: True

---

As part of getting this blog up and running, I'm sharing a few of the things I've set up for others
doing the same. This installment is the rather simple [CodePen](http://codepen.io) include to embed
pens in my posts. I took it a step further to allow including any pen by `hash` and `username` with
the latter defaulting to the `codepen_username` value in the site's `_config.yml`.

Below is the source (as of this writing) or you can see the [latest on Github.]
(https://github.com/ryanjduffy/blog/blob/gh-pages/_includes/codepen.html).

{% raw %}
```liquid
{% assign username = include.username %}
{% unless username %}
	{% assign username = site.codepen_username %}
{% endunless %}
<p data-height="228" data-theme-id="0" data-slug-hash="{{ include.hash }}" data-default-tab="result" data-user="{{ username }}" class='codepen'>See the Pen <a href='http://codepen.io/{{ username }}/pen/{{ include.hash }}/'>{{ include.title }}</a> by {{ username }} (<a href='http://codepen.io/{{ username }}'>@{{ username }}</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>
```
{% endraw %}

## Examples
{% raw %}
```liquid
{% include codepen.html hash="WQjmGo" title="Enyo Template" %}
```
{% endraw %}

{% include codepen.html hash="WQjmGo" title="Enyo Template" %}

{% raw %}
```liquid
{% include codepen.html hash="YXxmYR" username="Yakudoo" title="Top Pen of 2015" %}
```
{% endraw %}
{% include codepen.html hash="YXxmYR" username="Yakudoo" title="Top Pen of 2015" %}

