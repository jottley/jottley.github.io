---
title: Hula and Jabber
author: jared
layout: post
permalink: /general/hula-and-jabber
categories:
  - General
---
I did a lot of thinking last night about a recent post by [Dave Camp][1] on the possibility of [snorp][2] working on some patches to backend [Jabber][3] to [Hula][4]. Unfortunately, it doesn&#8217;t look like that is going to happen&#8230;.yet.

So, just my high level overview, not knowing a lot of the internals of either, I&#8217;m thinking this shouldn&#8217;t be to hard. You are probably just pointing jabber at the ldap server of hula..right? And then work out pulling or adding to the contact list into the users address book.

Then let&#8217;s move it a step further and look at adding to this a client. Now, I know there are lots of IM clients out there, multi-protocol and such, but lets think of this in terms of collaboration via Hula. So we develop a mono based client, with a plug-in architecture. We need to take advantage of the xml routing as Miguel de Icaza pointed [out][5]. So we build plug-ins that support pulling calendar information from Hula (personnel and shared), a plug-in that supports [iFolder3][6] (more integration with the identity store and user address book of Hula), and a plugin that supports [Beagle][7] searches from the client. (You would also want some standard plug-ins that support things like weather and such.) [Jawaad][8] pointed out to me that this may be doing a lot of the work the [Dashboard][9] is going to do. Maybe, but if designed right it could be made to be another backend that could be consumed by Dashboard. He also suggested maybe working out a new mono based version of [gaim][10] that would support all this&#8230;maybe.

So once that was all done, then work could be done on adding the idea of presence awareness to Hula&#8230;

It is mostly just pondering on an idea&#8230;.or the blabbering on lunatic.

I think mostly the later.

 [1]: http://campd.org/?p=16
 [2]: http://snorp.net/
 [3]: http://www.jabber.org/
 [4]: http://www.hula-project.com/
 [5]: http://tirania.org/blog/archive/2005/Aug-24.html
 [6]: http://www.ifolder.com/
 [7]: http://beaglewiki.org/
 [8]: http://blog.jawaad.net
 [9]: http://www.nat.org/dashboard/
 [10]: http://gaim.sourceforge.net