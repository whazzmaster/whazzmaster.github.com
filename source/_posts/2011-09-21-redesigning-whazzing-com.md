---
layout: post
title: Redesigning Whazzing.com
---
For the last month I've been working to create this new development-focused blog and project repository for my work.  I was previously self-hosting a [Wordpress](http://www.wordpress.org) blog, but I've developed a desire to own my destiny when it comes to the display of my work.  There were a ton of little features I wanted to develop but didn't really want to delve (back) into the PHP world or hack at the Wordpress template framework.

I'd recently read the excellent [Responsive Web Design](http://www.abookapart.com/products/responsive-web-design) and it made me very excited to design a site that effortlessly works on phones, tablets, monitors, and TVs.  Another project I have in mind is primarily aimed at mobile form factors, so I thought a semi-major experiment was in order to ensure I was thinking in the right direction when it came to media queries and mobile design.

## Technology ##

I built [whazzing.com](http://whazzing.com) with ruby 1.9.2, rails 3.1, mysql, and various gems.  My first commit was August 19th, 2011 and I deployed version 1.0 to production on September 20th, 2011.

As I started coding this site, Rails 3.1 was entering it's final release candidate testing phase so I decided to start fresh on it.  The asset pipeline is just a terrific addition to the framework, and combining it with the style organization I've been developing off of Dale Sande's incredible [Axle framework](http://axle.dalesande.com) provided for a __very smooth visual design experience__.  I added a little bit to Dale's CSS organization to support the media queries I wanted to support.

<pre>
assets
|-- stylesheets
    |-- admin.css.scss
    |-- application.css.scss
    |-- design.css.scss
    |-- devices
    |    |-- phones.css.scss
    |    |-- tablets.css.scss
    |    |-- high-resolution.css.scss
    |-- imports
    |    |-- colors.css.scss
    |    |-- form.css.scss
    |    |-- mixins.css.scss
    |    |-- navigation.css.scss
    |    |-- text.css.scss
    |-- reset.css.scss
    |-- typography.css.scss
</pre>

One of the interesting takeways from the excellent testing panel at [Madison Ruby](http://madisonruby.org/schedule) was that people are divided about the usefulness of [cucumber](http://cukes.info/).  I had always stumbled in wrangling cucumber to be really useful for the applications I was writing and so I decided to eschew its usage this time. I focused instead on getting good coverage via my [RSpec](https://www.relishapp.com/rspec) controller and model tests.  I found that leaving cucumber aside really streamlined the process and didn't leave me eternally feeling like I was losing focus on the functionality I wanted due to the the myriad tests that were demanded by all the tools I was using.

Madison Ruby was a fantastic source of information on how to write better ruby code and rails applications.  In particular, [Jeff Casimir's](http://twitter.com/#!/j3) talk on using view-models and decorators as opposed to helpers in rails was food for thought.  Halfway through implementation on this site I switched over to using [draper](https://github.com/jcasimir/draper) for my presentation logic and I __love it__.

Another tool I wanted to test out for this project was [git-flow](https://github.com/nvie/gitflow).  At [Scott Chacon's](http://scottchacon.com/) eye-opening Git workshop I increased my git knowledge 10 times, easily.  Going through the underlying structures of git taught me a lot about what's really going on during commits, push, and branching, and I wanted to try to formalize the release process of this project a bit more using git-flow's branching model.  The result: I loved the structure of the branches, even if this was a one-man project, and will definitely use it on more projects in the future.

Finally, I've been more and more intrigued by [Markdown](http://daringfireball.net/projects/markdown/syntax) since Github and many other sites support it these days.  Instead of pouring time into creating a WYSIWYG editor or trying to drop one in, I instead grabbed the [Redcarpet](https://github.com/blog/832-rolling-out-the-redcarpet) gem and simply use markdown to create the posts and project descriptions.  I also use [albino](https://github.com/github/albino) and [pygments](http://pygments.org/) for code highlighting.

## Methodology & Features ##

The problem I had to overcome this time (and every time I jumpstart a project) is that too often I get bogged down on tangents.  I'll get fixated on one unimportant feature or line of code, I'll get frustrated that I can't get it the way I want, and eventually I won't even want to open the text editor due to feelings of dread.  This time I was determined to mercilessly cut features until I had something simple I could get coded and deployed, and then iterate from there.  A few times I almost got bogged down, but forced myself to leave features on the table for future releases so that I could get my high priority functionality into production.

I settled on __four__ main features for the first release:

* __Responsive design__ so that the site would look great on regular monitors, phones, and tablets.
* __A micro-resume__ [viewable here](http://www.whazzing.com/me) that I could link to from my new business cards and elsewhere
* __Taggable blog posts__ without comments (for now)
* __A project guide__ so that I can link to all my projects in one place, with links to docs and code where appropriate

My major feature list for the next release includes:

* __Comments__ on blog posts (spam-protected and everything!)
* __RSS feeds__ of blog posts
* __Release tracking__ of projects
* __Rating and ranking__ of blog posts

I'll also, of course, be correcting anything that shakes out while using it.

## What's Next ##

I'm looking forward to extending this project, while using it as a platform to document the other things I'm working on.  I would really like to use [redis](http://redis.io/) for some analytics/tracking ideas I have, and I'm interested in how to tie in [akismet](http://akismet.com/) spam-blocking for comments.  

The images along the left side don't scan well on mobile devices, so I'm going to look into replacing them in the next release as well.  I like the structure of the site, but the specific details could be tightened a little.

I'd like to get a styleguide up and running so that my CSS can be standardized a little.  I tried as much as possible to be OO in my CSS but failed in a few places that I'd like to clean up.

## Thanks ##

Many, many thanks to [Joe Nelson](http://begriffsschrift.com/), from whom I shamelessly stole the idea for the micro-resume.  Thanks so much for the idea- it jump-started what became the new site, blog, etc.

Also, thanks to [Kevin Burke](http://kev.inburke.com/). I read his blog post on [designing your personal site](http://kev.inburke.com/kevin/site-redesign/) and was inspired to go beyond the usual "reverse chronological order of posts" home page.  Instead I tried to create a homepage that advertises all the things I'm working on, blog posts being only one of the pieces.

Thanks to [Dale Sande](http://www.dalesande.com/), who helped me step up my CSS game with his excellent [Axle framework](http://axle.dalesande.com) and thoughts on OOCSS (object-oriented CSS).  If you ever get a chance to attend his OOCSS talk definitely do it; as a primarily backend dude I found it immensely helpful for structuring my CSS and semantic markup.
