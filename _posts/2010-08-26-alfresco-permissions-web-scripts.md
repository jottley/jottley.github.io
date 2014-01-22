---
title: 'Alfresco: Permissions Web Scripts'
author: jared
layout: post
permalink: /alfresco/alfresco-permissions-web-scripts
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - AMP
  - web script
---
A couple of months back I was asked to write a couple of web scripts to help one of our customers to be able to check and modify permissions for content/spaces in the Alfresco repository.  I&#8217;ve finally had the chance to spend sometime testing and now writing about them.

The core of the web scripts was quick to write.  The fun (more time consuming) part was working with exception handling in javascipt.  I know tons of fun right!  There are few different ways to use exception handling based on which version of Alfresco you are using.  The customer is on Enterprise 3.1 and I wanted to make sure that the web scripts also worked on the more current releases of Alfresco as well.  A [change (re: addition)][1] was made in Enterprise 3.2.1 and Community 3.3 to help simplify exception handeling.  I&#8217;ll talk about exception handling and these differences in a follow up post.  For now let&#8217;s talk about these new web scripts.

## **permissions GET**

The first web script returns all of the permissions for a specified node.

The URL used is `/alfresco/service/permissions/{store_type}/{store_id}/{id}`

Where

<p style="padding-left: 30px">
  <code>store_type</code>: The type of store you want to query, ex: <code>workspace</code>
</p>

<p style="padding-left: 30px">
  <code>store_id</code>: The ID of the store you want to query, ex: <code>SpacesStore</code>
</p>

<p style="padding-left: 30px">
  <code>id</code>: The UUID of the node, ex: <code>aed218e8-df44-4865-84cd-0105252f4993</code>
</p>

The above values are joined together to form the nodeRef.

If the node is not found a 404 error will be returned, any missing URI parameters will result in a 400 error and if you don&#8217;t have permission to view the node you will get a 401 error.

The web script will return a JSON object that looks like the following:

<pre class="brush: jscript; title: ; notranslate" title="">{ "permissions": [
"ALLOWED;user1;Coordinator",
"ALLOWED;user2;Coordinator"
] ,
"inherit": false }</pre>

The return object lists the permissions in a triplet for that node. The permissions triplet follow this format:

<p style="padding-left: 30px">
  <code>[ALLOWED|DENIED];[USERNAME|GROUPNAME];PERMISSION</code>
</p>

It also returns a boolean value indicating if some permissions are inherited from the parent node.

The above example shows two permissions are assigned to the node: the Coordinator permission is given to user1 and user2 on this node. Permissions are not inherited from the parent node.

## **permissions POST**

This web script enables you to modify the permissions for a given node

It is called through the same URL as the above web script but as a POST instead of a get: `/alfresco/service/permissions/{store_type}/{store_id}/{id}`

Again, if the node is not found a 404 error will be returned, any missing URI parameters will result in a 400 error and if you don&#8217;t have permission to modify the node, you will get a 401 error.

You must also pass a JSON object containing the permissions that are being changed, deleted or added.

<pre class="brush: jscript; title: ; notranslate" title="">{ permissions: [
"REMOVE;user3;All",
"REMOVE;user2;All",
"ADD;user4;Coordinator",
"ADD;GROUP_usergroup1;Consumer"
] ,
"inherit": false }
</pre>

The above example uses the following triplet to define a permission

<p style="padding-left: 30px">
  <code>[ADD|REMOVE];[USERNAME|GROUPNAME];PERMISSION</code>
</p>

Where the values are defined as:

<p style="padding-left: 30px">
  <code>ADD | REMOVE</code>: Do you want to add or remove the permission for this user/group? Any other value passed will result in a 400 error.
</p>

<p style="padding-left: 30px">
  <code>USERNAME | GROUPNAME</code>: The user or group you want the permission to be added or removed for. Group names must be prefixed by GROUP_. Unknown users or groups will result in a 400 error.
</p>

<p style="padding-left: 30px">
  <code>PERMISSION</code>: The supported permissions options are defined in<br /> <code>org.alfresco.service.cmr.security.PermissionService</code> or through custom extension to the permission model. Unknown permissions will result in a 400 error.
</p>

The object can also contain an optional inherit permission value to specifying if the permissions for this node should be inherited from the parent node. Without the inherit option, the current value for the node is maintained. Inherited permissions can not be removed from a node.

The return format is the same as the return format of the permissions GET web script above.

This web script is also transactional: any errors will result in the node being returned to the state before the call was made. (The exception handling in the controller was added for these conditions.)

These scripts can be installed as an AMP.  The [code][2] and [AMPs][3] are hosted in the [alfresco-permissions-webscripts][4] project on Google Code.  The code is available for either [pre-3.2.1 (starting with 3.1)][5] or [3.2.1 to 3.3.1][6].  These are all Enterprise releases numbers.  The web scripts have been tested against these releases.  There may not be any need to modify the web scripts for Community releases (except for the min and max version numbers in the module.properties file).  Pre community 3.3 should use the pre 3.2.1 release.  No community releases have been tested. (If you try these on a community releases, please comment either here or in the Google Code Project.)

In a follow up post, I&#8217;ll cover exception handling with JavaScript.

 [1]: https://issues.alfresco.com/jira/browse/ALF-1958
 [2]: http://code.google.com/p/alfresco-permissions-webscripts/source/checkout
 [3]: http://code.google.com/p/alfresco-permissions-webscripts/downloads/list
 [4]: http://code.google.com/p/alfresco-permissions-webscripts/
 [5]: http://code.google.com/p/alfresco-permissions-webscripts/downloads/detail?name=alfresco-permissions-webscripts-pre.3.2.1.amp
 [6]: http://code.google.com/p/alfresco-permissions-webscripts/downloads/detail?name=alfresco-permissions-webscripts.amp