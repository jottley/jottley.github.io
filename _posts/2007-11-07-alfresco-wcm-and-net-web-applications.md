---
title: Alfresco WCM and .NET Web Applications
author: jared
layout: post
permalink: /alfresco/alfresco-wcm-and-net-web-applications
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - .Net
  - Alfresco
---
There are a few basic ways of using <a href="http://www.alfresco.com" target="_blank">Alfresco</a> in a <a href="http://en.wikipedia.org/wiki/.NET_Framework" target="_blank">.NET</a> web application; the <a href="http://wiki.alfresco.com/wiki/Alfresco_Content_Management_Web_Services" target="_blank">web services APIs</a>, <a href="http://wiki.alfresco.com/wiki/Web_Scripts" target="_blank">web scripts</a> (<a href="http://en.wikipedia.org/wiki/REST" target="_blank">REST</a>-API) and .NET code calls retrieving content directly from the filesystem.

Your content authors in the Authoring instance of Alfresco, inside your firewall, would create content.  This content would be deployed to a runtime instance of Alfresco outside of your firewall and/or to your IIS servers.  (Publishing to your IIS servers would be deploying static content.)

When creating content under <a href="http://wiki.alfresco.com/wiki/Web_Content_Management" target="_blank">Alfresco WCM</a>, <a href="http://wiki.alfresco.com/wiki/Forms_Developer_Guide" target="_blank">forms</a> are used to collect/enter the content.  The content is defined in a [XML Scheme Definition][1] (XSD).  This XSD defines the form  (content model) that the content author uses to enter content.  The XSD is used to create the XML file that holds the content that you are creating.  Applying templates against this XML file you can then create <a href="http://wiki.alfresco.com/wiki/Forms_Developer_Guide#Rendering_Engines" target="_blank">renditions of the content</a>.  These templates can be the HTML <a href="http://en.wikipedia.org/wiki/FreeMarker" target="_blank">Freemarker</a> templates, <a href="http://en.wikipedia.org/wiki/XSLT" target="_blank">XSLT</a> or <a href="http://en.wikipedia.org/wiki/XSL-FO" target="_blank">XSL-FO</a>.  The renditions can be full pages, micro-pages, code snippets or render the content in another format, like creating a PDF.

Your .NET web application could retrieve/ use content in a couple of ways:

0/ The Alfresco runtime could be queried via the SOAP APIs. The results displayed in your web application  
1/ URL calls to web scripts, which would <a href="http://wiki.alfresco.com/wiki/Web_Scripts_Framework#HTTP_Response_Formats" target="_blank">return</a> XML (that could be formatted as RSS or ATOM), JSON or rendered HTML (that could be a widget, or html with reference to your wire framework) that could be included directly in the page, or wrapped in an IFRAME.  
2/ Dynamic calls to static content on an IIS server.  (Content can be deployed to a specific location, and your .NET code could display that content in some fashion. [Reading files by naming convention, filetype, etc.])

The same type of implementation can be use with Java/J2EE, PHP, Ruby on Rails, CF, etc.

 [1]: http://en.wikipedia.org/wiki/XSD