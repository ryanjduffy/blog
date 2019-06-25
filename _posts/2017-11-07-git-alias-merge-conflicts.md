---
layout: post
title: 'Git Alias: Merge Conflicts'
date: 2017-11-07 20:44 -0700
---

> Originally posted on Code Mentor: https://www.codementor.io/ryan286/git-alias-merge-conflicts-dmkcw8r3p

Like many engineers, I spend a lot of time working with `git`. It's an incredibly powerful tool with more options than most people ever need. If you work in the CLI like I do, you've probably added a few aliases to help make you more productive. Today, I added a new one to list merge conflicts.

<!--more-->

```
[alias]
    conflicts = "!f() { git status -s | grep ^UU | cut -d ' ' -f 2; }; f"
```

This will give you something like:

```
❯ git conflicts
package.json
src/file.js
src/another.js
```

The value for me is piping it into `git diff` to show only the merge conflicts:

```
git conflicts | git diff --
```

Or opening all the conflicting files in Sublime to reconcile the changes.

```
sublime $(git conflicts)
```

> If you don't have a link set up for Sublime, check out Ashley Nolan's [post on setting it up](https://ashleynolan.co.uk/blog/launching-sublime-from-the-terminal).

_That's it! Do you know a better way to solve this problem? What other git aliases do you use?_