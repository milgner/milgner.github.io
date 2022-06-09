+++ 
draft = false
date = 2022-06-09T22:05:00+02:00
title = "Becoming a better developer"
description = ""
slug = "2022-06-09-becoming-a-better-developer"
authors = ["Marcus Ilgner <mail@marcusilgner.com>"]
tags = ["career"]
categories = []
externalLink = ""
series = []
+++
# Programming isn't software development

While it might be obvious to many or most people, it nonetheless
needs to be said sometimes. But before getting to the interesting
part about how to move from programming to software development
and how to become better at that, let's start with an example of
what I'm talking about. To illustrate the underlying concept, let's
use a completely ludicrous, over-the-top example 🤪

## Programmer's approach to solving tickets

Let's say your team is planning to build a Pomodoro-related app.
The first ticket to be implemented reads

> A blank page with a button labeled "Start".
> Clicking the button opens a dialog that shows the end of the
> pomodoro (25 minutes)

Our imaginary project is only staffed with one (extremely) novice
programmer who will grab that ticket, open `index.html` in their
editor and write

```html
<html>
  <body>
    <a href="#" onclick="alert(`Pomodoro ends at ${new Date(new Date().getTime() + 60*25*1000).toLocaleString()}`)">Start</a>
  </body>
</html>
```

Save, commit, push, create merge request - ticket done! Record time! 🎉

Given the absurdity of this made-up example, it is easy to see what
went wrong here. Interestingly enough, it's the same things that often
go wrong in real-world projects, just on a different scale.

## Doing software development

In order to build and evolve a software over time, before writing any
code, we'll need to come up with a model of the problem that we're
trying to solve. As I'm often writing about Ruby-on-Rails-related
projects, it bears mentioning that sometimes, these models are
isomorphic to a database schema - but more often they just represent
some abstract underlying logic: a `Pomodoro` model and its instance
will still have the same behaviour regardless of whether they're
written to the database or just reside in memory.
At first, our model might only have a starting timestamp and a
constant, class-level duration from which the end timestamp can be
derived.
In the future it might also have an associated `User`, a dynamic
duration, an attribute used to log ones subjective productivity during
the interval - and so on. 

Next, we'll need to ask questions about what might go wrong and
come up with a plan on how to deal - or not deal - with them. What's
supposed to happen if there's daylight savings change during the
Pomodoro? If I start the Pomodoro at 2:40am, would it output that it
ends at 2:05am? What output would we expect?
In all likelihood we will decide that this is an edge case that we
don't care about - but we're still going to document it so in case a
user complains about the time shown being in the past, we'll know that
we discussed this scenario.

Finally, we'll have to remember that the feature we're implementing
right this moment is going to be part of a bigger application which
we're developing. Do we really want to use the `alert` function for
our dialog? There's a good chance it's not going to be the only dialog
that we want to show, so a `showMessageDialog` function would be more
appropriate: even if the first version only uses `alert`, it can be
changed to, e.g., use a Bootstrap Modal component and then all
functionality that wants to show a message dialog will look the same.

Choosing the right level of abstraction, the balance between
re-usable code and [YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)
is one of the most challenging aspects of software development.
If everything goes right - unfortunately it rarely does 😅 - your
software development process will speed up as time goes on since
you're able to quickly build new functionality based on existing
concepts.

# Three things that helped me become a better developer

(so they might help you, too)

## 1. Books

Even if the past two decades have produced a lot of content on
Youtube, online communities and freely available courses on
programming, I still find that the most in-depth knowledge required
for software development is to be found in books a couple of decades
old.

A few stand out in my experience:

### "Code Complete" by Steve McConnell

When I started my first full-time job as a developer, my team lead, as
the first thing on the agenda, gave me this book to read as homework.
And while there was much that I already knew from smaller projects
that I worked on, it still gave me many impulses to evolve my thinking
from programming to software development.

### "Clean Code: A Handbook of Agile Software Craftsmanship" by Robert C. Martin

Another well-known book in the industry, its title probably doesn't
need further explanation. But the aspect of thinking in terms of
craftsmanship and its underlying concepts is a very good frame of mind
to have for successful software development!

### "Domain-Driven Design" by Eric Evans 

Concepts like the ubiquitous domain language and the afore-mentioned
domain modelling are (or should be) at the heart of modern software
development. This book is a fantastic starter to get involved with
these concepts.

### Honorary mention: "The Mythical Man-Month" by Frederick Brooks

This book is less about the implementation aspect itself but about
management of software projects instead. And I am completely
flabberghasted that even though this book is older than me (first
publication in 1975), it's easy to find the same kind of mistakes
outlined in the book still being made in 2022.

## 2. Reading other peoples code

Reading the code of experienced developers can help you transfer some
theoretical concepts that you heard or read about to real-world
software where you'll suddenly recognize them in actual code.

Of course you'll have to match the code that you're going to read to
your individual context. Often, starting with the frameworks and
libraries that you're going to use for your day-do-day job is a good
start to get inspired: be it [Ruby On Rails](https://github.com/rails/rails/),
[Django](https://github.com/django/django) or [Symfony](https://github.com/symfony/symfony) -
for full-stack web developers (plus whatever framework might be
current in the world of JavaScript this week 😜), [Unity](https://github.com/Unity-Technologies/UnityCsReference)
or [Unreal](https://www.unrealengine.com/en-US/ue-on-github) for game
developers - for every kind of application one is going to write,
someone else usually has been there and made high-quality source code
available to learn from.

Among the most inspiring moments of reading source code for me were - 
just to name two - both Ruby On Rails and [git](https://github.com/git/git).

## 3. Katas & multi-paradigm development

Doing [Katas](http://codekata.com/) is a really helpful way to evolve
your development skills. At first you'll start doing them in a
programming language that you're already familiar with, approaching
them from different angles and try to learn from that to improve your
style.

At some point, you might want to try another programming language and
see how it changes the way that you think about the problem and its
implementation in your original language.

I have [mentioned it before](/posts/2022-05-18-pet-projects/) but it
bears repeating: spending some time learning about functional
programming along with a short stint into logic programming at a later
point really helped me boost my "traditional" imperative, OOP
development to a new level.

# The foundation

If you actually read this far, you already have the most important
thing required to become a better developer - the inherent desire
to do so. 😉
If you're looking to challenge yourself and actively look for
inspiration, that dedication is the most crucial aspect to succeed!
