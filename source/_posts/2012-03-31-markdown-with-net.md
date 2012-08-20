---
layout: post
title: "Markdown with .NET"
date: 2012-03-31 13:05
comments: true
published: false
categories: markdown, c#,
---
[Intuit's](http://www.intuit.com) developer culture has a loosey-goosey nature when it comes to documentation.  At least in the
[QuickBooks](http://quickbooks.intuit.com) code base there has traditionally been no proscribed way to document functionality outside
of code comments; teams and individuals could choose their documentation methods, which would often vary on a per-project basis.

My goal for the past week has been to __provide a better documentation solution in the QuickBooks codebase__, and it's loosely based
on the Github model.

### QuickBooks Documentation

It all started when I wanted to investigate the original design behind the payroll subscription management API, and I couldn't find
the documentation I knew had to exist somewhere.  I asked around found that there was a Word document somewhere with the original
API design, but it had been uploaded to a (homegrown) document storage solution that was no longer maintained and in fact had
been decommissioned.  I'd been thinking for awhile that we needed a better, more durable and accessible form of component/API
documentation in QuickBooks code but this pushed me over the edge.

I really, really like the idea of README-driven development and the [Github](http://github.com) model of putting the README front-and-center when you look at the source repository.  It gives you an up-front sense of what is going on with the code in question, and the [Markdown](http://daringfireball.net/projects/markdown/) format has really grown on me in the last year.  With these things in mind I dove into creating a better way to write documentation.

### The Constraints

In order to make a utility usable by every QuickBooks developer I had to stay away from scripting languages.  The environment is Windows (already a tough environment for scripting development) and every developer has an opinion on using this-versus-that language.  Perl is available but locked at version 5.008 with no option to upgrade; I note this because the original Markdown implementation explicitly states:

> Markdown requires Perl 5.6.0 or later. Welcome to the 21st Century.

There is a collection of batch scripts, Perl scripts, and utilities that are included with every developer environment of QuickBooks

### The Solution

I created a markdown-to-html executable to include in the developer tools collection that is automatically included in every build/dev environment.  I briefly thought about C or C++, going as far as downloading the source for [Discount](http://www.pell.portland.or.us/~orc/Code/discount/) after reading great things about it.  My problem with Discount (and it was my problem, not the code's) was that I was on a short time-frame and there were no clear instructions for compiling on Windows with the Visual Studio compiler.  The Discount web site mentions that

> It may build on Windows with mingw, but I’m not sure about that.

I wasn't terribly in the mood to deal with converting makefiles that _maybe_ work on <code>mingw</code> to work with <code>cl</code>, so I set this aside to consider other options.

After a little more googling I came across [MarkdownSharp](http://code.google.com/p/markdownsharp/), a C# library for working with Markdown conversion.  This interested me a little more: if the library was good enough I could write a command-line wrapper around it and still allow for whatever extra features I wanted to include.  I got the source and started hacking around with it.

### The Result: md2htmlc.exe

I settled on a minimum feature set to get things up and running as quickly as possible:

* Take an input Markdown file and turn it into an HTML file (same base file name, change extension from .md to .html)
* Take a directory of .md files and output all of them to HTML in a specified output directory
* Take a template HTML file (written with [Mustache](http://mustache.github.com/)) which the transformed content would be inserted into

It took two days to get it to a point where I felt comfortable spitting out a release build to include in the developer tools.  The template functionality came a little later, but it was incredibly easy thanks to the [Nustache](https://github.com/jdiamond/Nustache) library.  I also spun up some makefile targets in the QuickBooks build system to be able to generate a directory of HTML guides from a directory of Markdown source files.

My first two guides were a guide to writing guides (meta!) and a guide on how to generate both Guide and API documentation using the new make targets.

### The Future

I have two more items that I'd like to include in the utility:

1. Support for [YAML Front Matter](https://github.com/mojombo/jekyll/wiki/YAML-Front-Matter) in the guides. It would allow us to put some very interesting metadata in the guide for later usage (see #2 below).
2. The ability to optionally generate an index HTML page if you do a batch transform on a directory of guide source files.  If we had metadata through the above YAML Front Matter support we could use it to include things like a brief description or tags on the index page.

It's sometimes frustrating to work in development environments with limited support for your favorite scripting-slash-get-shit-done-quick language, but you can still be productive and move forward by hunting down the best available libraries and tools for your particular situation.  It doesn't hurt to be fairly agnostic towards languages and acknowledge the constraints you're working with.

I'm greatly looking forward to continuing to
