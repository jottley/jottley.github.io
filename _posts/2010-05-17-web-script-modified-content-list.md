---
title: 'Web Script: Modified Content List'
author: jared
layout: post
permalink: /alfresco/web-script-modified-content-list
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - web script
---
For a recent customer engagement I was asked to write a web script which they&#8217;ve agreed to let me share.

The script was written to give them:

*   A list of content that had been modified from a point in time until &#8220;now&#8221; (The time of execution).
*   Specify a specific space or recurse the child spaces for the named space.
*   return a specified XML structure

In preparing for this post I&#8217;ve added some modifications/fixes:

*   I&#8217;ve added two new return formats: JSON and HTML
*   fixed an issue that allows it to run on the 3.2.x release of Alfresco.

Here are some insights to a few areas of the script:

**Lucene Date Queries**

There are two specific things to remember about Date queries:

<p style="padding-left: 30px">
  <strong>1 &#8211; Date Format.</strong> Date queries with Lucene are  looking for the date in an I<a href="http://en.wikipedia.org/wiki/ISO_8601">SO8601</a> format using <a href="http://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations">combined date and time </a>in <a href="http://en.wikipedia.org/wiki/UTC">UTC</a>.  (Even if you are looking just for the date it will complain if you don&#8217;t pass the time ).
</p>

<p style="padding-left: 30px">
  To do this I had originally used  the JavaScript Date prototype from <a href="http://delete.me.uk/2005/03/iso8601.html">http://delete.me.uk/2005/03/iso8601.html</a>. This works fine in pre 3.2 releases.  But due to a <a href="https://issues.alfresco.com/jira/browse/ETHREEOH-2760">change in the 3.2 code line</a> top level objects are now sealed which means you can&#8217;t make these type of prototype change to Root level objects in the JavaScript libraries. So I&#8217;ve made simple modification to Date prototype code allowing me to pass a Date Object into the code which then does the coversion and returns a properly formated ISO8601 string.
</p>

<pre style="padding-left: 30px">function toISO8601String(date) { // based on http://delete.me.uk/2005/03/iso8601.html
	/*
	 * accepted values for the format [1-6]: 1 Year: YYYY (eg 1997) 2 Year and
	 * month: YYYY-MM (eg 1997-07) 3 Complete date: YYYY-MM-DD (eg 1997-07-16) 4
	 * Complete date plus hours and minutes: YYYY-MM-DDThh:mmTZD (eg
	 * 1997-07-16T19:20+01:00) 5 Complete date plus hours, minutes and seconds:
	 * YYYY-MM-DDThh:mm:ssTZD (eg 1997-07-16T19:20:30+01:00) 6 Complete date
	 * plus hours, minutes, seconds and a decimal fraction of a second
	 * YYYY-MM-DDThh:mm:ss.sTZD (eg 1997-07-16T19:20:30.45+01:00)
	 */
	if (!format) {
		var format = 6;
	}
	if (!offset) {
		var offset = 'Z';
		var date = date;
	} else {
		var d = offset.match(/([-+])([0-9]{2}):([0-9]{2})/);
		var offsetnum = (Number(d[2]) * 60) + Number(d[3]);
		offsetnum *= ((d[1] == '-') ? -1 : 1);
		var date = new Date(Number(Number(date) + (offsetnum * 60000)));
	}

	var zeropad = function(num) {
		return ((num &lt; 10) ? '0' : '') + num; 	}

 	var str = "";
 	str += date.getUTCFullYear(); 	if (format &gt; 1) {
		str += "-" + zeropad(date.getUTCMonth() + 1);
	}
	if (format &gt; 2) {
		str += "-" + zeropad(date.getUTCDate());
	}
	if (format &gt; 3) {
		str += "T" + zeropad(date.getUTCHours()) + ":"
				+ zeropad(date.getUTCMinutes());
	}
	if (format &gt; 5) {
		var secs = Number(date.getUTCSeconds() + "."
				+ ((date.getUTCMilliseconds() &lt; 100) ? '0' : '')</pre>

<pre style="padding-left: 30px">+ zeropad(date.getUTCMilliseconds()));</pre>

<pre style="padding-left: 30px">str += ":" + zeropad(secs);</pre>

<pre style="padding-left: 30px">} else if (format &gt; 4) {
		str += ":" + zeropad(date.getUTCSeconds());
	}

	if (format &gt; 3) {
		str += offset;
	}
	return str;
}</pre>

