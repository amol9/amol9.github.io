---
layout: post
title: GitHub / Jekyll Blog
tags: blog, github 
---

I am mirroring my blog at amol9.blog to amol9.github.io. The GitHub blog is based on Ruby/Jekyll. I searched a little, it does not seem  to have what I need. My posts count is now in two digits, and increasing, so, I was thinking about grouping the posts better, in the directories, one each for the year, month and the date. I could not find much information on the Internet, in the Jekyll docs. There is a page on collections there. More on it, in a bit.

So, I went ahead, and decided to go for the trial approach. I added the directories and the post file. I committed the changes and pushed the changes to GitHub. The blog did not show the new post, using the URL, in the ‘year/month/date/filename’ form did not help, after doing the necessary renames, of course.

Next, I looked at the Jekyll collections config option. But, quickly saw that, for my need, of arranging the posts in three levels of directories, the configuration required will be quite a lot. So, this approach will not work.

There was no other option, but, to go with the single directory approach. There may be many posts there, hundreds, or even thousands. This is the way to manage it.

I have pushed the post in the simple arrangement now, in the ‘_posts’ dir. Upon refreshing the blog page, it does not show. Something weird shows. A page with today’s date repeated three times, without a blog title and then the old posts are shown.
