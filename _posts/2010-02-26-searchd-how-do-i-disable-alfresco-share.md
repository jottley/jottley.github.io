---
title: 'Search&#8217;d: How do I disable Alfresco Share'
author: jared
layout: post
permalink: /alfresco/searchd-how-do-i-disable-alfresco-share
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
  - "Search'd"
tags:
  - Alfresco
  - "Search'd"
---
I&#8217;ve always found it fascinating what people searched for to find my blog.  And more fascinating why my blog even came up in the results!  The idea occurred to me that there were actually some meaningful questions in those searches. Yes, it was like one of those Windows 7 epiphanies.  I&#8217;m going to start a new weekly (cross your fingers) series based on those search terms. New posts will come every Friday and will incorporate some facet of the search terms that brought people to this blog.  The post size will vary based on the question and the technical depth it may require.  Hopefully, this can prove to be valuable. I&#8217;ll also add a pitch here for [Alfresco Professional Services][1] &#8212; bread on the table, mouths to feed and all &#8212; we offer the in-depth knowledge you need to help with your [Alfresco Enterprise Subscription][2]. So without further ado&#8230;.I give you: Search&#8217;d

Today&#8217;s search terms came in as &#8220;alfresco start up disable share studio&#8221;.  For most Java Developers who have worked with Java application servers this will be pretty simple &#8212; you may even want to find something else to do now.  For those new to Java or not even in the market to learn, it will not be.  But the answer is pretty simple, but first some background&#8230;..

Java based applications can be packaged into several different formats. Typically you will find JARs and WARs.  In some circumstances they are packaged in other formats as well&#8230;RARs and EARs being the most common.  We at Alfresco like to use AMPs (which may or may not actually include Java code) to manager extensions to Alfresco.  These archives are zip files with specific file layouts. Go ahead, take a copy of the alfresco.war file and open it with a archive manager that understands zip.  It won&#8217;t hurt.

Now, if you downloaded Alfresco as one of the full blown installers or as a bundle with tomcat, go ahead and run the installer or unpackaged the archive.  Once that is done navigate down into the tomcat/webapps directory.  In there you will see a few directories and a couple of files with the extension .war (alfresco.war, share.war, mobile.war (Community only), studio.war (Community only)).  To disable any of these simply remove the war file or rename it by anding a new/additional extension.  I like to add the extension .off, which lets me clearly see that this part of Alfresco is &#8212; off.  Removing the war files also may seem a bit brute force and unless you are really desperate for the space it won&#8217;t hurt anything to leave them.  This is my preferred approach.  If you ever want to add that part of Alfresco back, just remove the extension and restart Alfresco.

If you have already started Alfresco you will see directories that match the name of the war file.  You will also need to remove those directories. (Don&#8217;t forget to restart Alfresco after removing those directories).

It&#8217;s as simple as that.

**Q:  Is there ever a case where I would disable  the Alfresco application? **Yes.  If you are using Alfresco Share, you can add a bit of scale to your deployment by having Share installed on another server.  It does require some additional configuration, but we won&#8217;t cover that today.

**Q: I just removed __\_\\_\_.war!  Or I just removed the \_\_\____ directory! How do I get it back? **For basic out of the box installs, stop Alfresco and simply grab the war file from the [Alfresco Community Page][3] or [Alfresco Network][4] (versions must match) and copy it back into the webapps directory.  If you have added extensions via AMPs to your war files, reapply the amps to the new war file. Restart Alfresco.  For the case where you deleted an expanded directory, with the war files in place, restart Alfresco and they will be deployed again.

 [1]: http://www.alfresco.com/services/consulting/
 [2]: http://www.alfresco.com/services/subscription/
 [3]: http://wiki.alfresco.com/wiki/Download_Community_Edition
 [4]: http://network.alfresco.com