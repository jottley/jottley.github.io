---
title: glTail.rb and vhosts
author: jared
layout: post
permalink: /general/gltailrb-and-vhosts
aktt_tweeted:
  - 1
categories:
  - General
---
Last week I tried out using <a href="http://fudgie.org/" target="_blank">glTail.rb</a> against my server and sites.  It worked fine against my default site (<a href="http://www.ottleys.net" target="_blank">www.ottleys.net</a>) However, against the <a href="http://jared.ottleys.net" target="_blank">bl</a><a href="http://adrienne.ottleys.net" target="_blank">ogs</a> and the <a href="http://gallery.ottleys.net" target="_blank">gallery</a>, I got back no info.  To fix the problem, I just modified the apache parser expression.

My vhosts  log entries start out with the DNS name of the site.  The parser expression is looking for the client IP address at the start of each entry.  By removing the &#8216;^&#8217; I was able to start pulling data.