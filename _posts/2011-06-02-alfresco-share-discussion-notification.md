---
title: 'Alfresco: Share Discussion Notification'
author: jared
layout: post
permalink: /alfresco/alfresco-share-discussion-notification
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
---
A few months ago in the Alfresco Forums, a user asked about adding email notification to Share Site Discussions.  We went back and forth working out a rough way to accomplish this (some of this in private messages).  And then a couple of weeks ago the question came up internally.  I passed on what we had figured out and told them I would put together a blog post.

## **So what are we trying to accomplish?**

The Share Site Discussion Components supports the following:

<div>
  <ul>
    <li>
      Threaded Discussions &#8211; Post topic and replies
    </li>
    <li>
      Dynamic Filtering &#8211; Tags/New/Hot/All/My Topics
    </li>
    <li>
      RSS Feed for latest discussions
    </li>
  </ul>
</div>

Notifications of new posts can happen over RSS and to the Site Activities Dashlet both of which require access to the system, most likely behind a firewall.  Email notifications can provide updates the don&#8217;t require direct access to the system, unless you are looking to update the post.

## **Where to start?**

There are a few things that we need to know as we get started. First, when a site is created in Alfresco Share, the content nodes supporting discussions are not created.  Only after a user accesses the discussion component does it get created.

Second, discussions are created as `fm:topic` types.  Posts are added as children of `fm:topic` as a `fm:post`.

## **Using a Rule**

The simplest way to add notifications is as a rule via the Alfresco Explorer client on the discussion space that is created for the site, under the Sites space.  Adding a rule to the disucssions space matching all content items will result on two emails (one for the creation of the topic and one for the first post).  So we  need to be more specific: We need to expose the Forum Post and  Topic types as Subtypes to the rules engine.  You add fm:topic as well if you want to enable delete notifications

To `web-client-config-custom.xml` add

<pre class="brush: xml; title: ; notranslate" title="">&lt;alfresco-config&gt;
    &lt;config evaluator="string-compare" condition="Action Wizards"&gt;
        &lt;subtypes&gt;
            &lt;type name="fm:post"/&gt;
            &lt;type name="fm:topic"/&gt;
        &lt;/subtypes&gt;
    &lt;/config&gt;
&lt;/alfresco-config&gt;
</pre>

Topics are the best choice for building rules on.  Notifications on `fm:topic` will work for create and delete but not update.  The title of a topic is stored on the first post of a discussion.  You can&#8217;t delete individual posts but you can delete a Topic so when specifying a delete notification it can be made directly against a topic,but again&#8230;it is easier to just do the same using fm:post.

The notification template would be added to email templates and would look like:

<pre class="brush: plain; title: ; notranslate" title="">A new post is available in the '${document.parent.childAssocs["cm:contains"][0].properties.title}' dicussion in the '${document.parent.parent.parent.properties.name}' site, it was added by ${person.properties.firstName}&lt;#if person.properties.lastName?exists&gt; ${person.properties.lastName}&lt;/#if&gt;.

You can view it through this link:
${url.serverPath}/share/page/site/${document.parent.parent.parent.properties.name}/discussions-topicview?topicId=${document.parent.children[0].name}&listViewLinkBack=true
</pre>

## **Email Notification Rule**

An email rule is simple to setup but the maintenance cost can be high.  You have to add each user/group manually and Share site groups are not visible to the Explorer client.

The next option is to script the notification.   A Share site group or dynamically build email list of those participating in the discussion.  (A nice to have would be to build in a watch option for those who may not participate in the discussion but want to be notified.)

## **Mail action sample for group or individual.**

The `to_many` paramater currently only supports one group at a time&#8230;.if you need to send to multiple people or groups, you&#8217;ll need to loop over the `to_many` or `to` paramaters and execute against each.

<pre class="brush: jscript; title: ; notranslate" title="">var siteGroup = "GROUP_site_" + document.parent.parent.parent.name;
var template = "Data Dictionary/Email Templates/Notify Email Templates/share_discussion_notification.ftl";

// create mail action
var mail = actions.create("mail");
mail.parameters.to_many = siteGroup;
//mail.parameters.from (with no from address provided the email address of the user triggering the action is used)
mail.parameters.subject="New Post in Discussion: "+document.parent.childAssocs["cm:contains"][0].properties.title;
mail.parameters.template = companyhome.childByNamePath(template);

//execute action against a document
mail.execute(document);
</pre>

This option also will send email to all users in a group&#8230;even if their account has been disabled.

