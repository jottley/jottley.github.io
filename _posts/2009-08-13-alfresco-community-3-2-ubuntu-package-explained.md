---
title: Alfresco Community 3.2 Ubuntu Package Explained
author: jared
layout: post
permalink: /alfresco/alfresco-community-3-2-ubuntu-package-explained
aktt_notify_twitter:
  - no
categories:
  - Alfresco
tags:
  - Alfresco
  - ubuntu
---
***NOTE: **This instructions apply to Jaunty Jackalope (9.04).  The package is currently being updated for Karmic Koala (9.10).*

***UPDATED:** Added link to CIFS/iptables configuration*

***UPDATED:*** *Added note about applying amp files.  
*

A couple weeks back we [released][1] the [Alfresco Community V3.2 ][2]for Ubuntu package. The package was developed in conjunction with [Canonical][3]. We&#8217;ve been working on this for a [while][4], but now have in place a general structure that will enable us to deliver updates to the package in a timely fashion.

**Which version of Ubuntu? What is installed?**

This initial release is packaged for Ubuntu 9.04 (Jaunty Jackalope).  There maybe some back ports coming for older Ubuntu releases but nothing definite at this time.  The idea is to make Alfresco Community easy to install, with most functionality available right out of the box.  We&#8217;ve made a concerted effort to use base packages from the Jaunty distro, only adding ours when needed. The major packages that are installed when Alfresco is installed are: Tomcat 6, Sun Java 6, Mysql 5.1, OpenOffice.org 3 (Headless), ImageMagick, pdf2swf (from swftools &#8212; this is our own packaged version of the devel branch.  It will be replaced with the next release of swftools when it is updated in the base of the distro).

**How Do I Install It?**

The alfresco-community package is located in the Ubuntu partner repository.  *These instructions assume that you are using the command line.* You can also use an X-Windows desktop to install alfresco.  If you do, some of the steps are a bit different, but the general outline can be grasped from these instructions. So let&#8217;s start&#8230;

You can add the partner repository by editing `/etc/apt/sources.list`. You need to uncomment the partner repo:

<span style="font-family: monospace, 'Times New Roman', 'Bitstream Charter', Times, fantasy">deb http://archive.canonical.com/ubuntu jaunty partner<br /> deb-src http://archive.canonical.com/ubuntu jaunty partner</span>

Next, you need to add the signed repository key:

<span style="font-family: monospace, 'Times New Roman', 'Bitstream Charter', Times, fantasy">sudo apt-key adv &#8211;keyserver keyserver.ubuntu.com &#8211;recv-keys E048451D9EE6D873</span>

Refresh the apt cache for the newly added repository:

<span style="font-family: monospace, 'Times New Roman', 'Bitstream Charter', Times, fantasy">sudo apt-get update</span>

You can now install Alfresco:

<span style="font-family: monospace, 'Times New Roman', 'Bitstream Charter', Times, fantasy">apt-get install alfresco-community</span>

You will be prompted to enter the mysql admin password, to accept the Sun Java license, configure the alfresco database username and password (the default is to choose alfresco/alfresco), allow for the automatic configuration of tomcat.

Once complete, [Alfresco Explorer][5] and [Alfresco Share][6] will be available by pointing your browser to `http://<IP Adresses or DNS Name>:8080/alfresco` or `http://<IP Adresses or DNS Name>:8080/share`

**Where does everything get put?**

Using one of the default linux installs from [www.alfresco.com][7], you will typically find alfresco installed under `/opt/alfresco`.  By using the default tomcat, shipped with ubuntu, we followed its directory/configuration structure.

*   **Alfresco indexes and contentstore:** The Alfresco indexes and contentstore can be found under `/var/lib/alfresco`
*   **Alfresco war files:** The alfresco and share war files can be found under `/var/lib/tomcat6/webapps`
*   **Alfresco extension directories:** The extension directory, where you should make configuration changes to alfresco and share, can be found under `/var/lib/tomcat6/shared`
*   **Alfresco log files:** The alfresco log files can be found under `/var/log/tomcat6`
Many of these directories are symbolic links to directories under `/usr/share/tomcat6`. However, the default tomcat configuration points to the `/var/lib/tomcat6` directory.

By using the default tomcat we gain the advantage of an out of the box init script to control start and stop of Alfresco.  You can now use `/etc/init.d/tomcat start|stop|restart|try-restart|force-reload|status` to control or check the status of  the tomcat server.

To change the startup parameters of tomcat or those of Alfresco, (ie. memory settings, configuring JMX, etc.) modify the JAVA_OPTS variable in `/etc/default/tomcat6`.

Tomcat is run using a non-root user.  This means that Alfresco cannot open the ports needed for CIFS, FTP, or NFS. Follow the [instructions to configure alfresco and iptables][8] to listen on non-standard ports and perform port forwarding. (Look at [David Baker&#8217;s blog][9] for [step by step instructions on how to configure Alfresco CIFs with port forwarding using iptables][10].) The same use of non standard ports and iptables configurations should be applied to inbound [SMTP][11] and [IMAP][12] configurations.

**Lagniappe**

Here are a few other important things to note

<span style="font-weight: normal"><strong>AMP Files</strong> &#8211; One noticeable difference is that there is no apply_amps script.  This is an oversight on my part and will be corrected in the next release.  Meantime, if you want to apply an <a href="http://wiki.alfresco.com/wiki/AMP_Files">AMP</a>, you can <a href="http://process.alfresco.com/ccdl/?file=release/community/build-2039/alfresco-mmt-3.2.jar">download the MMT tool</a> and follow the <a href="http://wiki.alfresco.com/wiki/Module_Management_Tool">instructions to install AMPs with MMT</a>. <strong>Note:</strong> After applying an amp you will need to stop the alfresco server, remove the existing alfresco directory (<code>rm -rf /var/lib/tomcat6/webapps/alfresco</code>) and start alfresco. The new war file which contains the amps will then be deploy</span>

**<span style="font-weight: normal"><strong>Web Studio & Mobile</strong> &#8211; <a href="http://wiki.alfresco.com/wiki/Web_Studio">Alfresco Web Studio</a> and  <a href="http://wiki.alfresco.com/wiki/Alfresco_Community_Edition_3.2#Mobile_client_for_iPhones">Alfresco Mobile</a> are not included in the current package.  They will be included in the next release.</span>**

**What does the future hold?**

The plan we are discussing would be to provide the core repository and Alfresco Explorer as the central package, with separate packages for share, mobile, web studio, WCM and the filesystem receiver.

We are also open for suggestions.  Let us know what you think.

 [1]: http://www.alfresco.com/media/releases/2009/07/ubuntu/
 [2]: http://wiki.alfresco.com/wiki/Alfresco_Community_Edition_3.2
 [3]: http://www.canonical.com
 [4]: http://www.ubuntu.com/news/alfresco-enterprise-content-management
 [5]: http://wiki.alfresco.com/wiki/Alfresco_Explorer
 [6]: http://wiki.alfresco.com/wiki/Alfresco_Share
 [7]: http://www.alfresco.com
 [8]: http://wiki.alfresco.com/wiki/File_Server_Configuration#Running_SMB.2FCIFS_from_a_normal_user_account
 [9]: http://www.davidbaker.cc/
 [10]: http://www.davidbaker.cc/?q=node/3
 [11]: http://wiki.alfresco.com/wiki/Inbound_Email_Server_Configuration
 [12]: http://wiki.alfresco.com/wiki/IMAP