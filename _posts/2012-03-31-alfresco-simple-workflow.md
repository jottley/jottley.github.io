---
title: 'Alfresco: Simple Workflow'
author: jared
layout: post
permalink: /alfresco/alfresco-simple-workflow
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - Simple Workflow
---
When I worked on the sales side at Alfresco, one of the easiest things to demo was a simple workflow. It was usually something that enabled the prospect to tick off a major requirement on their long list of things that the solution they were looking for needed. Alfresco has two kinds of workflow: simple and for lack of a better word advanced. Advanced workflows usually require some level of business logic, multiple steps, several actors, etc. Advanced workflows are implemented using [Activiti][1] (Alfresco 4.0 forward) or JBPM (pre Alfresco 4.0 and maintained for legacy in Alfresco 4.0) Simple workflows provide an accept, a reject, and then the content is either copied or moved to another folder.

What I really like about simple workflows is how they are implemented and how you can model your own customizations or applications using this pattern. As a follow up to this post I&#8217;m going to share a web script to add a simple workflow to a node and then another to progress the workflow by accepting or rejecting it.

**Let&#8217;s dig in!**

Simple workflows present, as stated above, the ability to accept and reject the document and then copy or move that document to another space. The key to a simple workflow is the model.

<pre class="brush: xml; title: ; notranslate" title="">&lt;aspect name="app:simpleworkflow"&gt;
         &lt;parent&gt;app:workflow&lt;/parent&gt;
         &lt;properties&gt;
            &lt;property name="app:approveStep"&gt;
               &lt;type&gt;d:text&lt;/type&gt;
               &lt;protected&gt;true&lt;/protected&gt;
            &lt;/property&gt;
            &lt;property name="app:approveFolder"&gt;
               &lt;type&gt;d:noderef&lt;/type&gt;
               &lt;protected&gt;true&lt;/protected&gt;
            &lt;/property&gt;
            &lt;property name="app:approveMove"&gt;
               &lt;type&gt;d:boolean&lt;/type&gt;
            &lt;/property&gt;
            &lt;property name="app:rejectStep"&gt;
               &lt;type&gt;d:text&lt;/type&gt;
               &lt;protected&gt;true&lt;/protected&gt;
            &lt;/property&gt;
            &lt;property name="app:rejectFolder"&gt;
               &lt;type&gt;d:noderef&lt;/type&gt;
               &lt;protected&gt;true&lt;/protected&gt;
            &lt;/property&gt;
            &lt;property name="app:rejectMove"&gt;
               &lt;type&gt;d:boolean&lt;/type&gt;
            &lt;/property&gt;
         &lt;/properties&gt;
      &lt;/aspect&gt;
</pre>

Each node that has a simple workflow attached to it has this aspect applied to it, which specifies through properties what steps are available, what that step should be called (acceptStep and rejectStep) and then what should occur when that step is taken: should it be moved or copied (acceptMove and rejectMove)? and where it should go (acceptFolder and rejectFolder).

Knowing this opens many doors for us. Because the steps are properties on the node, we can modify them, if needs be, using either javascript or Java. We can also build on top of this. When I worked on the Consulting team at Alfresco a customer wanted to have a three step workflow: accept, reject and other. Modeling this was simple. All we really needed to do was add three simple properties: the step name (text), the destination node (noderef) and if it is a move or copy (boolean). A bit more complex was adding the logic for exposing this through the Explorer UI. (This customer had not yet moved to Share). Because we did not want to affect all of the simple workflows already in place, we opted to add an additional 3 step simple workflow, giving them the ability to have up to 5 different paths that could be taken for a single node.

We can also tie simple workflows into more complex actions: dynamically adding simple workflows or scripting more complex actions that allow users to manually take the workflow steps but also allowing programatic lifecycle or state to take simple workflow actions. By jumping out of Explorer or Share and into a custom application you can expose simple workflows from the repository through a web script (an easier task than modifying JSF) and one that I&#8217;ve used for several different Alfresco implementations. We will cover that in my next post.

 [1]: http://www.activiti.org/