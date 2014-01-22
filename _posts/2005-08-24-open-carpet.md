---
title: Open Carpet
author: jared
layout: post
permalink: /general/open-carpet
categories:
  - General
---
I have amassed a small collection of RPMs, from various sources, that I use and share with other team members. Instead of pointing them all to the different repositories and other sources to get them, we set up a shared directory through which people would just grab them and updated them as needed. Knowing that this was probably not the most efficient way to do this, and seeing how the person assigned to set up a [ZLM server][1] for the team to use failed at this miserably (he got a another machine for himself which was a plus for him I guess. But it has been almost a year and still no ZLM server!). To not step on any toes, and not needing all the functionality of a zlm server, I set up an [open carpet server][2].

As soon as I let people know it was up, they come asking to use it. [OES Netware][3] has the ability to use red-carpet (the rug command line tool) to update rpms. (Anybody out there building rpms for Netware?) So to get this working for OES Netware you need to add it as a distribution in the distribution.xml file. Quite easy to do, just add:

<pre>&lt;distro&gt;
	&lt;name&gt;OES&lt;/name&gt;
 	&lt;version&gt;NW65&lt;/version&gt;
 	&lt;arch&gt;i386&lt;/arch&gt;
 	&lt;type&gt;rpm&lt;/type&gt;
 	&lt;target&gt;oes-nw65-i386&lt;/target&gt;
 	&lt;status&gt;supported&lt;/status&gt;
 	&lt;detect&gt;
		&lt;file source="/etc/netware-release" substring="OES NW65" /&gt;
 	&lt;/detect&gt;
&lt;/distro&gt;
</pre>

This will get you up and running. While OES Linux is based on the the same code as SLES 9. You should probably add something like the following to use open-carpet with it:

<pre>&lt;distro&gt;
	&lt;name&gt;Novell Open Enterprise Server Linux&lt;/name&gt;
	&lt;version&gt;9&lt;/version&gt;
	&lt;arch&gt;i586&lt;/arch&gt;
	&lt;type&gt;rpm&lt;/type&gt;
	&lt;target&gt;oes-linux-i586&lt;/target&gt;
	&lt;status&gt;supported&lt;/status&gt;
	&lt;detect&gt;
		&lt;file source="/etc/novell-release" substring="Novell Open Enterprise Server Linux" /&gt;
	&lt;/detect&gt;
&lt;/distro&gt;
</pre>

I will probably end up setting up a ZLM server any way as I have not discovered how to use open carpet to deliver patches from a YOU (YaST Online Update) server yet. (Which is what we are spending a lot of time testing right now.)

 [1]: http://www.novell.com/products/zenworks/linuxmanagement/
 [2]: http://www.open-carpet.org/
 [3]: http://www.novell.com/products/openenterpriseserver/index.html?sourceidint=productsmenu_oes