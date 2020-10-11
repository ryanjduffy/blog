---
layout: post
title: Including Git History in a Jekyll Post
categories: []
tags: []
published: True

---

Now that I've bought into a Jekyll-based blog running on GitHub Pages, the next step was to surface
the revision history for a post at the bottom so readers can easily see when and what was updated.

<!--more-->

## Retrieving the History

I had a couple thoughts on how I might get the git history onto the post. My first instinct was to
find a plugin for Jekyll to do the trick. I found, however, that [GitHub doesn't support custom
Jekyll plugins](http://jekyllrb.com/docs/plugins/) on deployment so that wouldn't work. I also
briefly considered a commit hook which I think would work but was quickly interrupted by a better
idea: [GitHub's JSON API](https://developer.github.com/v3/)!

The relevant endpoint is the [commits](https://developer.github.com/v3/repos/commits/) for the repo
of interest. By default, this will give you the commits for the entire repository but can be
filtered by path which is exactly what I needed.

```
GET /repos/:owner/:repo/commits?path=/_posts/post-name.md
```

Creating that URL in Jekyll requires stitching together a couple variables from the site's
`_config.yml`.

{% raw %}
```liquid
https://api.github.com/repos/{{ site.github_username }}{{ site.baseurl }}/commits?path={{ page.path }}
```
{% endraw %}

> In my case, `site.baseurl` was the repository name prepended by a slash. If you have a custom
> domain in place, you may need to alter that to your needs.

## Fetching via JS

With the URL built, I'm ready to write up the JS to fetch it and inject it into the DOM. I'll admit
that my pure XHR skills have rusted a bit but I didn't want to introduce a library and the refresher
was good.

```javascript
(function () {
	var url = 'https://api.github.com/repos/{{ site.github_username }}{{ site.baseurl }}/commits?path={{ page.path }}',
		xhr = new XMLHttpRequest();
	xhr.open('GET', url);
	xhr.send();
	xhr.onreadystatechange = function () {
		var commits;
		if (xhr.readyState !== 4 || xhr.status !== 200) return;

		commits = JSON.parse(xhr.responseText);
		if (commits && commits.length) {
			// create the DOM nodes
		}
	};
})();
```

## Into the DOM

I elected for simple string concatenation to build the inner HTML. The UI is loosely based on the
commit history UI from GitHub. I've simplified it to only show the authorship and the first line of
the commit message which links to the commit on GitHub for the curious. Keeping with the relative
time used on GitHub, I pulled in [moment.js](http://momentjs.com).

```javascript
var html = commits.map(function (commit) {
	var author = commit.commit.author.name,
		avatar = commit.author.avatar_url,
		date = moment(new Date(commit.commit.author.date)).fromNow(),
		message = commit.commit.message.split('\n')[0],
		url = commit.html_url;
	
	return '<div class="history-entry">' +
				'<img src="' + avatar + '">' +
				'<a href="' + url + '" target="gh-history">' + message + '</a>' +
				'<span>' + author + ' updated this ' + date.toLocaleString() + '</span>' + 
			'</div>';
});
```

Though it's unlikely to catch me, I decided to santize the author's name and commit message to avoid
any markup that might be there from breaking the UI. Borrowed some [code from SO]
(http://stackoverflow.com/questions/1219860/html-encoding-in-javascript-jquery) and away we go.
Finally, we'll inject that HTML into the DOM. To guard against network/JS failures, the target is
initially hidden and only shown when the HTML has been generated.

```html
<div id="post_history" class="post-history" style="display: none">
	<h3>Revisions</h3>
</div>
```
```javascript
var post_history = document.getElementById('post_history');
post_history.innerHTML += html;
post_history.style.display = 'block';
```

## Conclusion

I'm pretty happy with the result and I think it'll give good visibility into the post. You can see
the [most recent code](https://github.com/ryanjduffy/blog/blob/master/_includes/history.html) for
the include on GitHub. Questions/Comments below or open a PR with a change and see yourself in a
post showing how it's possible to see yourself!