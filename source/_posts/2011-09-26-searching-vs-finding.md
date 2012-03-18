---
layout: post
title: Searching vs Finding
---
I've found that many of the truly interesting and outlook-altering pieces of information I've acquired have come via finding something randomly or pseudo-randomly as opposed to searching it out.

These days the information in question is often a tool, configuration, or piece of code that radically shortcuts an oft-repeated activity.  It has nothing to with luck, or ability to use Google, however.  __The delight in finding the perfect tool or option is often that you never conceived that it could exist in the first place__.

The most stark example I've had recently is in pursuit of my quest to [learn vim](http://yannesposito.com/Scratch/en/blog/Learn-Vim-Progressively/).  In changing over from TextMate to vim many times I would end up pondering really newb questions: how do I cut a line of text? How do I move to the end of a file?  A couple of HI-YAH's of Google-Fu later I've got the answer an I'm trucking along.  Some days, however, I'll be reading something unrelated and all of a sudden learn that __dt followed by a character__ _will delete from the cursor position to the next instance of that character_.  It's weird, but that was a game-changer for me.  Changing out strings became: __dt"__ and then insert and go.

I could also characterize my use of [guard](https://github.com/guard/guard) in this way.  I was at a [Mad-Railers](http://www.meetup.com/Mad-Railers/) hack day awhile back and [Brad](http://twitter.com/#!/listrophy) happened to be using [guard for his rspec execution](https://github.com/guard/guard-rspec) during TDD.  It was a revelation that I could just edit files and tests and it would automatically rerun.  Here's where things get interesting, however.  It had previously never occurred to me to even look for a way to speed up my tests or optimize the time spent manually executing them.  It was a problem I didn't know I had.  After learning about guard I looked through [the list of available guards](https://github.com/guard/guard/wiki/List-of-available-Guards), however, and found some other interesting ones.  These additional plugins for the guard model were also solving problems I didn't even know were bugging me until I found they were solved problems.  

This all came to me this morning as I was watching Ryan Bates' [Railscast on using Spork](http://railscasts.com/episodes/285-spork) to speed up test execution further by preloading the Rails framework and keeping it in memory between tests. 

When worrying about getting a primary goal completed (say, a feature in an application) you don't often think about solutions to one-off problems or annoyances, let alone prioritize solving them.  This is partially why I invest time at the beginning of a project to develop and optimize the code-test-fix cycle, why I obsess over the deployment model when I only have a README, and why I like continuous integration before a test has been written.  Once your process for coding is solid, you can optimize it piecemeal as you __need to__ or as you __find unexpected tools or processes__ that would increase your productivity drastically.

An example: my tests were already running really well under guard, but 15 minutes or install and configuration of [guard-spork](https://github.com/guard/guard-spork) I saw immediate speed gains.
