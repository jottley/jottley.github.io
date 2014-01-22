---
title: 'Alfresco: Alfresco PDF Tool Kit &#8211; Insert PDF Action'
author: jared
layout: post
permalink: /alfresco/alfresco-alfresco-pdf-tool-kit-insert-pdf-action
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - AMP
  - Extension
  - PDF Toolkit
---
I&#8217;ve taken a bit of Holiday time to update the Alfresco PDF Toolkit.  [Nate][1] has been doing an outstanding job adding [Watermarking][2], [Digital Signatures][3], [Encryption][4] and cleaning up my messy code.  But it was time to add a little bit myself.

So I took sometime this evening to add in one of my planned actions: Insert PDF.  This action allows you to insert a PDF into another PDF at a specific page.

This is a pretty straight forward action to test:  From the Document Details page of the PDF you want to insert content into, select Run Action (This action can also be run through the rules engine or scripted).

From the list of Actions you&#8217;ll want to select Insert PDF.

<a href="http://jared.ottleys.net/files/2010/11/Screen-shot-2010-11-27-at-7.52.02-PM.png" rel="lightbox[406]"><img class="aligncenter size-full wp-image-410" src="http://jared.ottleys.net/files/2010/11/Screen-shot-2010-11-27-at-7.52.02-PM.png" alt="" width="389" height="104" /></a>

Now you will want to set the parameters for the insert.  There are 4 parameters for an insert:

1.  **Name:** This is the name of the new file that will be generated.  No extension is needed, it will automatically be added.
2.  **Destination:** Space where the new file will be stored.
3.  **Insert at page:** Where page 1 of the inserted PDF will start in the targeted PDF.
4.  **Insert:** The PDF to insert into the targeted PDF

<a href="http://jared.ottleys.net/files/2010/11/Screen-shot-2010-11-27-at-7.52.24-PM.png" rel="lightbox[406]"><img class="aligncenter size-full wp-image-411" src="http://jared.ottleys.net/files/2010/11/Screen-shot-2010-11-27-at-7.52.24-PM.png" alt="" width="621" height="319" /></a>

Finally, you&#8217;ll get a message summarizing the action you are taking

<a href="http://jared.ottleys.net/files/2010/11/Screen-shot-2010-11-27-at-7.52.58-PM.png" rel="lightbox[406]"><img class="aligncenter size-full wp-image-412" src="http://jared.ottleys.net/files/2010/11/Screen-shot-2010-11-27-at-7.52.58-PM.png" alt="" width="488" height="62" /></a>

See pretty simple!

The latest [amp][5] and [source][6] can be found at the [Alfresco PDF Toolkit][7] project site.

 [1]: http://www.unorganizedmachines.com/
 [2]: http://www.unorganizedmachines.com/site/software-and-technology/34-software-development/99-watermarking-pdf-documents-with-alfresco
 [3]: http://www.unorganizedmachines.com/site/software-and-technology/34-software-development/106-alfresco-pdf-toolkit-digital-signatures
 [4]: http://www.unorganizedmachines.com/site/software-and-technology/34-software-development/101-encrypting-pdf-documents-with-alfresco
 [5]: http://alfresco-pdf-toolkit.googlecode.com/files/alfresco-pdf-toolkit-0.97.amp
 [6]: http://code.google.com/p/alfresco-pdf-toolkit/source/checkout
 [7]: http://code.google.com/p/alfresco-pdf-toolkit