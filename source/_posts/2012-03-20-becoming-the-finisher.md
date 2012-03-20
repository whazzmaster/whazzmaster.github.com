---
layout: post
title: "Becoming the Finisher"
date: 2012-03-20 09:13
comments: true
categories: hacker-news, discipline
---
Via [Hacker News](http://news.ycombinator.com/item?id=3728664) this morning I read Jacques Mattheij's [The Starter, the Architect, the Debugger and the Finisher](http://jacquesmattheij.com/The+Starter+the+Architect+the+Debugger+and+the+Finisher).  It was a great topic and one that I think about often; for me personally __finishing projects smoothly is my biggest challenge__.

For me, starting projects is a great time to be alive.  I love bootstrapping repeatable builds, testing systems, automatic doc generation, etc.  It's a reflexive response having seen so many projects that were doomed from the word 'go' by developers who named folders any old thing.  Even folks that joined the team within the first year were confused as to where code was located and what (if any) naming conventions existed.

Similarly, I've been writing software for millions of users for ten years now and these days I think intensely about architecture when bootstrapping new components or applications.  Making the appropriate trade-offs between short- and long-term design up front will accelerate development while leaving you open to pivoting or scaling when the time is right.

My problem comes to the final touches of a release.  For example, often when I'm working on a new release of [fitgem](https://github.com/whazzmaster/fitgem) I code up the functionality, write new tests, write inline API documentation as I create new methods, etc.  After I think I'm done I'll take the library for a spin in [Pry](http://pry.github.com) and start coding against the new API functions.  Things will go well for awhile and then I'll hit a snag where I forgot a parameter or had a typo in the code.  Great! My ad-hoc testing revealed something I hadn't included in the unit tests. __AND THEN I RELEASE__.  This is not a good thing, but I often spaz out when I get 95% of the way to finishing, hit a snag, and then fix my issue.  I guess I feel exasperated about how long this thing is taking and then just wig out and push it (WE'LL DO IT LIVE!)

To be a good finisher I need to slow down and restart my release process (as light as it is!) when I hit problems.  I need to suppress the anxiousness when I hit minor problems before pushing live, go back to the top of my checklist, and start it over.  Doing that on small projects like a gem will instill good mental resistance to spazzing for large, important releases.