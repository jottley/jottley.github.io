---
title: 'Alfresco: Simple File Diff'
author: jared
layout: post
permalink: /alfresco/alfresco-simple-file-diff
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
---
I&#8217;ve heard asked many times by customers and community members if there was a way to diff files in Alfresco and alas there isn&#8217;t an OTB way to do this.  A month ago the discussion came up again internally.  And I thought it might be fun to tackle this as side project just to see if/what was possible.  So I took an evening and hammered out a simple Java class that did a comparison between two text files.  Once I saw that I had at least the basics (annotate the differences between two files) and had gotten the question of basic possibility/difficulty out of the way I moved on to other projects.

Today almost the entire family is sick so I thought I&#8217;d pick up the project again, moving the Java class to a Java Backed web script.

The web script is a simple GET that takes the nodeRef of two files, or two versions of the same file and outputs a simple HTML page that highlights the differences between the two.  There are no complex algorithms that take into account shifts in blocks or identifies just the text in a line that has changed.  It is a simple line by line comparison of two pieces of content. It is not integrated in to Share or Explorer at this time.  I might take that as a separate sick day project (or accept any code contributions to add that).

I&#8217;ll admit right off that the code is ugly and repetitive.  But this is more of a Proof of Concept than a full production ready implementation (though it could definitely be used as such to provide a quick view of differences).

I&#8217;ve also probably bored you with the above so let&#8217;s just jump right in before I completely lose you&#8230; <img src="http://jared.ottleys.net/wp-includes/images/smilies/icon_wink.gif" alt=";-)" class="wp-smiley" /> 

## **Using The Web Script**

The web script is called by the following URL:

alfresco/service/file/{protocol\_1}/{identifier\_1}/{id\_1}/diff/{protocol\_2}/{identifier\_2}/{id\_2}

For real world examples we&#8217;ll first look at comparing two files

alfresco/service/file/workspace/SpacesStore/dd83c9f6-81b7-462b-8a1a-1e9a2af251dd/diff/workspace/SpacesStore/ca65d129-6c2c-4ba0-936d-d7626f94f23a

Second comparing two versions of the same file

alfresco/service/file/workspace/SpacesStore/dd83c9f6-81b7-462b-8a1a-1e9a2af251dd/diff/versionStore/version2Store/ca65d129-6c2c-4ba0-936d-d7626f94f23a

What is returned is, as stated above, a simple HTML highlighting the differences

<p style="text-align: center">
  <a href="http://jared.ottleys.net/files/2010/11/Screen-shot-2010-11-28-at-4.46.07-PM.png" rel="lightbox[416]"><img class="aligncenter size-full wp-image-417" src="http://jared.ottleys.net/files/2010/11/Screen-shot-2010-11-28-at-4.46.07-PM.png" alt="" width="582" height="197" /></a>
</p>

Each line that is different is highlighted in blue.  Simple and to the point.

## **The Code**

This is just a little Declarative Web Script that reads the content line by line and then compares the hash of each line to see differences.  When a difference is found it is wrapped in HTML to annotate the difference so that when displayed, CSS can take care of highlighting the differences.

A couple of things that I think are important to note:

*   **File length:** When comparing two files there is always the possibility that one is longer/shorter than the other.  To simplify the comparison, I just append lines with a single space to the shorter file, simplifying any computational work needed for the comparison caused by the difference in length. 
    *   I mentioned above that the appended line contains a single space.  This is done so the that the line appears in the output.  <div> tags with no content can be ignored by some browsers. The annotation/presentation uses a combination of <div> and <pre> tags.  The space is maintained in a <pre> tag forces the div element to be visible.
*   **Special Characters: **Because the output for the comparison is targeted for HTML, it is important to escape all characters/strings that could be interpreted by the browser as presentation elements.  Apache Commons (included with Alfresco) has classes to help do this.
*   **Gotcha!:** When I was initially testing the code, the file content kept appending files to the previous request.  So remember when defining a Collection as a class scoped variable to call clear() on the List to make sure it is empty before it gets reused.

This extension is available as an [AMP][1]. The source is available in the [Google Code Project][2].

 [1]: http://alfresco-simple-file-diff.googlecode.com/files/alfresco-filediff-webscript-0.1.amp
 [2]: http://code.google.com/p/alfresco-simple-file-diff/