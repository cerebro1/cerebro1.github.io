---
layout: post
title:  "Github good practice : Merge Vs Rebase[WIP]"
date:   2022-11-15 16:30:00 +0530
categories: jekyll update
---
<p align="center">
  <h1 align="center">Did you know though they solve same problem but in
completely different way and not interchangeable</h1>
</p>

Hi geeks, I am Sonali Srivastava and this article talks about one of my
practical encounters while making an open source contribution to systemd. So
these maintainers like to maintain a clean repository which is actually good
practice but it can be a nightmare if you are a newbie and do not understand
how git merge is different from git rebase.
Relying on Stackoverflow for help is good. One should know the right question
to ask but running commands without complete knowledge of what it might do can
be destructive. So let's start with the problem.

## what is the problem ?

![github-merge-commit-pr](/assets/github-problem-rebase-merge.png)

As you can see, while pushing my latest changes to the repo, i pulled latest
changes from the main repo and merged my branch so that its latest
but that added an unwanted merge commit which is not preferred in daily
practice. So here the maintainers mentioned the problem as well as the solution
but even though i understood the problem and solution, i could not understand
the impact it would have so i did not run the command until i was sure and thus
this article.

## what is an extra commit ?

![github-extra-commit](/assets/github-extra-commit.png)

For your reference, this is what it looks like. If you are okay with it no
issue but if you are not, well it's time to learn what rebase magic is and how
it is different from git pull or git merge.

## How it happens ?

When you start working on a new feature or some change in a dedicated branch
after a checkout from `main`, you get all the latest changes of main repo in
your branch. Now another team member ( which happens a lot when you work on an
open source project ) updates the main with new commits. Ohh that is when you
realise what git is as a collaboration tool. You don't have those changes in
your branch.

                         |->a->b->c ( Your branch )
                         |
         ->11->12->13->14->15->16 ( Main branch )

From the above representation, you have commits till 14 and then a, b and c.
You do not have commit 15 and 16.

Now there are multiple ways to get those changes.

1. `git merge` : I call it the **easiest non-destructive way** if you are new to
git.

2. `git rebase` : **destructive way** if you do not know what it does behind
the scenes.

Note : It is always a good practice to keep you branch latest unless you are
testing something on an old specific version. There can be updated changes in
the main that could conflict with yours at the time of push.


