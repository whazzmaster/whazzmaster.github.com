---
layout: post
title: "Whazzing.com Moved to Github"
date: 2012-03-17 23:09
comments: true
categories: 
---

Two things forced my hand to move my development blog off of my own Rails-based blogging codebase and onto
[Github Pages](http://pages.github.com/) via [Octopress](http://octopress.org/): 

1. My VPS is becoming severely overcrowded with deployed apps; I use it as a testbed for a lot of ideas as well as my [main swearing-at-pop-culture blog](http://www.whazzmaster.com) and I didn't want to host yet another heavyweight rails app to chew through limited resources.
2. I reaaally like the 'one-file-per-post' model of Octopress/Jekyll, and keeping my development blog/presentations right next to my open source code is very appealing.

Since I wrote my latest blog engine to take markdown input it was easy-as-pie to export the existing posts to files and then use them to bootstrap this Octopress site.  For now I'll ride on the default theme and see how it goes.  I hope to spend more time iterating on my presentations and a bunch of C++ code that I'm working on for a work project, so it's also a plus that I don't have maintain the platform source tree anymore.