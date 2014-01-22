---
title: 'Alfresco: Simple Workflow Web Scripts'
author: jared
layout: post
permalink: /alfresco/alfresco-simple-workflow-web-scripts
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - AlfrescoAlfrescoSimple WoSimple Workflow
  - Simple Workflow
---
Continuing on from my previous post on [Simple Workflows][1] let&#8217;s look at using the Simple Workflow model in a few web scripts.

The first web script will add a simple workflow to a node. The second will allow us to signal the transition of accept or reject on that node.

**Add a Workflow**

As discussed in the previous post, a simple workflow is made up of two paths: accept or reject with a copy or move to another folder. You are also able to set the name presented to the user for the accept or reject step. This is very useful as what we may be presenting to the user is to not accept or reject documents, but rather sending the document to two different group owned folders based on context/content of those documents. At least one step is required. Alfresco defaults &#8220;accept&#8221; for the required step and makes &#8220;reject&#8221; the optional step. These need to be passed to our web script. The payload is passed as json. A full example of this looks as follows:

<pre class="brush: plain; title: ; notranslate" title="">{
    "simpleworkflow": {
        "accept": {
            "name": "acceptstep",
            "move": true,
            "folder": "workspace://SpacesStore/ebd7ff99-b423-4e45-ad4f-71929ce4c089"
        },
        "reject": {
            "name": "rejectstep",
            "move": true,
            "folder": "workspace://SpacesStore/0239b2ed-ebfa-4b9d-a7fa-f0404448cffc"
        }
    }
}
</pre>

In the above example, the reject object is optional.

Both the accept and reject objects are made up of the three required values of the simple workflow aspect: name, move and folder.

The name field maps to either approveStep or rejectStep. It is the name presented in the Alfresco UIs. But it could also be used in a custom interface as they are returned as part of the list of properties for the node.

The move field maps to either approveMove or rejectMove. This boolean field indicates with a true value to move the node or, if false, to copy the node.

The final field, folder, maps to approveFolder or rejectFolder. This is the folder node where the document should be moved or copied to when the appropriate action is taken.

The web script can be call by passing the simpleworkflow json object to `simpleworkflow/add/{store_type}/{store_id}/{id}`. Upon completion a json object of `{ "success": true|false] }`. True upon successful completion, false if there was an error adding the simpleworkflow aspect to the node.

**Using a Simple Workflow from a Web Script**

This web script was originally developed for a customer to return XML, but I&#8217;ve now updated it to return json by default. The XML is maintained here for backwards compatibility.

When a simple workflow aspect is added to a node, it just adds the properties of what should occur when an accept or reject action is taken. The actual actions of what happens is built into UIs. So the web scripts need to do the same: perform the copy or move to the targeted folder. The call is pretty simple: `simpleworkflow/{accept|reject}/{store_type}/{store_id}/{id}`. The json or XML that is returned let us know of either the success or the failure of this action. Failures can provide some information on why the failure occurred. A success messages, provides more detail: what action occured &#8212; copy or move, which step was taken &#8212; returning the name as set in the approveStep or rejectStep properties, the folder that document was moved or copied to. The destination that is returned is the human readable path. (If there is any interest we can update this to return the nodeRef as well as the path.)

The json returned looks like:

<pre class="brush: plain; title: ; notranslate" title="">{
    "success": true,
    "action": "move",
    "step": "acceptStep",
    "destination": "/Company Home/Sites/workflow/docmentLibrary/workflow/accept"
}
</pre>

These web scripts can be [downloaded][2] from its Google Code Project.

 [1]: http://jared.ottleys.net/alfresco/alfresco-simple-workflow "Alfresco: Simple Workflow"
 [2]: http://code.google.com/p/alfresco-simple-workflow-webscripts/