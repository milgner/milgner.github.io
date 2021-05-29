+++
title = "A Bit of Git Part 1"
date = 2021-05-29T14:08:33+02:00
description = ""
slug = "2021-05-29-a-bit-of-git-part-1"
authors = ["Marcus Ilgner"]
tags = ["git"]
categories = []
externalLink = ""
series = ["A Bit of Git (Each Day)"]
+++

# A bit of git (each day) - Part 1: Basics

Since its inception in 2005, [git](https://git-scm.com/) has quickly
established itself as the de-facto standard for version control in software
development.

Yet I find that surprisingly many developers struggle with using it to its
full potential even after many years of having been in contact with it.
There is an excellent [book "Pro Git"](https://git-scm.com/book/en/v2/)
available online but for some reason not many people are familiar with its
contents or are lacking some focussed exposure to the many aspects therein.

This series is my personal endeavour to rectify this by providing a number
of short articles to explain both concepts behind git as well as practical
tips, so that by spending just a few minutes each day, you'll soon have all
the knowledge you need to competently employ git in your daily work.

To be on the safe side, I won't assume any prior knowledge and start with a
few basics. If you already had practical exposure to git, you'll probably know
most if not all of this. In that case, feel free to skip ahead and come back
tomorrow to read about another aspect of working with git: branching
and branching strategies.

## What's a (distributed) version control system?

Git is a distributed version control system. Essentially, a version control
system is a piece of software that helps you to keep track of changes to one
or more files and, when working in a team, coordinate the changes made by the
members of that team so they can change version-controlled files without
accidentally overriding or deleting someone elses work.

_Distributed_ means that the whole set of information about these tracked
files and changes therein - the _repository_ - is not only kept in one
central place but can be distributed among many places. That usually means
locally on the computers of the people working with it - i.e. you - as well
as a number of servers, called _remotes_. Indeed, any machine that has a copy
of the repository has all the information required to act as a remote, too.
It also means that every action regarding that repository can be executed
without having to consult with another system.

> Creating another copy of a repository is called _cloning_. That's why we
> use the command `git clone <repository-url>` to start working with code
> from services like Github, Gitlab and others on our machines.

### Gitting started (SCNR)

If you still have some time on your hands and are looking for some commands
to do some hands-on experimentation, I refer you to [chapter 2](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository)
of the aforementioned book Pro Git where you'll find just that.

But from my side, that's all for today - keep learning and see you tomorrow!
