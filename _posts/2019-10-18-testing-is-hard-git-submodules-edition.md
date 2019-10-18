---
title:  "Testing is hard: git submodules edition"
description: Git submodules, legacy code, and how things break.
date: 2019-10-18 15:00:00
layout: post
---

Continuous Integration systems are complex. A lot of different tools are involved, including
`git`, Docker, a scheduler, agents scheduling builds, etc. So let's imagine the following scenario:

We have a `git` repository that contains submodules. Our CI system caches `git` operations, so we don't 
always have to clone the repository from the remote. In the few occasions that we do have to clone it,
we need to initialize the git sumbodules. And we want to reset the submodules to their most recent commit.
That's why we run `git submodule foreach --recursive git reset --hard`. 

But one day, we decide to update `git` in our container. It is a minor update so we don't expect breaking
changes. We trigger some test builds and everything seems to work, so we roll out the new container to 
production.

Now all CI builds are using this new container. And one of our users show us a failing build.
The failure is `error: unknown option 'hard'`. We retry the build and it passes, so we assume that this 
is flakiness. Because [flakiness is everywhere](https://www.alexrs.me/2018/testing-is-hard-flakiness).

The day continues and no one complains, but at the end of the day we get more bug reports containing the same
failure. It might not be flakiness after all. We discover that a minor change in
the version number can contain breaking changes.

And we also discover that cache invalidation is one of the hardest problems in Computer Science and it can
hide legit issues during a few hours.

But we already know that automation and testing are hard.

