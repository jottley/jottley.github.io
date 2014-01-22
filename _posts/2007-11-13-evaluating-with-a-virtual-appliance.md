---
title: Evaluating with a Virtual Appliance
author: jared
layout: post
permalink: /alfresco/evaluating-with-a-virtual-appliance
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - evaluation
  - jeos
  - Virtual Machine
---
I have been working on a side project of developing a Virtual Appliance for [Alfresco][1]. My first attempt is <a href="http://ottleys.net/alfresco/Alfresco.vm.tgz" target="_blank">here</a>. I have played with several distros (<a href="http://fedoraproject.org/" target="_blank">Fedora</a>, <a href="http://www.opensuse.org" target="_blank">openSUSE</a>, <a href="http://www.ubuntu.com/" target="_blank">Ubuntu</a>, <a href="http://www.puppylinux.org/" target="_blank">puppylinux</a>, <a href="http://zenserver.zenwalk.org/" target="_blank">zenserver</a>) trying to figure out the right combination. This first release is based on <a href="http://ftp.heanet.ie/disk1/ubuntu-cdimage/jeos/" target="_blank">Ubuntu</a> <a href="http://en.wikipedia.org/wiki/Jeos" target="_blank">JeOS</a> as a VMware VM. I need to dig into <a href="http://en.opensuse.org/KIWI" target="_blank">Kiwi</a> and see if I can build out the same thing with openSUSE. I think that Fedora has a utility to do something similar.

There are lots of things to think about when planning a virtual appliance: size, dependencies, easy of use, what you want end-users to be able to do with the appliance, etc. I have gone back and forth on trade-offs. Initially, I would like to see the appliance as a quick and easy way for someone to evaluate Alfresco. So I stuck with HSQL for the DB. I removed a nice chunk of packages that I would not expect anyone to be using during an evaluation of Alfresco. I think key is keeping them focused. If you are trying to do everything with Alfresco you are bound to get distracted. One of the things that you want with an evaluation is that it just works. While there is a log in prompt, I have the appliance report back, on boot, the most important information they need to try out Alfresco: The URL to access Alfresco. Just point your browser at the URL provided. Everything self-contained.

The VM has Alfresco DM and WCM installed. I haven&#8217;t loaded it up with all the cool add-ons, etc. While that would be fun, it takes space, and keeps from the focus: The basic/core functionality of the repository.

I still need to cut down on the size, learn how to better package VMs and make them work under multiple VM managers.

P.S. If you download this and you feel you really need the login credentials and can&#8217;t figure them out&#8230;.let me know I will be glad to send them to you.

 [1]: http://www.alfresco.com