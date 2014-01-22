---
title: Open Carpet Redux
author: jared
layout: post
permalink: /general/open-carpet-redux
categories:
  - General
---
A while back, I [posted][1] about Open Carpet and OES. I thought I&#8217;d add some more about OES Linux.

On OES Linux I have already mentioned the novell-release file. There is also an SuSE-release file. In your distributions.xml file you would list it as:

<pre>&lt;distro&gt;
&lt;name&gt;SUSE LINUX Enterprise Server&lt;/name&gt;
&lt;version&gt;9&lt;/version&gt;
&lt;arch&gt;i586&lt;/arch&gt;
&lt;type&gt;rpm&lt;/type&gt;
&lt;target&gt;sles-9-i586&lt;/target&gt;
&lt;status&gt;supported&lt;/status&gt;
&lt;detect&gt;
&lt;file source="/etc/SuSE-release"
substring="SUSE LINUX Enterprise Server" /&gt;
&lt;/detect&gt;
&lt;/distro&gt;</pre>

Why are there two release files? Simple. Let&#8217;s say you are developing an application that you only want, or only will support on OES, but not on SLES 9 (the base OS behind OES). Voila. There is your control&#8230;if you are delivering the application through Open Carpet.

 [1]: http://jared.ottleys.net/archives/2005/08/open_carpet.html