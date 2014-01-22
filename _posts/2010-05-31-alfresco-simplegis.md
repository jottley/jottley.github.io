---
title: 'Alfresco: SimpleGIS'
author: jared
layout: post
permalink: /alfresco/alfresco-simplegis
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - GIS
  - Google Maps
---
***Update:** Share in Alfresco 4.0 now supports google maps integration out of the box.*

When I worked Pre-Sales here at [Alfresco][1] I had, on occasion, the chance to talk about integrating Alfresco with a [GIS (G][2][eographic Information System)][2]. I don&#8217;t have a lot of experience with these types of systems (actually none) but I always believed that integration was possible. (It falls into one of those categories where we believe that it is possible, but we have never actually seen it done.) Or even that a simple type of integration could be done with [Google Maps][3] to provide basic information.

I started playing with the Google Maps APIs almost 6 months ago and was able to throw together a rough POC, but felt that there was still more that I could do.  I&#8217;ve gotten closer to what I envisioned and a have put together a starting example, but there is still a lot of work to do, including working around issues from the underlying APIs. (ie one of my examples will not work in some versions of Firefox or IE&#8230;I&#8217;m working on these, but believe that the answer may actually be in the code of either the browser or the Maps API.)

You should note that my example is based off of US addresses.  If you are outside of the US you may need to customize these further to meet your needs, but the underlying code should work fine.

**Content Map**

My first attempt, used an aspect to contain the address. The fields I looked at are: address, city, state and zip code. Any of these, or any combination of these, can be passed to Google to get the coordinates. In this release, I&#8217;ve stuck with this design with a little modification.  The map that is rendered for the document (as a presentation template in the document details page) uses a static google map.  This is quick and easy, it is basically an image tag that has several parameter passed as part of the src.  This approach works find for our simple content item map, but has some limitations, for example you can&#8217;t design it provide more in-depth information like the little pop up information windows.

In both of these example maps (the document based map and the space map that we will discuss next) it requires some additional changes to the core Alfresco App.  I needed to do two things: make a call within my web script to an external resource, in this case Google Maps, to take an address turn it into map coordinates.  And have the presentation template call the web script.  With a little help from Yong Qu, I was able to address both of these.  For the first issue you can enable the &#8216;remote&#8217; object in javaScript.  This is enabled by default for Alfresco Share, but not in Alfresco Explorer.  Yong has an easy to follow, [example on his blog][4], on how to do this.

To use the remote object then, we make a simple set of calls like below:

<pre style="padding-left: 30px">uri = "http://maps.google.com/maps/geo?q="+encodeURI(fullAddress)+"&key="+key+"&sensor=false&output=json"

var connector = remote.connect("http");
var result = connector.call(uri);

var obj = eval("("+result+")");

coordinates = obj.Placemark[0].Point.coordinates[1]+","+obj.Placemark[0].Point.coordinates[0];</pre>

You should also note that the json that is returned to our call is returned as a string.  We will need to turn it into an javascript object to work with it by using the javaScript eval function.  Once we have done this, we can working on the json as a standard object.

This object, remote, however us not enabled for use in presentation templates. So again, borrowing from Yong, I used YUI (which we ship and use in Alfresco) to allow me to make the call out to get the map, via web script.

Our YUI code, again is pretty straight forward&#8230;make the call to the web script, set the returned code to be the html of a specified div tag:

<pre style="padding-left: 30px">&lt;script src="/alfresco/yui/yahoo.js" type="text/javascript"&gt;&lt;/script&gt;
&lt;script src="/alfresco/yui/event.js" type="text/javascript"&gt;&lt;/script&gt;
&lt;script src="/alfresco/yui/connection.js" type="text/javascript"&gt;&lt;/script&gt;

&lt;script type="text/javascript"&gt;

 var handleSuccess = function(o){
    var div = document.getElementById('mapping');
    div.innerHTML = o.responseText; }

var handleFailure = function(o){
    var div = document.getElementById('mapping');
    div.innerHTML = "Map not available"; }

var callback = {
 	  success: handleSuccess,
 	  failure: handleFailure,
 	  argument: {} }

var request = YAHOO.util.Connect.asyncRequest('GET', '/alfresco/wcservice/map/workspace/SpacesStore/${document.id}', callback);
&lt;/script&gt;</pre>

We want to incude the js references as the beginning. This enables us to make successful calls to this script without the wrapper of our presentation template.

Next, notice the url that we are calling, 1/ the call is relative to the application we are running. 2/We are using wcservice. This make it so that we are not prompted to login each time the web script is called.

<p style="text-align: center">
  <img class="size-full wp-image-339 aligncenter" src="http://jared.ottleys.net/files/2010/06/Screen-shot-2010-05-31-at-11.07.21-PM.png" alt="Screen shot 2010-05-31 at 11.07.21 PM" width="659" height="266" />
</p>

As I mentioned above, this is the easer part especially since the hard parts were already thought out by Yong&#8230;.

**Space Map**

For the second map, I initially wanted it to be a presentation template, with an amalgamation of spaces & content, on a single map.

The static maps API is a not a fit here.  I wanted to have a map that allowed me not to just specify the location for each child in a space, but also display a limited set of specific information about that piece of content. ** Note:** This does mean that you may need to develop a specific &#8220;folder&#8221; based taxonomy to better organize content by location.

As I said above, my original attempts here were to try and do everything in a presentation template.  It would make multiple calls through YUI to get coordinates for the children of the space.  But YUI here becomes the bottleneck, especially since, I could not find a way to pass information out of the YUI Connection Managers callbacks.  This greatly limits the design&#8230;and ends up nesting calls to a depth of *n* as we loop through all of the children in a space.

