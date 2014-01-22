---
title: Somebody needs to update this thing
author: jared
layout: post
permalink: /general/somebody-needs-to-update-this-thing
categories:
  - General
---
Well after many weeks of just being lazy&#8230;.I&#8217;ve decided to try to work through my issues and post.

Last night I was up late migrating my web server to a new machine. Saying good-bye to the 233GHz machine and moving up to a 450GHz box. Blazing speeds! The main reason for the move was memory. The old server had around 192Mb of memory and even with making it bare bones (tuning,etc) it still was a bit sluggish with the newest Gallery2 software.

The plan was to move up to about 750MB on the new box, but the memory, while &#8220;recognized&#8221; by the BIOS, would cause a kernel panic when I tried to install SUSE Linux 10.0. Hmmmm. I didn&#8217;t have much time to investigate&#8230;or desire. So I tried some other memory I had on hand and so now we are blazing along with 384MB. Which has made a world of difference.

Moving the data was fairly easy too. I exported and re-imported the database. The websites themselves were on a separate harddrive/partition. So I was able to just move the harddrive from server a to server b. A quick couple of clicks (can you say click when you are talking about ncurses?) in YaST and it was ready. (I know that I could have simply update the fstab file&#8230;.but YaST is there for a reason right!?) A couple of quick modifications to the Apache2 conf files. Copy over the vhosts files from the old server. Done.

A quick nessus scan to make sure that I hadn&#8217;t forgotten any of my other hardening steps. Voila!

Today I made a couple of tweaks to the running services, etc. just to remove a couple of unneeded things to free up some more memory.

I the next couple of days I want tune Apache some more. I may just have to try out a couple of new things on the server now.