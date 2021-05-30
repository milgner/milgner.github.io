+++ 
draft = false
date = 2021-05-30T14:54:35+02:00
title = "A Bit of Git Part 2: Branching Strategies"
description = "An overview about Git branching strategies and their properties"
slug = "2021-05-30-a-bit-of-git-part-2-branching-strategies"
authors = ["Marcus Ilgner <mail@marcusilgner.com>"]
tags = ["git"]
categories = []
externalLink = ""
series = ["A Bit of Git (Each Day)"]
+++

# A bit of git (each day) - Part 2: Branching Strategies

After a short introduction to the git basics, we should talk about branching.
If you work on your own, you're free to name your branches any way you like -
but if you try to develop software in a team, that's a clear recipe for chaos.

## What's a branching strategy?

To solve this issue, you'll have to communicate among your team to specify a
branching strategy which defines how to name your branches, how different
branches are merged into another and usually also have a say in which version
of your software gets deployed or delivered to your users.

There are a number of well-established schemes in circulation, each with their
specific advantages and shortcomings. And of course you are free to improve
and adapt these schemes to best fit your workflows - just take care to
document your workflow so you can communicate your chosen strategy to new team
members.

I'll forgo creating fancy graphs for the different models and instead link to
their respective documentation where you can look up any details.

## The original: Git Flow

Git Flow is - to the best of my knowledge the first widely publicized branching
strategy. It was originally published in 2010 in [a blog article](https://nvie.com/posts/a-successful-git-branching-model/)
by Vincent Driessen. It quickly became a kind of de-facto standard and there is
a lot of tooling available - but as the author points out in an amendment to
the original article, it's not a panacea and possibly unsuited in many
situations.

Roughly speaking, git flow starts work from a main branch called `develop`.
From there, most work is done in _feature branches_ (usually named `feature/[short-feature-description]`).
Once you are getting ready to release a new version of the software, one
creates a _release branch_. Bugfixes related to that release are made in
that branch and once the software is deemed ready to release, that release
branch is merged into **both** the `master` and `develop` branches.
If you need to make hotfixes to the software running on production, you create
a _hotfix branch_ which is merged into **both** `master` and `develop` again.
Every time something is merged into `master`, one creates a _version tag_.

### When to use Git Flow

Git Flow is especially useful if:

- you need to maintain multiple versions of your software in the wild
- you have a (possibly) lengthy QA period which tests every feature (i.e. not
  only new features) and need to ensure the software isn't evolving while it's
  undergoing testing

## A simple variant: Github Flow

This is where [Github Flow](https://guides.github.com/introduction/flow/)
comes in.
It's a very simple branching strategy based on one tenet: everything in the
`main` branch is always ready to deploy. Just like in Git Flow, you start
feature branches off of that default branch, but QA of new features is done
by deploying these branches and checking the functionality of these specific
changes.
There is no explicit mention of versioning and since everything in `main` is
always deployable any bugfix is treated like another feature branch.

### When to use Github Flow

Contrary to Git Flow, Github Flow is especially useful if you're running a
SaaS-style application on your own servers: there is only one instance of your
software and Github Flow enables you to focus on fast iteration and delivery
of both features and bugfixes.

## Another competitor: OneFlow

As lined out in the [article explaining OneFlow](https://www.endoflineblog.com/oneflow-a-git-branching-model-and-workflow#when-to-use-oneflow),
it is meant as a drop-in replacement for GitFlow and mostly tries to focus
on the short-livedness of the non-main branches.

The concepts are the same (main, feature, release and hotfix branches, version
tags) but there are also a couple of decisions to make, as there are multiple
ways to finish a feature branch:

- _rebase_: all changes from a feature branch are rewritten before merge to
  produce a linear history. In my opinion the "clean" history graph doesn't
  warrant the effort from rebasing, potentially loosing GPG signatures on the
  affected commits.
- _merge -no-ff_: a feature branch is merged into the main branch, ensuring
  that a merge commit is created even if fast-forwarding (linear history)
  would have been possible.
- _rebase + merge -no-ff_: a combination of the above. It's supposed to negate
  any disadvantages of the first two, but my reservations regarding _rebase_
  still apply.

Originally, OneFlow was born out of a desire to achieve a clean, easily
readable history, which is why the _rebase_-based approaches are still
prevalent with OneFlow users.

There is also a variant of OneFlow which again uses separate `develop` and
`master` branches after which - in my mind - the differences to GitFlow are
neglible.

## There's more! GitLab Flow

So of course that's not all. In order to get a good impression of what kind of
considerations go into choosing a proper workflow, let me also say a few words
about [GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html).

In GitLab Flow, you have so called _environment branches_. These are also
oriented towards development of SaaS-style software where you have one
codebase which is running in different environments like _testing_, _staging_
and _production_. Each of these branches receives its own branch.

It also advocates to reduce merge commits by rebasing your changes.
As mentioned previously, I'm not the biggest fan of rebasing, but it can be
helpful if it's being done manually by the original author(s) as a cleanup
step _before creating a merge request_, as outlined [by this blog article from Atlassian](https://www.atlassian.com/blog/git/git-team-workflows-merge-or-rebase) -
you just have to be willing to invest that effort to get a "nicer" history.

It is my opinion that proper mastery of git can easily compensate for a few
more merge requests.

Another thing that bothers me about GitLab Flow: environment branches are
clashing with my concept of software, where you decide to release a given
state and then run it in any environment you want to.

## Wrap up: my personal take on branching

I already mentioned that I'm not bothered by merge commits and that I'm a fan
of giving released versions of software a version tag.
Automatic rebasing and/or squashing is a big no-go in my opinion because it
looses a lot of information about the history. Especially if you're used to
writing good commit messages, potentially using [Conventional Commits](conventionalcommits.org/).

As such, OneFlow with _merge -no-ff_ is my preferred approach of the ones
listed here. For SaaS-style applications, Github Flow would be good, too, but
you'll have to ensure that it fits your remaining processes like QA and
continuous delivery, too.
