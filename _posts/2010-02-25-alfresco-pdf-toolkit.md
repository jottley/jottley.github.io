---
title: Alfresco PDF Toolkit
author: jared
layout: post
permalink: /alfresco/alfresco-pdf-toolkit
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - AMP
  - PDF Toolkit
---
A few weeks ago I made my first release of what I am calling the <a href="http://code.google.com/p/alfresco-pdf-toolkit/" target="_blank">Alfresco PDF Toolkit</a> on <a href="http://code.google.com" target="_blank">Google Code</a>.

Alfresco PDF Toolkit, or as I originally named it, <a href="http://svn.ottleys.net/public/alfresco/pdf-extension/" target="_blank">pdf-extension</a>, has been around for a while.  It was originally hosted on [my SVN server][1] and in then in <a href="http://svn.ottleys.net/alfresco" target="_blank">my Alfresco SVN Repo</a>.  It was developed as a one-off side project at the request of an early Alfresco customer.  The code has been sitting around with very few updates since that time.  An occasional rebuild to make sure it would work with a new release of Alfresco but that was it. (I think it was original built for 2.1 or 2.2)

**Why the update?** Focused attention.  It is a way to get my hands dirty and adding some new features that have been rattling around in my head for a while now.

**What can it do? **This initial release has three functions:

*   Split PDF &#8212; This option allows you to split a PDF every specified number of pages
*   Split at Page &#8212; This option allows you to split a PDF at a specified page
*   Merge PDF &#8212; This options allows you to append a PDF to another PDF

All three options generate new PDFs leaving the originals untouched.

**Where can I find these actions? **The actions are available in the Run Action Wizard.  They use the same names as above.

**What does the future hold?** The current list of enhancements is as follows (bot not necessarily in this order):

*   PDF Watermarks / Rubber stamping
*   Digital Signatures
*   TIFF to PDF transformation
*   Extract Pages
*   Delete Pages
*   Transform to PDF/A

**Can I help?** Yes!  I&#8217;m more than happy to accept code contributions or add features the list of enhancements.  And of course, merit can make you a contributor.

**Where can I find the code or the AMP again? **In the Google Code Project: [Alfresco PDF Toolkit][2]

***Note: ****This is NOT an officially supported Alfresco Project. *

 [1]: http://svn.ottleys.net/public
 [2]: http://code.google.com/p/alfresco-pdf-toolkit/