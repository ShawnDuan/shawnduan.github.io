---
layout: post
title:  "Most Used Git Operations"
date:   2016-04-12
categories: [Git]
---

Some Git commands/behaviors most used in my daily work.

### About config

```bash
# edit user.name / user.email for current repo
git config user.name "username"
git config user.email sample@gmail.com

git config --list	# list git config
git config --edit	# open git config for editing
```

### Move commit(s) to other branch

* #### Move to a new branch

```bash
git branch newbranch
git reset --hard HEAD~3 	# Go back 3 commits. You *will* lose uncommitted work.*1
git checkout newbranch
```

* #### Move to an existing branch

```bash
git checkout existingbranch
git merge master
git checkout master
git reset --hard HEAD~3 	# Go back 3 commits. You *will* lose uncommitted work.
git checkout existingbranch
```

### Create new branch

* #### Switch to new branch

```bash
git checkout -b new branch
```

* #### Stay in old branch

```bash
git branch new branch
```

### Merge multiple commits into one

```bash
git rebase --interactive HEAD~2		# gonna merge last 2 commits
```

In the editor:

```bash
pick b76d157  b
pick a931ac7  c

# Rebase df23917..a931ac7 onto df23917
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```

change the latter `pick` to `squash`.

```bash
pick b76d157  b
squash a931ac7  c
```

If you want to merge more than 2 commits, change all `pick` to `squash`, *except the first one*.

Save and quit, you'll get another editor whose contents are:

```bash
# This is a combination of 2 commits.
# The first commit's message is:

b

# This is the 2nd commit message:

c
```

You can rename the commits here if you want. Save and quit. Check `git log` to verify.

### Change name of the last commit

```bash
git commit --amend
```


