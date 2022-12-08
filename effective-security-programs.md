---
date: 2018-12-31
tags: ["security", "devops"]
---

# Effective Security Programs

## Table of Contents

- [Effective Security Programs](#effective-security-programs)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Culture](#culture)
    - [Listen to Your Users](#listen-to-your-users)
    - [Hold Office Hours, Community Meetings, Events](#hold-office-hours-community-meetings-events)
    - [Create a Security Champion Program](#create-a-security-champion-program)
  - [Engineering](#engineering)
    - [Invest in Tooling, APIs, Services](#invest-in-tooling-apis-services)
    - [Contribute to Engineering](#contribute-to-engineering)
    - [Practice What You Preach](#practice-what-you-preach)
  - [TLDR](#tldr)

## Overview

As we approach the end of a year and a decade, I decided to write down my
reflections on my experiences in the Information Security (InfoSec) field.

This is also, in part, coming from someone (me) who has struggled with how
security gets done at most places and nearly quit the field (on more than 1
occassion) due to so many bad (at best) and toxic (at worst) security teams and
cultures out there.

It all boils down to this: **Security is a service & you are a service
provider.**

To unravel this, let's explore what effective and successfull Security programs
look like from a culture and engineering perspectives.

If you want the _Too-Long; Didn't Read_ version, skip to the [TLDR](#TLDR)
section.

## Culture

Information Security is not only a technical problem, it is also a people
problem, specifically, security people. Our perceptions and reputations preceed
us, and they're not the good ones either.

We often come off as an
[isolated, egotistical, self-absorbed bunch with a god complex](https://securityboulevard.com/2019/12/seven-toxic-information-security-personalities/).

So, how can we fix our perception and reputation?

### Listen to Your Users

Our job is to enable developers and operations teams to do their work securely
and effectively.

As such, how do we expect to help them without listening to them, without
understanding their business needs, without asking them how we can help them
better?

Before you walk into the office, thinking you're a hero about to save the day,
stop and check your ego at the door:

- Ask your users what is their biggest problem or challenge when it comes to
  security?
  - If the answer is knowledge, then invest in **education**.
  - If the answer is management, then work with project management and
    leadership to demonstrate the **business value** of paying down technical
    debt and improving software quality (_hint: security is a subset of
    quality_).
  - If the answer is technology, then invest in **developer-friendly tooling**.
  - If the answer is culture, then **look in the mirror** and try to see why
    your users avoid you like the plague.
- Ask your users for their feedback on your work. What is working well? What is
  not working well? How would they suggest/recommend you improve? You'll be
  surprised by what they say, one way or another.

### Hold Office Hours, Community Meetings, Events

Most engineers love to learn new topics and concepts, so they could share with
their teams or improve their work.

- Hold weekly, bi-weekly, or monthly lunch-and-learn sessions or office hours to
  dive into a topic of their choosing.

- Put on a quarterly or bi-annually Capture-The-Flag (CTF) competition or a
  hack-a-thon around enabling, improving, or automating security engineering
  (who knows, you might even find a few engineers with a knack or a passion for
  security who might want to join your team?).

This doubles as a means to build or improve relationships with your users.

### Create a Security Champion Program

Not only is security a niche field, it is also a thankless job. Yet, I have
always noticed 1 or 2 engineers on every product development team that always go
above and beyond their work to improve security in their projects, however small
of an improvement it is.

Create a [Security Champion](https://www.owasp.org/index.php/Security_Champions)
program to designate engineers as primariy security points of contact, to
recognize them for their work, and to form a support group of sorts for
engineers who have to carry this mantle. Give them the space to share their
challenges and their learnings with each other and with you. It's not easy being
under constant pressure from project management to deliver on customers'
priorities within tight timelines and do extra-curricular activities.

This Security Champion program also doubles as a bi-directional channel of
communication between Security and Product Development teams that reinforces the
first point I made, i.e. [**listening to your users**](#listen-to-your-users).

## Engineering

As mentioned before, product developers are under project management pressures
to deliver within tight timelines and deadlines. If all we are doing is piling
on, we are failing.

Engineers, on product development teams or operations teams, are often closest
to the product than InfoSec teams are. They likely know more about potential
security concerns or vulnerabilities in their products than we do.

You giving them a security report generated from a vulnerability scanner is the
least helpful thing you can do.

### Invest in Tooling, APIs, Services

It is our job to enable our users to do their work securely and effectively.

This means we should be making their lives easier, not harder.

There is absolutely no reason we should be requiring our users to fill out forms
and submit tickets to get their work done. Instead, we should be enabling them
to be self-sufficient.

For example:

- If you require a threat model, provide tooling to automate generating threat
  models. If no tooling exists, or if existing tooling is insufficient, build
  the right tooling for your business.
- If you require security standards or benchmarks to be followed, provide
  tooling to automate the scanning or analysis of possible misconfigurations.

Here are some folks who are much smarter than I on the subject:

- [Ryan Huber](https://twitter.com/ryanhuber/status/1204492600541171717)
- [Dino A. Dai Zovi](https://twitter.com/dinodaizovi/status/1204758128253898753)

### Contribute to Engineering

Remember that security report generated from a vulnerability scanner?

Here are some crazy ideas, what if you:

- Validate the finding(s), weed out the false-positives, measure and assess the
  risk of each finding against the realities of the product and business needs,
  then work with the respective team(s) to remediate the issue, or even
  contribute the fix yourself?
- Contribute security unit tests to prevent that bug from recurring?
- Offer to peform security code reviews if engineers are working on
  sensitive/critical parts of the system (e.g. user input, certificate
  management)?
- Contribute additional security checks in CI pipelines?

Yes, most of these tasks would require you to operate closely with your users...
As if you're actually one of them; an engineer on their team, but with a
security background.

Now, you finally understand what **DevSecOps** and **Shift Left** truly entail!

### Practice What You Preach

- If you require threat models for every service, then start with your own.
- If you require security bugs to be remediated within an SLA, then start with
  your own bugs.
- If you don't or you can't, then you are in no position to tell engineers to do
  that work.

Plain and simple.

## TLDR

The short version of my rambling is this:

- Security is a Service. You are a Service Provider.
- Fix your culture
  - Listen to your users. Ask them what they need. Ask them for their feedback
    on your work.
  - Hold office hours, community meetings, events to build/improve relationships
    with your users.
  - Create a Security Champion program to designate engineers as security points
    of contact, to recognize their efforts, and to allow them to share knowledge
    with each other and with the security team (going back to listening to your
    users).
- Enable engineers to do their work securely and effectively
  - Automate manual processes. Don't introduce friction.
  - Contribute directly to engineering teams.
  - Practice what you preach.
