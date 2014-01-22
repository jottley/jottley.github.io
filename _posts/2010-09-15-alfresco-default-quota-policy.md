---
title: 'Alfresco: Default Quota Policy'
author: jared
layout: post
permalink: /alfresco/alfresco-default-quota-policy
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - AMP
  - behavior
  - Extension
---
*Updated: Added new tested versions*

For this post I want to share another policy from recent request from a customer.  The project was to help them develop a way to have usage quotas set to a default value when a new user/person was added to Alfresco. (There is an important distinction between the two.)  A few months ago I had a discussion about possible ways to implement this kind of functionality with a co-worker and had a few ideas brewing as we started the engagement.

First some background on quotas:

*   The default quota in Alfresco is unlimited.  In other words, there is no limit in the amount (total size) of content a user can add/own in Alfresco.
*   A quota can be set either on creation or at anytime during a users lifetime. 
    *   From within the UI a quota can be in either GB, MB or KB.
    *   From an API  (Web Script, JavaScript, Java) it accepts the size in bytes.
*   See [http://wiki.alfresco.com/wiki/Usages\_and\_Quotas][1] for additional details

When developing a behavior I continually reference [Jeff Potts&#8217;s][2] tutorial on [implementing behaviors][3]. His table of available policies is a great quick reference.  Of course the definitive source is the Alfresco source code, but not much has changed since Jeff wrote the article.

**JavaScript Behavior**

The first attempt to implementing this policy was to use a behavior implemented in JavaScript.  I used Jeff&#8217;s JavaScript example as the seed for my code.  In fact there was very little I needed to change (mostly just the business logic). It is a great outline to get you going.

Using JavaScript wasn&#8217;t completely successful.  The code did work up to a point and that point was setting the quota.  There is an undocumented (in the wiki) JS function that can be used to set quotas.

From the People class (note the comment):

<pre class="brush: jscript; title: ; notranslate" title="">/**
 * Set the content quota in bytes for a person.
 * Only the admin authority can set this value.
 *
 * @param person    Person to set quota against.
 * @param quota     As a string, in bytes, a value of "-1" means no quota is set
*/
public void setQuota(ScriptNode person, String quota)</pre>

During the transaction the quota was being set, but once the transaction was committed it was lost. The problem being (again see the comments) that the setQuota must be executed by an admin.  A quota must be set explicitly by an admin.

The JavaScript API lacks the ability to execute a script as one user but then run specific code in that script as another user.  This is done for [security reasons][4] (like being able to keep a user from uploading and then executing a malicious script).

While the script did not work as desired it still may be useful for something else.  So I&#8217;m including links to the [context file][5] and [script][6] as part of this post.

**Java Behavior**

Because we need to  set the quota as an admin user, we&#8217;ll switch to Java which provides a useful means to run code as one user, and execute specific parts as a different user.

Before we jump to that, let&#8217;s talk about some of the specifics of our policy:

First, there is not much difference in the structure needed for this behavior and the [Max Version Policy][7] I wrote about in a [previous post][8].  So I used it as a template for this project.

Next, users/people in Alfresco are stored as nodes.  If you start to dig into [the node browser][9] you may be drawn to find your users in the user://alfrecoUserStore store.  This is, as the name suggestions, the Alfresco User Store which is used as the native Alfresco authentication source ([alfrescoNTLM][10]).  The nodes in this store are of type {http://www.alfresco.org/model/user/1.0}user.  Quotas are a property of {http://www.alfresco.org/model/content/1.0}person and are found in the workspace://SpacesStore under system -> people.

Alfresco has no specific policies for user nodes (like an onCreateUser policy), but since they are just nodes we can leverage the onCreateNode policy. Also because they are just nodes, when using an onCreateNode policy we don&#8217;t want the policy to be run every time a node is created.  We want to target cm:person nodes.  Otherwise, we would need to overcomplicate the code with additional tests to only only execute the business logic when the newly created node is of type cm:person.  So we can bind our policy to a specific type.  Similarly to the max version policy we initialize our behavior and bind it in the init method.

<pre class="brush: java; title: ; notranslate" title="">public void init() {

this.onCreateNode = new JavaBehaviour(this, "onCreateNode", NotificationFrequency.TRANSACTION_COMMIT);

this.policyComponent.bindClassBehaviour(QName.createQName(NamespaceService.ALFRESCO_URI, "onCreateNode"), ContentModel.TYPE_PERSON, this.onCreateNode);

}</pre>

In the init() above we bind our policy: QName.createQName(NamespaceService.ALFRESCO\_URI, &#8220;onCreateNode&#8221;), the type: ContentModel.TYPE\_PERSON and the behavior: this.onCreateNode together.

Next we need to implement the onCreateNode method from the OnCreateNodePolicy in our new class

<pre class="brush: java; title: ; notranslate" title="">public void onCreateNode(ChildAssociationRef childAssocRef) {</pre>

We can grab our reference to the newly created user node from the ChildAssociationRef parameter.

<pre class="brush: java; title: ; notranslate" title="">final NodeRef user = childAssocRef.getChildRef();</pre>

Notice that it is using the final modifier.  Variables from an outer class that are being referenced in an inner class must be defined as final.

For our default policy we also want to allow the default quota to be overwritten.  If an admin wants to set a different value for the quota at creation time we need to allow it. So we need to first get the value that was assigned to the user on the node at creation

<pre class="brush: java; title: ; notranslate" title="">long currentQuota = contentUsageService.getUserQuota((String) nodeService.getProperty(user,ContentModel.PROP_USERNAME));</pre>

The default value of no quota is -1.  If an admin has set a value for the quota it will be greater than 0.

<pre class="brush: java; title: ; notranslate" title="">if (currentQuota &lt; 0) {

...business logic here...

}</pre>

Because quotas can only be set by an admin we need to use the runAs utility

<pre class="brush: java; title: ; notranslate" title="">AuthenticationUtil.runAs(

new AuthenticationUtil.RunAsWork&lt;Object&gt;() {

public Object doWork() throws Exception {

...code to be run as a different user...

}

}, "admin");</pre>

runAs takes two parameters: The first is the RunAsWork Object, which contains an inner class that has the code that needs to be run as another user.  Second the name of the user to execute the code as.

Now in our policy we can set the quota.  We&#8217;ll use the contentUsageService to set the new users quota, taking the username and the value of the quota as parameters. Also note that default value is in bytes which is read from the spring context file for the OTB default I&#8217;ve chosen 2GB (2147483648).

<pre class="brush: java; title: ; notranslate" title="">contentUsageService.setUserQuota((String) nodeService.getProperty(user, ContentModel.PROP_USERNAME), Long.parseLong(defaultQuota));

return user; </pre>

RunAsWork is also looking for a return value which I&#8217;ve set as user.  We won&#8217;t being doing anything with this value so in this case it is an arbitrary return.

**Installing**

The policy is [packaged as an AMP][11] and is downloadable from the [alfresco-defaultquota-policy project][12] hosted on Google Code.  (See [http://wiki.alfresco.com/wiki/Module\_Management\_Tool][13] for detailed instructions on installing AMPs.)

The amp has been tested with Alfresco Enterprise 3.1.1 to 3.3.2.

I&#8217;ve only tested this on Enterprise 3.3.2.  I&#8217;ll look to expand my testing soon.  If you have a chance to try it on another version of Alfresco please let me know.  (In that case you&#8217;ll need to build the amp from source, changing the version.min/max property to include the version you&#8217;re targeting.)

**So what next?**

While this policy is limited to setting the user quota, your probably thinking what else can I use this for? I know I am!  Maybe your using a database or some other none supported user store in Alfresco and you want to populate the user node properties with details from that system.  Or you want to add them to specific groups based on some complex business logic.  Or interact with external systems when a user is created in Alfresco.  What are your ideas?

I also believe that this should work with any external authentication/synchronization of users into Alfresco.  I&#8217;ve not tested it, but it is the list of things to test next.

I&#8217;m always looking for feedback: let me know what worked or didn&#8217;t for you.

 [1]: http://wiki.alfresco.com/wiki/Usages_and_Quotas
 [2]: http://ecmarchitect.com/
 [3]: http://ecmarchitect.com/archives/2007/09/26/770
 [4]: http://forums.alfresco.com/en/viewtopic.php?f=6&t=28475&start=0
 [5]: http://svn.ottleys.net/alfresco/misc_scripts/js_behavior/behaviour-context.xml
 [6]: http://svn.ottleys.net/alfresco/misc_scripts/js_behavior/behaviour.js
 [7]: http://code.google.com/p/alfresco-maxversion-policy/
 [8]: http://jared.ottleys.net/alfresco/alfresco-max-version-policy
 [9]: http://www.alfresco.com/help/webclient/tasks/tuh-admin-nodebrowser.html
 [10]: http://wiki.alfresco.com/wiki/Authentication_Subsystems#AlfrescoNtlm
 [11]: http://code.google.com/p/alfresco-defaultquota-policy/downloads/detail?name=alfresco-defaultquota-policy.amp
 [12]: http://code.google.com/p/alfresco-defaultquota-policy/
 [13]: http://wiki.alfresco.com/wiki/Module_Management_Tool