Another option when scripting is to dynamically build the notification list  &#8211; send only to those that are part of the discussion.  You can walk the posts through the Child Associations of the topic to build an array of users.  For smaller discussion threads there will not be that high of a cost. (Small here is a relative term, as I don&#8217;t have any metrics to say what is small).

<pre class="brush: jscript; title: ; notranslate" title="">var emailAddresses = [];

//for p expresion variable
var p, e, a;

//change to use your template
var template = "Data Dictionary/Email Templates/Notify Email Templates/share_discussion_notification.ftl";

function getEmail(person) {
	var personNode = people.getPerson(person);
	return personNode.properties.email;
}

// build emailAddresses
for (p = 0; p &lt; document.parent.childAssocs["cm:contains"].length; p += 1) {
	var user = document.parent.childAssocs["cm:contains"][p].properties.creator;
	var email = getEmail(user);

	//Is the emailAddress already in the array? If not, add it
	if (emailAddresses.length &gt; 0) {
		var match = false;
		for (e = 0; e &lt; emailAddresses.length; e += 1) {
			if (emailAddresses[e] === email) {
				match = true;
				break;
			}
		}

		if (!match) {
			emailAddresses.push(email);
		}
	} else {
		emailAddresses.push(email);
	}
}

// create mail action
var mail = actions.create("mail");
mail.parameters.subject = "New Post in Discussion: " + document.parent.childAssocs["cm:contains"][0].properties.title;
mail.parameters.template = companyhome.childByNamePath(template);

//send an email for each address
for (a = 0; a &lt; emailAddresses.length; a += 1) {
	mail.parameters.to = emailAddresses[a];
	//execute action against a document
	mail.execute(document);
}
</pre>

Another option would be to add an aspect to the topic that stores a collection of all the users who have participated in the thread.  (You could also add a place to collect watchers, but you&#8217;d also need to extend the UI to allow you to capture the users for this list.)  The aspect would be added on creation on a new topic. (Use Usernames, so that you can avoid  any issues with email addresses changing &#8212; Usernames are unmutable).  When a user adds a post, their email address is added to the collection for notification.

<pre class="brush: jscript; title: ; notranslate" title="">var creator = document.properties.creator;

//for expresion variable
var u, a;

//change to use your template
var template = “Data Dictionary/Email Templates/Notify Email Templates/share_discussion_notification.ftl”;

function getEmail(person) {
	var personNode = people.getPerson(person);
	return personNode.properties.email;
}

//Look for match in n:users, if no match add them other wise, ignore the user
function updateUsers(user) {
	if (document.parent.properties["n:users"] === null) {
		document.parent.properties["n:users"] = [];
		document.parent.properties["n:users"].push(user);
		document.parent.save();
	} else {
		var match = false;

		for (u = 0; u &lt; document.parent.properties["n:users"].length; u += 1) {
			if (document.parent.properties["n:users"][u] === user) {
				match = true;
				break;
			}
		}

		if (!match) {
			document.parent.properties["n:users"].push(user);
			document.parent.save();
		}
	}
}

//Check for notifiable aspect
if (document.parent.hasAspect("n:notifiable") {

	//if the user is not already in the n:users property of the notifiable aspect, add them
	updateUsers(creator);

	//create mail action
	var mail = actions.create(“mail”);
	mail.parameters.subject = “New Post in Discussion: ” + document.parent.childAssocs["cm:contains"][0].properties.title;
	mail.parameters.template = companyhome.childByNamePath(template);

	//send an email for each user in n:users
	for (a = 0; a &lt; document.parent.properties["n:users"].length; a += 1) {
		mail.parameters.to = getEmail(document.parent.properties["n:users"][a]);

		//execute action against a document
		mail.execute(document);
	}
} else {
	logger.log("The notifiable aspect has not been added to the topic: " + document.parent.name;
}
</pre>

These options should also take into account disabled accounts: (`person.isAccountEnabled(userName);` The problem with this is that `isAccountEnables` requires admin privileges.  There is no runAs for this kind of javascript so implementing this would require implementing the action in Java, which is outside the scope of this exercise.

Hopefully, I&#8217;ve given you some ideas of ways to approach adding discussion notifications.  The key here in discovering how to approach this was knowing where to look at how discussions are stored in the repository.  In fact this is the key for most any extension to Alfresco:  Asking yourself how does this work for this feature, which is similar to what I need, work in Alfresco? Understanding how this is pulled together from looking at the content model, looking at actual content/nodes through the node browser, understanding the relationships between nodes: parent/child and peer and sometimes digging into the source code can help to expand your understanding when looking to extend Alfresco.