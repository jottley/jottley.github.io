---
title: Alfresco Appliance 2.1.2
author: jared
layout: post
permalink: /alfresco/alfresco-appliance-212
aktt_tweeted:
  - 1
aktt_notify_twitter:
  - yes
categories:
  - Alfresco
tags:
  - Alfresco
  - appliance
  - virtualization
---
**UPDATE**:  I am trying to release a new instance of this sometime in August 09.  The current image is based on Ubuntu & I may build it with the new 3.2 community package.  But I have also been playing with SUSE Studio&#8230;So look for those (one or both) this month.

*Update: Those who downloaded the appliance, may have noticed a packaging problem (ie part of it was missing! :()  I have fixed that as well as done a little tunning and a few modifications.  I&#8217;ll post about those later.*

Available now! I&#8217;ve just finished the latest evaluation version of the VMware based <a href="http://ottleys.net/alfresco/appliances/vmware/alfresco-eval-2.1.2.tgz" target="_blank">Alfresco Appliance</a> built on the latest <a href="http://www.alfresco.com/products/" target="_blank">Alfresco Enterprise release 2.1.2</a>. This is a full 30-day trial release. It comes with the core Document Management Repository and Web Content Management pre-installed.

This is based on my <a href="http://jared.ottleys.net/alfresco/evaluating-with-a-virtual-appliance" target="_blank">previous work</a> using <a href="http://www.ubuntu.com/products/whatisubuntu/serveredition/jeos" target="_blank">Ubuntu Jeos</a>. (I am, however, keen to try out <a href="http://en.opensuse.org/LimeJeos" target="_blank">LimeJeos</a> based on <a href="http://www.opensuse.org/" target="_blank">openSUSE</a>.)

A couple notes:

*   When it first starts, Alfresco will take a few minutes to come up fully. It is creating the initial DB. Subsequent restarts are quicker.
*   This is an evaluation appliance. So once you start it, you have 30-days before the trial expires.
*   When the appliance first starts it will ask you if you copied or moved the appliance. The correct answer is moved it.

Look for other (community) appliances in the next month or so. I also have some RPMs for Alfresco Community in the works (using the <a href="https://build.opensuse.org/" target="_blank">openSUSE Build Serivce</a>). I&#8217;ll post more this evening about what I learned since the last time I built the appliance.