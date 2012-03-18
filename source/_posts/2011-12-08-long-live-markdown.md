---
layout: post
title: Long Live Markdown
---

I've been doing a lot of development around [markdown](http://daringfireball.net/projects/markdown/syntax) lately and I'm really starting to love it.  Specifically, [Github-flavored markdown](http://github.github.com/github-flavored-markdown/) has grown on me with its ability to do easy fenced code blocks and syntax highlighting.


	```ruby
	comments = Comment.all
	```

becomes

<pre lang="ruby">
comments = Comment.all
</pre>

When I rewrote the code for this web site awhile back I decided to support markdown, but didn't really think I'd use it much; maybe to do code blocks but not much else.  Now I find myself writing markdown for all content, and only translating it through various interpreters when I need the HTML.  In retrospect I'm extremely glad I went through the effort to add markdown support for the site-- I use it all the time now.

There's an __incredible__ app for writing markdown on the fly: [Mou](http://mouapp.com/) is a Mac application that lets you input markdown on the left pane and the HTML is dynamically updated in the right pane.  I'm literally using it right now to write this post, just to get a preview of what it will look like when I drop it into the &lt;textarea&gt; on the web site.  

I basically use it for all markdown content authoring, and then I copy/paste the result into whatever application/content field I will use it in.  It allows me to do edit/revision cycles on content as well as seeing the application of styles.  Highest recommendation.

I also recently forked [git-wiki](https://github.com/sr/git-wiki) and started working on improving it for my own use.  It also uses Markdown to write the wiki entries, which gets a little weird if you've ever used [MediaWiki](http://www.mediawiki.org/wiki/MediaWiki) or [Wikipedia](http://www.wikipedia.com).  The default way to define [wikiwords](http://twiki.org/cgi-bin/view/TWiki/WikiWord) in git-wiki didn't really work for me so I updated it and checked everything back into [my own repo](https://github.com/whazzmaster/git-wiki).  I'll be updating more about my git-wiki fork once I get it refactored and a little more configurable.  

So there you go- markdown is awesome and you should write your content in it; if you've never worked with it then try it out on Github (just make a public repo and create a README.md in the index).
