---
title: Using Alfresco to manage blog content
author: jared
layout: post
permalink: /alfresco/using-alfresco-to-manage-blog-content
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - blogs
  - MS Office Integration
---
<a href="http://www.alfresco.com" target="_blank">Alfresco</a> is very much about collaboration, specifically collaboration around/with content.  I have written about <a href="http://jared.ottleys.net/alfresco/i-bit-the-bullet" target="_blank">Alfresco-Facebook Collaboration</a> but the first integration that I saw/used with Alfresco was its integration with blogs.  Alfresco delivers an <a href="http://wiki.alfresco.com/wiki/Blog_Publishing_User_Guide" target="_blank">add-on module</a> that allows you to integrate Alfresco with, by default, <a href="http://wordpress.org/" target="_blank">WordPress</a> or <a href="http://www.movabletype.org/" target="_blank">Movable Type</a>.   The interfaces that the add-on uses are open, and therefore could also be used to integrate with other blog software/platforms.

With the add-on installed, you can designate a folder that contains content (HTML or text files) that can be posted to a defined blog.  But that is not where the strength of the Alfresco integration comes.  The strength of using Alfresco to manage your content comes in the Alfresco feature set.

**All of the blog content can be versioned**.  During the creation phase/life of your blog articles, the content can be <a href="http://wiki.alfresco.com/wiki/Versioning" target="_blank">versioned</a>.  If an article is being worked on by multiple people in the system, you can keep track of who made changes and what those changes are.

**Content can be managed as part of a workflow**.   I have seen potential blog posts from management and developers being passed around for review in email.  There is no accountability in that.  There is no way to track where it went and what comment/modifications are being made along the way.  And it really would stink if important changes were lost in someones inbox.  Potential posts could be submitted as part of a <a href="http://wiki.alfresco.com/wiki/Workflow" target="_blank">workflow</a>, where the post is assigned to people for review.  As part of the process email notifications could be sent to each person during the process to let them know of assignment, completion of task and final posting of the content. (One of the cool features of Alfresco is that all content is URL referenceable, so an actual file never needs to be sent to anyone, just the URL.  And the URL always references the latest version of the content.)

**Content transformation.** Content could also start out in one format and be <a href="http://wiki.alfresco.com/wiki/Client_Manage_Content_Rules#Transform_and_copy_content_to_a_specific_space" target="_blank">transformed</a> to another by rule.   It is possible that the post could first start out as a Word document and then be introduced into the repository using the <a href="http://wiki.alfresco.com/wiki/Microsoft_Office" target="_blank">Alfresco Microsoft Office Integration</a>. Before content makes it to be published, the Word document could be transformed into a text file.

Alfresco makes it possible for you to create collaborative content, in a collaborative way.  With more businesses starting to use blogs to communicate with customers, being able to manage the content is becoming more important.