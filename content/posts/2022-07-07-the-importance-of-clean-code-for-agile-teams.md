+++ 
draft = false
date = 2022-07-07T19:56:00+02:00
title = "The Importance of Clean Code for Agile Teams"
description = ""
slug = "2022-07-07-the-importance-of-clean-code-for-agile-teams"
authors = ["Marcus Ilgner <mail@marcusilgner.com>"]
tags = ["software craftsmanship"]
categories = []
externalLink = ""
series = []
+++
# The Importance of Clean Code for Agile Teams

Most everyone who works in software development will know the situation
where you have written some code that manages to somehow solve a current
problem - but when looking at it the next day (or showing it to another 
person), it turns out that there are a couple of design issues with it.

Now the question automatically arises: should we merge it or give it a
bit of love and clean it up?

Personally I'm in the latter camp. This is not only driven by an aesthetics-
driven claim to high-quality code, but is also informed by business and the
requirements of an agile environment.

## Requirements change; well-crafted code can change

In agile software development, we have learned to acknowledge the fact that
requirements change. And if it were possible to anticipate these changes,
then probably they'd be there already. So, do yourselves a favour and 
[stop predecting the future of your code](https://medium.com/swlh/stop-predicting-the-future-of-your-code-e0d403116f90). 
If it were possible to build workable abstractions that will last the
lifetime of the application at the start of the project... well, it's
possible there wouldn't have been a cause to write the Manifesto for Agile
Software Development in the first place.

TIL that Robert C. Martin apparently proposed to add a fifth value to it: ["Craftsmanship over Execution"](https://medium.com/tacta/software-craftmanship-writing-code-for-the-future-a2caf87312e5).

It is my interpretation that this is because well-crafted code constitutes a
collection of composable pieces that enable you to react to changes. You can
easily re-arrange and adapt them to suit new requirements and build new features
upon them - thereby allowing you to speed up development as time goes on.

The fastest execution is only worth so much if it leads to a mountain of
technical debt, slows you down and makes you think of spaghetti every time
you open a file in your code base.

## Maybe an issue with OOP?

It is my hypothesis, that a large part of this problem is due to the
built-in [Gorilla-Banana-problem](https://medium.com/codemonday/banana-gorilla-jungle-oop-5052b2e4d588)
of object-oriented programming languages. But unfortunately I don't have
enough experience working with larger production-grade applications written
in a functional style to confirm this suspicion. 

## It's a process

The craft of software development is a complex one and there never comes a
point where one wakes up one morning and suddenly finds that they know all
there is to know about it.

That is why the [Manifesto for Software Craftsmanship](http://manifesto.softwarecraftsmanship.org/),
picking up on the Agile Manifesto and extrapolating on it,
mentions the *community of professionals*: software development is such
a broad topic, that everyone has something to learn and we should strive to,
continuously, both teach and to learn from another.

So, let's everyone do their best and [avoid breaking windows](https://medium.com/@learnstuff.io/broken-window-theory-in-software-development-bef627a1ce99)
during stressful phases.