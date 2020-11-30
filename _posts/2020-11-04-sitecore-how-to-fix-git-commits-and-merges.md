---
layout: post
title: "Sitecore How-To: Fix Git Commits and Merges"
date:   2020-11-04 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: David Guan and Mike Skutta
excerpt: This describes two different methods you can use to correct commits and merges.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

This describes two different methods you can use to correct commits and merges.

## Step-by-step guide

The first way is using revert, where no history is lost and is recommended when you are working with changes that have already been pushed up to remotes. The second way is using reset, where you erase history and is only recommended when working with changes that are local.

### The Revert Way

This method uses git revert. Git revert does not delete anything, but rather creates a new commit that inverses the commit that is specified. https://git-scm.com/docs/git-revert

> Note: This is the recommended method when working with commits/merges that have been pushed up to remotes.

#### Reverting Commits

Reverting commits is pretty straight forward and can be accomplished in most git clients. Here are the git CLI commands:

``` bash
# reverting the last commit
git revert HEAD
 
# reverting specific commit(s)
git revert <commit1 hash> <commit2 hash>
 
# reverting the last 3 commits
git revert HEAD HEAD~1 HEAD~2
 
# reverting the last 3 commits, but do not commit it (you can create a single commit reverting the last 3 commits)
git revert --no-commit HEAD HEAD~1 HEAD~2
```

> Note: If reverting multiple commits, make sure you revert them in reverse chronological order.

#### Reverting Merges

Reverting merges is just like reverting a commit, but there is an additional parameter involved. To revert a merge, you need to specify which parent is the mainline. Typically, parent 1 is what we need to specify. Reverting merges is also not typically supported in git clients; git CLI may be your only option:

``` bash
# cherry pick a single commit
git cherry-pick <commit hash>
 
# cherry pick multiple commits
git cherry-pick <commit1 hash> <commit2 hash> <commit3 hash>
 
# cherry pick a merge
git cherry-pick -m 1 <merge hash>
```

> Note: When cherry picking multiple commits, you should cherry pick them in chronological order.

### The Reset Way

This method uses git reset. Git reset re-writes git history, so it is highly recommended to only use this with local changes. You can also lose work with this method, so use it with caution. https://git-scm.com/docs/git-reset

> Note: Only use this with changes that have not been pushed up to remotes. Re-writing git history that others may have will cause many more issues.

``` bash
# resetting a branch to a specific commit (deleting other commits/merges upstream of the target commit)
git reset --hard <commit hash>
```

> Note: Git reset should not be executed with pending changes in your working tree. Stash any changes you may have before working with reset.

#### Reset Caveat

If you need to push your re-written git history to a remote (this should not be the case if you follow the rules), you will need to execute a forced push. https://git-scm.com/docs/git-push

``` bash
git push -f <remote name> <branch name>
```