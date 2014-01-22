---
title: Lessons learned in Appliance building, Ideas for the Future
author: jared
layout: post
permalink: /alfresco/lessons-learned-in-appliance-building-ideas-for-the-future
aktt_tweeted:
  - 1
categories:
  - Alfresco
  - General
tags:
  - Alfresco
  - appliance
---
Errands run, kids in bed, and <a href="http://www.imdb.com/title/tt0085811/" target="_blank">Krull</a> <a href="http://en.wikipedia.org/wiki/Video_on_demand" target="_blank">On Demand</a>. Finally, sometime to put some thoughts down about building the <a href="http://jared.ottleys.net/alfresco/alfresco-appliance-212" target="_blank">Alfresco Appliance</a>.

Some of what I write here <a href="http://jared.ottleys.net/alfresco/evaluating-with-a-virtual-appliance" target="_blank">refers back</a> to when I original put together, the first version of the appliance. Some is from revisiting it for <a href="http://jared.ottleys.net/alfresco/alfresco-appliance-212" target="_blank">this release</a>.

**Choosing a distribution**

I started playing with building an appliance right about the time that Ubuntu announced their <a href="http://www.ubuntu.com/products/whatisubuntu/serveredition/jeos" target="_blank">Jeos distribution</a>. I have a strong preference for <a href="http://www.opensuse.org/" target="_blank">openSUSE</a> and would have liked to have built the appliance on it. (<a href="http://en.opensuse.org/LimeJeos" target="_blank">LimeJEOS</a> answers my concerns, and I am excited to give it a spin. Just a few more things to move off my plate before I can devote time to it.) I tried several different distributions: <a href="http://fedoraproject.org/" target="_blank">Fedora</a>, <a href="http://www.opensuse.org/" target="_blank">openSUSE</a>, <a href="http://www.ubuntu.com" target="_blank">Ubuntu</a>, <a href="http://www.puppylinux.org/" target="_blank">Puppy Linux</a>, <a href="http://www.zenwalk.org/" target="_blank">Zen Walk</a>. I liked some, was optimistic about others, and disappointed and frustrated with a couple.

What I wanted was a distribution that provided me a small lightweight base, and gives me the ability to add just the packages I needed to run my application. Most of these came in close to what I wanted, but due to dependencies, grew to be much larger than I wanted. Some didn&#8217;t even provide me with a nice clean way to get the applications I was dependent on. Those I dropped immediately. Those with decent package mangers got pluses. I wanted to be to keep base packages up to date.

This brings us to Ubuntu Jeos (<a href="http://en.wikipedia.org/wiki/Jeos" target="_blank">Just Enough OS</a>). It gave me the size, the packages, and the manager that met my base criteria for a distribution.

**Getting Started**

While the base install is already fairly small, there still are many packages that I was going to need and not need. This takes some time, but go through the installed packages and identify what you aren&#8217;t and are going to need. Make a list and develop a script to remove the ones you don&#8217;t need. Another to add the ones you need. You may like to have some of the packages around while developing your appliance, but when you go to deliver it they may add more weight than what you would like, or give tools that just aren&#8217;t need by your appliance, or your end users. Keep them installed until you are sure they aren&#8217;t needed any more. My list includes: man pages, wireless-tools, sound packages, laptop packages, different editors, hardware utilities, etc. Your list will vary. Sometimes you may want to force remove dependent packages if you know their functionality is not needed. Don&#8217;t be afraid to take them out.

I know my list is not perfect, but over time it will improve. Some of this will come with trial and error. Don&#8217;t expect to get it right the first time.

Once I narrowed the package list, I started to add my dependent packages. Some of these could require you to re-add removed packages. That is why I suggest leaving packages installed until you have all your dependencies installed.

One of the cornerstones of an appliance is zero configuration. You want people to be able to just deploy the appliance, start it and use the appliance. In the case of <a href="http://www.alfresco.com" target="_blank">Alfresco</a>, it needs <a href="http://tomcat.apache.org/" target="_blank">Tomcat</a> to be started at boot time. For this I needed an <a href="http://en.wikipedia.org/wiki/Linux_startup_process#Init_process_.28SysV_init_style_only.29" target="_blank">init script</a>. (Mine can be found <a href="http://svn.ottleys.net/public/alfresco/init/ubuntu/alfresco" target="_blank">here</a>.) I use the init script to start Tomcat, start needed services, and perform runtime configurations.

Alfresco is a web-based application, so I need a way to provide the URL needed by users to access the application. For an appliance this can be changed from install to install. There are several ways to handle this, eventual, you will want to have a static address or use Dynamic DNS to update IP to name mappings. My appliances is primarily for evaluation. It is not really ideal to play with DNS or static IPs. I have kept the implementation simple, I&#8217;ve created a script that updates the /etc/issue file with runtime URL. (The script I use is <a href="http://svn.ottleys.net/public/alfresco/appliance/update-issue" target="_blank">here</a>.) The major issue I faced was when the script was to be run. I placed the script in /etc/network/if-up.d. I fully expected it to be run when the interface came up. The script did run when I manually ran ifup. But it didn&#8217;t run in every case, especially the most import, at boot time. I tried several different things to work through this. I added a reference to the script into /etc/network/interfaces using the post-up directive. (in openSUSE, you would add a POST\_UP\_SCRIPT=&#8221;<name of your script>&#8221; to the interface file in /etc/sysconfig/network and put the script in /etc/sysconfig/network/scripts). This added a bit more power, but did quite fix my problem. I finally narrowed it down to <a href="http://en.wikipedia.org/wiki/Udev" target="_blank">udev</a>. The interface was being brought up by udev, but when /etc/init.d/networking is run, it runs ifup with a -a option (all interfaces marked auto). The problem with this is if the interface is already up, the associated scripts won&#8217;t be run. I haven&#8217;t found an appropriate fix for this yet. (Any ideas?) So I just disabled the udev rule for the network interfaces . <a href="http://www.rpath.com" target="_blank">rPath</a> has a good solution for this, they change /etc/issue file in an init script. I like this idea, but didn&#8217;t want to write another init script or pollute my existing alfresco init script with this.

**Plans for the Future**

One of the things that I want to do is provide some Alfresco branding to the appliance. I also want to create some more production ready appliances that use MySQL and point at an external virtual drive to store content and the indexes.

**What is Missing**

One of the big things that is missing in a management interface for the appliance. rPath provides a very nice extensible <a href="http://wiki.rpath.com/wiki/rPath_Appliance_Platform_Agent" target="_blank">web-based administrative interface</a>. It is extensible both for <a href="http://wiki.rpath.com/wiki/rPath_Appliance_Platform_Agent:Plugins" target="_blank">function</a> and <a href="http://wiki.rpath.com/wiki/rPath_Appliance_Platform_Agent:Branding" target="_blank">branding</a>. But it is rPath specific. I&#8217;d like to see an open source cross distro solution. One that was not only extensible but adds <a href="http://en.wikipedia.org/wiki/Common_Information_Model_%28computing%29" target="_blank">CIM based management</a> features. This would make it easy to administer the CIM instrumented applications on the appliance, but also could allow it to be managed externally, but CIM enabled management tools. This would allow the appliance to play nicely in the data center. This is <a href="http://wiki.xensource.com/xenwiki/XenCim" target="_blank">something</a> that Xen is working toward.