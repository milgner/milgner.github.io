+++ 
draft = false
date = 2021-05-20T20:39:10+02:00
title = "Wikis aren't meant for software documentation"
description = ""
slug = "2021-05-20-no-wikis-for-software-documentation"
authors = []
tags = ["Documentation", "arc42"]
categories = []
externalLink = ""
series = ["Software Documentation"]
+++
# Wikis aren't meant for software documentation

> "Documentation is a love letter that you write to your future self."
> -- <cite>[Damian Conway](https://en.wikipedia.org/wiki/Damian_Conway)</cite>

Although there are many arguments being made to rationalize our laziness and
avoid writing proper documentation, there is no doubt in my mind that good 
software documentation is a great tool to improve the development lifecycle
of long-running projects.

If you know your project is going to be scrapped in 2 months time, feel free
to stop reading here and enjoy a good book or a walk outside. :sunny:
But if it's any more than that, you should make your live and the lives of
your co-workers easier by documenting your software. Even projects with a
originally limited lifetime have been known to live long past their best-
before date.

The question is, how to document them. As for the structure, I can recommend
[arc42](https://arc42.org/). But when it comes to implementation details, I
am surprised that many people are still using Wikis to keep their
documentation. 

The [Gitlab Handbook](https://about.gitlab.com/handbook/) has a great section
about how [Wikis don't scale](https://about.gitlab.com/handbook/handbook-usage/#wiki-handbooks-dont-scale)
for handbooks and I have found that most of these arguments also apply when
it comes to software documentation.

The points being made are, essentially:

### 1. There is no review process

Most Wiki software, especially Confluence, which is highly established in the
software development business, doesn't have a proper review process. You can't
easily propose a change and check it for correctness or discuss it with your
colleagues: you need to make the change and the old page gets overridden.

### 2. You can't change or roll back multiple pages

When evolving your software, oftentimes your changes will relate to more than
one page of the documentation. In a Wiki you can only make changes on a page-
by-page basis and changing multiple pages or rolling back all changes in a
changeset is not possible

### 3. Documentation changes are independent of your software development cycle

Ideally you'll update the documentation at the same time as you're making
these changes. Some changes, like [Architecture Decision Records](https://adr.github.io/)
might even precede any actual implementation. So ideally you'd want to wait
with publishing these changes until your code has been reviewed and gone
through any potential improvement cycles. But you can't do that with a wiki,
so you'll either have to document something that isn't live yet or you risk
forgetting to document some aspect because you postpone it until after making
the change.

## What to use instead?

Luckily, there are some alternatives. For one, there's the [docToolchain](https://doctoolchain.github.io/docToolchain)
project which can output a number of formats from a [Asciidoc](https://asciidoctor.org/)-
based source. But as I am under the impression that Asciidoc isn't very
widely adopted in a non-technical audience, I find that I prefer Markdown-
centric solutions - after all, you want to empower everyone on the team to
contribute to the documentation.

Another approach which I have grown fond of recently is the use of static
site generators for documentation and serving these generated sites via
Gitlab or Github pages.

Which static site generator you'll use will probably depend a lot on personal
preference. But there are pre-made themes for [Hugo](https://themes.gohugo.io/tags/documentation/),
[Gatsby](https://gatsbyawesome.com/tag/documentation/) and [Middleman](https://tdt-documentation.london.cloudapps.digital/).

They are usually very easy to set up for either [Gitlab](https://gitlab.com/pages/gatsby/-/blob/master/.gitlab-ci.yml)
and/or [Github](https://gohugo.io/hosting-and-deployment/hosting-on-github/).

And even for private projects, both Github and Gitlab allow configuring your
pages integration to only allow access for logged-in project members.
