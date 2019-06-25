---
title: Squashing Commit Histories
layout: post
---

I've encountered a couple scenarios in which I wanted to squash the git history of a particular
branch.

<!--more-->

The most common is to prepare a branch for a pull request so that I'm only merging a single
commit for the feature or bug fix. I used to rebase the branch on my target (e.g. `master`) but that
often led to multiple rounds of conflict resolution as each commit was rebased. I've since changed
to first interactively rebasing the branch on the merge base commit so I could squash my work into
a single commit and then rebase the result of my target branch. Assuming that my feature branch has
already resolved any potential merge conflicts with my target branch, the rebases are both clear of
conflicts and the result is a nice, clean single commit of my changes.

```bash
git rebase -i $(git merge-base HEAD master) && git rebase master
```

Because I am often reviewing other engineers' code, I've included this within my `.gitconfig` to
simplify my workflow for prepping code to be cleanly merged.

```
[alias]
squash = "!f() { git rebase -i $(git merge-base HEAD $1) && git rebase $1; }; f"
```

> *Note:* GitHub includes this feature to its UI but it's sometimes useful to do manually (or if
> you're using something other than GitHub!).

A second scenario arose as a result of the first! I had two features in different stages of
development and the second was based on the first. When the first was merged into `master`, it was
squashed down to a commit. My second branch still had the original commits of the first branch so I
didn't want to merge those into master but, more importantly in this case, I didn't want my
coworkers trying to sift through which commits were relevant to the change.

In this case, the commit history wasn't necessary for review so I created a new branch based on
master and `git apply`-ed the changes (from `git diff`) from my second feature branch. The result is
a set of unstages changes representing the difference between master and my feature branch.

```
git branch feature-branch-merge master
git checkout feature-branch-merge
git diff master feature-branch | git apply
git commit -m 'add new feature'
```