One way to overcome this would be to include in the model/aspect the coordinates. This would reduce the YUI calls to zero.  One problem with this approach would be the changes to address need to result in updates to those coordinates.  This can be achieved through a behavior, watching the onUpdateProperties method for changes on those nodes. So for this first attempt, I&#8217;ve decided to skip this.  I&#8217;ll revisit this in an upcoming revision to the code.

I started my new web script by moving the logic that I worked on from the presentation template to this web script.  This was pretty straight forward.  We reused the remote object we enabled from before to pass our location (address string) back to Google.  I ignore 99% of what is returned in the json because what we are really after is the coordinates that we can pass to the Google Maps javaScript API.  I choose the javaScript API this time, because it allows me to create infoWindows where I can present the user with some basic metadata about the content along with links to either download or browse to the content.  I&#8217;m not much a UI designer so my poor attempts at layout are sure to make some squirm.

One of the things that was useful in this approach was to pass javaScript Associative Arrays to Freemarker.  This allowed me to specify the properties that I wanted/needed directly from witha larger sequence of objects without needing use multiple loops.

This was pretty straight forward:  I created an array in my controller.  At each element in the array, I created a new Array where I stored my values referenced by a key.  I could then in the FTL, start my list, and then reference each child by key name instead of looping again.

<pre>var big = new Array();

//some code that ends of looping over something
//take our properties/metadata stick them in an array

var _child = new Array();

_child["nodeID"] = child.id;
_child["location"] = _location;
_child["latitude"] = _latlong["latitude"];
_child["longitude"] = _latlong["longitude"];

.push(_child);
//end loop

model.big = big</pre>

Now in the ftl:

<pre>&lt;#list big as b&gt;

var _latlng${child_index} = new google.maps.LatLng(${child["latitude"]},${child["longitude"]});

&lt;/#list&gt;</pre>

When we query for the specific node we want to work with where the value we want to pass to the query is in the Associative Array, we need to first clean the variable up, then it will work without throwing an error:

<pre>&lt;#assign ref = child["nodeID"]&gt;
&lt;#list companyhome.childrenByLuceneSearch["ID:workspace\\:\\/\\/SpacesStore\\/${ref}"] as _child&gt;</pre>

<p style="text-align: center">
  <img class="size-full wp-image-340 aligncenter" src="http://jared.ottleys.net/files/2010/06/Screen-shot-2010-06-01-at-1.26.00-AM.png" alt="Screen shot 2010-06-01 at 1.26.00 AM" width="750" height="255" />
</p>

In my design I planned to reuse to the same pattern of using YUI within the presentation template to make the call to the web script, however when attempting to write the generated javaScript to the div tag where the map was to be displayed, I was reminded that the javaScript was never executed. It was treated as a string.  It is possible to get around this by using the javaScript eval function to execute the script, but this means turning the content retuned into a big string with lots of escapes&#8230;This would kill one of my design choices of making it possible to make direct calls to the web script and have it return a map that could be embedded in different things or be used stand alone. It could also be a big performance hit depending on how often the looip was made. Now, I could go back and write into the content branches to turn areas of the web script ftl on and off as a big sting, but this would over complicate the ftl.  This lead to the simplest fallback: have the presentation template reference the web script as part of an iFrame.  This worked nicely, but it helped to highlight what I thought was an issue with using the eval function:  there is a bug in the newest release of the maps API:  it was also happening in the iFrame on IE and Firefox.  For the moment I&#8217;ll let the script stand, it does work in Safari and Chrome, but it leaves out the major browsers.

<p style="text-align: center">
  <p style="text-align: center">
    <img class="size-full wp-image-341 aligncenter" src="http://jared.ottleys.net/files/2010/06/Screen-shot-2010-06-01-at-1.26.52-AM.png" alt="Screen shot 2010-06-01 at 1.26.52 AM" width="772" height="283" />
  </p>
  
  <p>
    <strong>How to use</strong>
  </p>
  
  <p>
    We are still a bit early for heavy use of our plug.  We need to work out a few caching issues, the display issues are IE and Firefox, and modification of core Alfresco configs.
  </p>
  
  <p>
    For this initial release I won&#8217;t provide an amp.  I&#8217;m want people to know that they will need to dig into this to learn it and extended it.  I&#8217;d also like to transfer this to be used in Share.
  </p>
  
  <p>
    <a href="http://code.google.com/p/alfresco-simplegis/source/checkout">The source code</a> is available at the <a href="http://code.google.com/p/alfresco-simplegis/">Google Code Project &#8212; alfresco-simplegis</a>.
  </p>
  
  <p>
    If you want to call the web scripts directly:
  </p>
  
  <p>
    For the document map use: /map/{store_type}/{store_id}/{id}
  </p>
  
  <p>
    For the Space map use: /map/coordinates/{store_type}/{store_id}/{id}
  </p>
  
  <p>
    Both of these will require that the space/content have the mappable aspect applied along with at least the zipcode applied.
  </p>
  
  <p>
    Spaces can control the default zoom level for their maps.  By default that value is 8.  You should adjust it based on your need.
  </p>
  
  <p>
    Document maps are static: there is no zoom or pan.  Space maps support both zoom and pan.
  </p>
  
  <p>
    <strong>What next?</strong>
  </p>
  
  <p>
    This project is still early in its development.  If you have ideas on how to overcome the Firefox or IE issues, your help would be appreciated.  I&#8217;m also going to be tackling the &#8220;caching&#8221; issue shortly.  This seems to me to be a useful feature.  Along with a good understanding of how you can script behaviors for properties when they are updated.
  </p>

 [1]: http://www.alfresco.com
 [2]: http://en.wikipedia.org/wiki/GIS
 [3]: http://code.google.com/apis/maps/
 [4]: http://drquyong.com/myblog/?p=49