---
layout: post
title: EnyoJS on Cloud9
date: 2013-05-21 20:39 -0700
---

<p><p>I've been playing around with the Cloud9 IDE a bit recently to do some remote development. It's come a long way from when I first used it and is quite functional. A great addition is an interactive command line with a git client. So I thought, "It'd be nice to get the EnyoJS bootplate up and running here."  Following the <a href="https://github.com/enyojs/enyo/wiki/Dupliforking">general instructions from the wiki</a>, I put together a set of commands that gets everything set up.</p>
<p><b>TL; DR</b></p>
<p>Pull in bootplate, update the submodules, and "disconnect" the master branch from the bootplate repo.</p>

<!--more-->

<pre><code>git clone https://github.com/enyojs/bootplate.git; rm -rf .git; shopt -s dotglob; mv bootplate/* .; git remote rename origin bootplate; git submodule update --init; printf "\n.c9*" &gt;&gt; .gitignore; git add .gitignore; git config --unset branch.master.remote; git config --unset branch.master.merge;</code></pre>

<p>Assuming your pushing this to a new git repo (on <a href="http://github.com">Github</a> or <a href="http://bitbucket.org">Bitbucket</a> for example), add the remote repo and set the master branch to track it.</p>

<pre><code>
git remote add origin <origin path>
git fetch origin; git branch --set-upstream master origin/master</origin></code></pre>

<p>Here's the first command broken up for readability:</p>
<pre><code>
# clone the bootplate repo
git clone https://github.com/enyojs/bootplate.git

# remove the existing git config (it'll be overwritten by bootplate)
rm -rf .git

# move the bootplate contents to the root folder (including the .git directory)
shopt -s dotglob
mv bootplate/* .

# rename the origin branch so your repo can be origin
git remote rename origin bootplate

# update the submodules
git submodule update --init

# add .c9* to .gitignore so the cloud9 revision tracker isn't added to git
printf "\n.c9*" &gt;&gt; .gitignore
git add .gitignore

# make master not track the remote bootplate repo
git config --unset branch.master.remote
git config --unset branch.master.merge
</code></pre></p>