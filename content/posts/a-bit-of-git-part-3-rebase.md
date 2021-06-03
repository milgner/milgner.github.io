+++ 
draft = false
date = 2021-06-03T14:45:31+02:00
title = "A Bit of Git Part 3: Rebase"
description = "Everything you ever wanted to know about rebase"
slug = "2021-06-03-a-bit-of-git-part-3-rebase"
authors = ["Marcus Ilgner <mail@marcusilgner.com>"]
tags = ["git"]
categories = []
externalLink = ""
series = ["A Bit of Git (Each Day)"]
+++

# A bit of git (each day) - Part 3: Rebase

Once you are fluent in the use of the basic git commands `add`, `rm`,
`checkout`, `commit` and `branch`, it is time to look at the next most
important bit of it: `rebase`.

In the last excursion on the subject of branching strategies, `git rebase` was
mentioned a couple of times so it makes sense to have a closer look at it.
The most common use case for rebasing of feature branches is to achieve a
clean, linear history. But I find that I also like to rebase when I know that
my changes would bring up conflicts with the target branch of my merge request.
In that scenario, rebase allows me to resolve them myself, making sure that
the end-result doesn't introduce unexpected behaviour.

## How it works

In its most simple and straightforward form, `git rebase` takes one argument
which points to a commit - also called a [*reference*](https://git-scm.com/book/en/v2/Git-Internals-Git-References)
or *ref* for short.
This is usually the name of a local branch (i.e. `main`), a remote branch
(i.e. `origin/main`), a relative reference like `HEAD~4` (a bit dangerous) or
a specific commit id like `a50d31c`.
When started, `rebase` traverses the graph of commits for the first common
ancestor of the branch (or commit) you have currently checked out and the 
one given as an argument.

It then takes all commits from your current branch and, one by one, tries to
apply the changes from these commits to the target branch.

At this point, you are likely to encounter merge conflicts from one of your
commits. Git will stop and give you a chance to resolve these commits. You can
do this by using a [merge tool](https://www.git-scm.com/docs/git-mergetool) or
via traditional `add`, `checkout` and `rm` subcommands.

Once the conflicts have been resolved to your liking, `git rebase --continue`
will continue with the next commit from your branch.
Just in case you find that resolving these merge conflicts is more trouble
than rebasing is worth - i.e. if you're just doing it to get a clean history -
you can `git rebase --abort` the process and everything will be in the same
state as before starting the rebase process. No worries to accidentally break
something while playing around with rebasing!

## Interactive rebase

The real power of rebase comes from the interactive mode. If you're really
interested in producing a well-readable history, this is where the magic lies.

Instead of just automatically applying all the commits since the last common
ancestor of your current branch and the target ref, `git rebase -i <ref>`
brings up your favourite editor with a pre-filled list of all the identified
commits.
By default, they are all prefixed with `pick`, which, if you just saved and
closed that file, would result in the same behaviour as a regular rebase.

But instead of just closing the file, you're free to completely customize the
behaviour of rebase. The most common commands are:

- if you want to omit a commit from the branch (maybe because it has been
  applied separately - i.e. hotfix - already), just remove the line or add
  `drop`/`d` in front
- if you want to merge one of the commits with the preceding one, change `pick`
  to `squash`/`s` (combines the commit messages) or `fixup`/`f` (discards the
  commit message from the second commit while applying the changes)
- if you want git to pause rebase so you can edit a commit, use `edit`/`e`
- sometimes you also might want to reorder commits although this is probably
  not that common a use case...
- you might want to fix the commit message of a commit that wasn't the last
  one (you can fix these with `git commit --amend`). Use `reword`/`r` as the
  command in front of the commit id
- if you find that the list of commits is somehow not what you expected and
  decide that you rather abort the rebase process before doing anything, just
  remove everything from the file and save it


There are a few more possibilities and interactive rebase can also be used to
split commits - but I'll stop here for now as the whole point of the series is
brevity and I'm already stretching things a bit.

As usual, if you are looking for more in-depth explanations or use cases, I
encourage you to check out the [official Pro Git book on rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing).