<p style="padding-left: 30px">
  I&#8217;ve turned the original prototyping into a function, where I pass a Date object, with the date I want to work with, into this function. I also replaced any reference to &#8216;this&#8217; with that data parameter.  I will admit there is some excess code here that may never be called. But this was the quickest change to accomplish what I needed.
</p>

<p style="padding-left: 30px">
  <strong>2 &#8211; Only dates are indexed.</strong> By default, only the date portion of the property is indexed.  In general, most use cases only require the date .  But in this case, the customer was interested in what may have changed over the period of an hour.
</p>

<p style="padding-left: 30px">
  There are two options for this: Code around or modify how Alfresco indexes these DateTime properties. In this case I choose to code around it.  This means that I am not modifying default behavior in Alfresco&#8230;always a plus.
</p>

<p style="padding-left: 60px">
  <strong>Code Around.</strong> We have all the pieces we need to do this: We know the datetime we use for the start and end our range query.  We also have access to the full datetime property (even if it wasn&#8217;t indexed).
</p>

<p style="padding-left: 60px">
  I chose to pull this into a function that takes the collection (array) of documents found in our range query and then test to see if the datetime property of the node (in this case the modified datetime property) is between our the beginning and end of our range, if so then add it to our new filtered results array.
</p>

<pre style="padding-left: 60px">function filterResults(unfilteredResults) {
	var now = new Date();
	var filteredResults = new Array();

	for each (node in unfilteredResults) {

		var testDate = new Date(node.properties.modified);

		//if the nodes modified date is between the passed date/time and now add
		//to filteredResult Array
		if ((filterDate &lt;= testDate) && (testDate &lt;= now)){
			filteredResults.push(node);
		}
	}

	return filteredResults;
}</pre>

<p style="padding-left: 60px">
  This is clean and simple.
</p>

<p style="padding-left: 60px">
  <strong>Modify Alfresco.</strong> While this sounds heavy, it actually isn&#8217;t.  Our support team pointed me at <a href="http://forums.alfresco.com/en/viewtopic.php?f=4&t=9122">this forum post</a> in which <a href="http://twitter.com/andy_hind/">Andy Hind</a> from our engineering team shows us how to change the default behaviour:
</p>

<p style="padding-left: 60px">
  In dataTypeAnalyzers.properties (located inside the war file in WEB-INF/classes/alfresco/model) change:
</p>

<p style="padding-left: 60px">
  d_dictionary.datatype.d_datetime.analyzer=org.alfresco.repo.search.impl.lucene.analysis.DateAnalyser
</p>

<p style="padding-left: 60px">
  to
</p>

<p style="padding-left: 60px">
  d_dictionary.datatype.d_datetime.analyzer=org.alfresco.repo.search.impl.lucene.analysis.DateTimeAnalyser
</p>

<p style="padding-left: 60px">
  Now rebuild your indexes.  Depending on the amount of content in your repository it this may take some time.  It may also not be possible to make this immediate because the  SLA you have with your customers won&#8217;t permit it until you have scheduled maintenance downtime. (HA Clustering is helpful in these situations.)
</p>

**System Folders**

One thing that may show up in these queries are system folders.  These folders contain any of the rules that may be associated with a space.  In most queries these folders and their content has no value, so we want to exclude them.

This string can be appended to your query string to exclude them:

<pre style="padding-left: 30px">var systemFolder = "-TYPE:\"cm:systemfolder\" " +
		"-TYPE:\"{http://www.alfresco.org/model/rule/1.0}rule\" " +
		"-TYPE:\"{http://www.alfresco.org/model/action/1.0}compositeaction\" " +
		"-TYPE:\"{http://www.alfresco.org/model/action/1.0}actioncondition\" " +
		"-TYPE:\"{http://www.alfresco.org/model/action/1.0}action\" " +
		"-TYPE:\"{http://www.alfresco.org/model/action/1.0}actionparameter\"";</pre>

**PARENT vs PATH**

When we are performing queries for content within a space we have a couple of options.  Two of the most common are PARENT and PATH.

PARENT queries work directly on a space. No recursion. It will return all of the nodes (spaces and content) in that space.  In other words, PARENT queries work directly on a space without recursion. It will return all of the nodes (spaces and content) in that space alone, but will not return any nodes (spaces and content) that are in sub-spaces of that space.

PATH queries are a subset of XPATH.  They are eagerly evaluated.  Thus they can be memory (Caching) and CPU intensive. They are useful if you want more than what is just in the space you are working, may not know the location of the space, or if a space of that name may exist in multiple locations.

Be smart in your choice.  Use PARENT as often as possible.

*Note For Java extensions:<span style="font-style: normal"> u</span><span style="font-style: normal">nless the other clauses in your query are complex, it&#8217;s likely more efficient to enumerate (list / ls / dir) a space using the FileFolderService rather than a Lucene query. </span>*ie. a simple query of the form &#8220;PARENT:[noderef]&#8221; would be better implemented using FileFolderService.list()

<span style="font-style: normal">OK, enough about some of our design decisions let&#8217;s talk about how to use the web script.</span>

<span style="font-style: normal"><strong>Install</strong> </span>

<span style="font-style: normal">The web script is packaged in an amp file.  It will work on Alfresco 3.1.1 and newer versions. (I tested up to Alfresco 3.2r) The <a href="http://code.google.com/p/alfresco-modified-content-list/downloads/detail?name=alfresco-updatedcontent-webscript.amp">alfresco-modified-content-list amp</a> and <a href="http://code.google.com/p/alfresco-modified-content-list/source/checkout">the source</a> can be found on the <a href="http://code.google.com/p/alfresco-modified-content-list/">google code project site</a>. Use the apply_amps script appropriate for your OS.</span>

**How to use it**

The web script can be called using:

<p style="padding-left: 30px">
  http://localhost:8080/alfresco/service/updated/in/{path}/since?date=01/01/2010 13:10:10
</p>

<p style="padding-left: 30px">
  http://localhost:8080/alfresco/service/updated/{path}/since?date=01/01/2010 13:10:10
</p>

There is a slight difference in the two URLs: &#8216;in&#8217;

The &#8216;in&#8217; allows you to tell the web script to just list the content in the space passed in the path parameter. Without the &#8216;in&#8217; the web script will recurse the space structure from the passed path parameter down.

Next, the DateTime structure is not fixed:  Possible (tested) formats are

<p style="padding-left: 30px">
  mm/dd/yyyy
</p>

<p style="padding-left: 30px">
  yyyy/mm/dd
</p>

<p style="padding-left: 30px">
  mm/dd/yyyy hh:mm:ss
</p>

<p style="padding-left: 30px">
  yyyy/mm/dd hh:mm:ss
</p>

The hh is expecting a 24 hour clock format.  And millisecond and timezone tests are not supported

The query tests against the modified datetime property of the content.  (Fairly easy to change it and test against the created property or any other custom metadata DataTime property.)

There are three return formats: XML (the default), JSON and HTML.

<p style="padding-left: 30px">
  <strong>XML</strong> The XML format is a simple structure that returns back the name of the file, the path to the file and the modification date. It is also the default format returned.
</p>

<pre style="padding-left: 30px">&lt;contentItems&gt;
	&lt;contentItem name='simple.doc' path='/Company Home/test/Portal' modDate='2010-04-26T16:21:20.612-06:00'/&gt;
	&lt;contentItem name='simpledraft.docx' path='/Company Home/test/Portal' modDate='2010-04-26T16:28:44.569-06:00'/&gt;
&lt;/contentItems&gt;</pre>

<p style="padding-left: 30px">
  If there is no modified content a simple <span style="font-family: Consolas, Monaco, 'Courier New', Courier, monospace;line-height: 18px;font-size: 12px"><contentItems/> </span>is returned.
</p>

<p style="padding-left: 30px">
  <strong>JSON </strong>The JSON object is somewhat similar, a modified object containing a collection of node objects is returned
</p>

<pre style="padding-left: 30px">{ "modified": [
              { "node": {
                         "name": "test3.txt", "path": "some/path","modified": "2010-03-17T00:05:43.311-06:00"
                        }
              },
              { "node": {
                        "name": "test2.txt", "path": "some/path","modified": "2010-03-17T15:07:30.301-06:00"
                        }
              }
              ]
    }</pre>

<p style="padding-left: 30px">
  <strong>HTML</strong> This final format is more for testing than actual production use.  It returns a simple html page with a list of modified items. It displays a simple unordered list, where each item is a csv list with key and values seperated by colons.
</p>

In all of these return formats, the freemaker template processes the returned scriptNodes for a simple collection.  This makes it easy to extend the return with the returned nodes properties to include things like nodeRef, creation date, node icons, download url, etc.

If you have suggestions for improvements or questions please feel free to comment here or on the google project.  As always I&#8217;m willing to open access to the project to those will to help improve the code.