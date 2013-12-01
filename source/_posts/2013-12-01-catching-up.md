---
layout: post
title: "Catching Up"
date: 2013-12-01 08:45
comments: true
categories:
- general
- year in review
---

Since this summer I've been quite busy both at work and on local projects and
figured it was time to reflect and look towards the upcoming year.

## 2013: The Year in Review

Let's talk about work stuff and home stuff.

### @Work

At work, I started 2013 as primarily a **C/C++ developer** working on connecting
our flagship accounting application to connected services.
It was extremely interesting, if sometimes frustrating, to enhance a 15-year-old
code base to talk to a new generation of more nimble web services. Throughout
the summer we stamped out bugs and responded to early feedback from testers, as
well as worked on merges out to all the major versions of the code base. At the
same time we started prepping documentation and transition materials to ease the
difficulty in transferring knowledge of a 20+ year-old
code base to a new team.

Starting in August I began splitting my time between my desktop development
tasks and my new group. I'm now working on
[ViewMyPaycheck](http://paychecks.intuit.com), which is built on
[Backbone.js](http://backbonejs.org)/HTML5/[SCSS](http://sass-lang.com) for
the front-end and Java Web Services on the backend. It's really my first time
developing in Java, and so I'm learning new things every day.

Luckily, due to my extensive tours of different Javascript frameworks in my
Rails projects at home I was able to come strong out of the gate in working
on the Backbone code. My experience with Sass allowed me to not only pick
up our styles quickly, but re-architect them to ease the evolution and
concurrent development by multiple engineers.

There was some tech debt wrapped up in the new code base but my excellent team
has thus far done a great job of prioritizing refactoring and
build/infrastructure improvements against the race to be feature-complete for
the 2013 tax season.

### @Home

The holiday season of 2012 was quite a disaster in my family's annual gift
exchange. Each adult had to submit a list of things they wanted to my mom, and
then she drew the names for **two** different exchanges: one for our immediate
family and one for our extended family. When Xmas Morning arrived we found a
ridiculous outcome: *since most everyone had gone for the 'easy' thing on their
target's gift list, most people received two of one thing on their list*.

It spoke to an age-old problem of holiday gift-giving where, when grandparents,
aunts, uncles, etc. all clamor for a list of things a kid wants, they have no
way of ensuring that someone else didn't already buy something on the list.
So in the ensuing winter, when I just so happened to have a lot of time on my
hands due to the birth of my son, I set out to create a web app to solve the
problem.

I finished a pre-alpha version of [giftr.us](http://giftr.us) in the spring and
my family used it for birthdays throughout the spring and summer. It's in a
pretty good spot right now, but I wasn't able to finish the formal **Gift
Exchange** functionality for the 2013 Holidays.

This year I also began a journey to help out the
[MadRailers Meetup](http://madrailers.org) which culminated in me taking over
the organization in the fall. I've been a member of the meetup since roundabouts
shortly after I moved back from California, and it's enabled me to meet a ton
of great folks in the Madison tech community. I started out volunteering for
talks in the spring, and in the summer I started suggesting other topics or
speakers.  By the time of the
[Madison Ruby conference](http://madisonruby.com) we'd laid out a great schedule
of speakers throughout the end of the year, and I'm really pleased with where
the group is headed and our new space: the newly-renovated
[Madison Public Library](http://mynewlibrary.org/)!

## 2014: The Year to Come

### @Work

I'm incredibly excited to continue the modernization our Backbone app. A short
list of the changes and/or improvements I'm planning:

* Move to using DevOps-provided [Vagrant](http://www.vagrantup.com/) images on
developer machines to more closely align development and deployment
environments.
* Move our static site code (HTML/CSS/JS/Images) to a CDN to improve load times.
* Move from [Ant](http://ant.apache.org/) to [Grunt](http://gruntjs.com) to
build the site (transpile SCSS, run JS tests, etc.)
* Move from base Backbone.js to [Marionette.js](http://marionettejs.com/) to
get better support for composing layouts, regions, and views.
* Router changes to better control the fetching of data.

These are all technical in nature; we're obviously going to parallelize these
more architectural and platform-ish changes alongside the improvements to the
user workflow and enhancements based on user feedback. My ultimate goal is to
get the app into a continuously deployable build process. We're a ways off right
now but it's definitely doable.

### @Home

I'm going to continue to develop Giftr's gift exchange functionality, and I may
explore either turning it into a single-page app via Ember.js or similar, or
I may dip my toe into mobile native development. I'm incredibly interested
in learning more about [Xamarin](http://xamarin.com/) for doing cross-platform
mobile development.

I'm also going to continue to develop more programming for MadRailers. In 2014
we'd love to explore joint-sponsored meetups with other local tech groups,
as well as move towards more diversity in our speaker lineups and membership.
We'll be sponsoring more Newbie Nights as well as itermittent Hack Days.

## And Sooooo...

It's been a pretty good year! I was somehow able to maintain *some* level of
progress on my own development projects even with the birth of the first kid,
and I'm getting out of my comfort zone in my professional development which
is refreshing. Here's to continuing to learn and improve in 2014!