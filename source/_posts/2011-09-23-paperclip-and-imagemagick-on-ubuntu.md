---
layout: post
title: Paperclip and ImageMagick on Ubuntu
---
In getting the [paperclip gem](https://github.com/thoughtbot/paperclip) working on my server for some screenshots I ran into a frustrating problem.  I kept getting an error during model saving: __"/tmp/1234bduct-123.png is not recognized by the 'identify' command"__.  Eventually I solved the problem, so here is my solution, hoping that it helps someone else out there.

After some googling and reading [Stack Overflow](http://www.stackoverflow.com) I quickly came across the solution that Paperclips :command_path option wasn't being set correctly.  The problem, however, was that it looked like I was correctly setting the ImageMagick path:

<pre lang="bash">
$ which identify
/usr/local/bin/identify
</pre>

I kept double- and triple-checking that the option was set correctly in my config/environments/staging.rb file:

<pre lang="ruby">
WhazzingCom::Application.configure do
  ....
  Paperclip.options[:command_path] = "/usr/local/bin"
end
</pre>

That seemed to be correct, but I still kept getting the same error.  Eventually I came across an [excellent suggestion](http://blog.e-thang.net/2009/12/30/paperclipimagemagickrmagick-error-is-not-recognized-by-the-identify-command/comment-page-1/#comment-1067) that perhaps my installed version of ImageMagick was incomplete, corrupt, or missing libs or dependencies.  First I reran the ImageMagick apt-get install...

<pre lang="bash">
$ sudo apt-get install imagemagick --fix-missing
</pre>

...which installed identify to __/usr/bin__ rather than the existing version in __/usr/local/bin__.  I then updated my environment file to the following...

<pre lang="ruby">
WhazzingCom::Application.configure do
  ....
  Paperclip.options[:command_path] = "/usr/bin"
end
</pre>

...and refreshed the page and everthing worked as expected!

