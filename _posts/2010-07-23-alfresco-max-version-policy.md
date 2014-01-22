---
title: 'Alfresco: Max Version Policy'
author: jared
layout: post
permalink: /alfresco/alfresco-max-version-policy
aktt_notify_twitter:
  - yes
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - AMP
  - Versioning
---
*UPDATE:  Updated code examples to match updates to source code*

As part of a POC for a customer I was asked to write an extension that allowed them to control the total number of versions allowed per versioned content. (Download links at bottom of page)

Alfresco has a strong versioning story, that gives you the ability to version any content stored in the repository, no matter what the file type.  Versions are full files and not diffs of the files.  Alfresco gives you the ability to have both major and minor versions of content.  Versions can be created/updated by checkout/checkin, by rule, through any interface or through script/APIs.

Alfresco also provides the ability to apply behaviors / policies to content/metadata within the repository.  You can think of these as event listeners, that allow you to take custom actions based on what is happening within the repository.  [Jeff Potts][1] has written an excellent [tutorial on creating behaviors][2], with examples for both Java and javascript. (Other resources include: )

There are 4 versioning policies that we can work with:

*   beforeCreateVersion
*   afterCreateVersion
*   onCreateVersion
*   calculateVersionLabel

(For an [example of a calculateVersionLabel policy][3] look at this post and the accompanying code by [Peter][4] [Monks][5].)

For this policy we are going to use afterCreateVersion:  after a version is created we want to remove any version that puts us past the max version value.

You start by implementing the [policy interface][6] for the policy you want to apply.

<p style="padding-left: 30px">
  <code>public class MaxVersionPolicy implements AfterCreateVersionPolicy</code>
</p>

Adding the method implemented by the interface

` `

` `

`</p>
<pre style="padding-left: 30px">public void afterCreateVersion(
				NodeRef versionableNode,
				Version version)</pre>
<p>`

We also need to [bind][7] the [behavior][8] to the policy and we need to do this when the class is loaded by the springframework.  We wrap this registration in an init() method

` `

` `

`</p>
<pre style="padding-left: 30px">	public void init() {
		this.afterCreateVersion = new JavaBehaviour(this, "afterCreateVersion",
				NotificationFrequency.TRANSACTION_COMMIT);

		this.policyComponent.bindClassBehaviour(QName.createQName(
				NamespaceService.ALFRESCO_URI, "afterCreateVersion"),
				MaxVersionPolicy.class, this.afterCreateVersion);
	}</pre>
<p>`

The init method is then called when spring loads the bean

` `

` `

`</p>
<pre>		<bean id="maxVersion"
                      class="org.alfresco.extension.versioning.MaxVersionPolicy"
                      init-method="init">
			<property name="policyComponent">
				<ref bean="policyComponent" />
			</property>
			<property name="versionService">
				<ref bean="versionService" />
			</property>
			<!-- The max number of versions per versioned node -->
			<property name="maxVersions">
				<value>10</value>
			</property>
		</bean></pre>
<p>`

Now to the meat of the policy, enforcing the removal of versions.

First we we have our maxVersion property.  This is set in the springbean (see above) and read when the bean is loaded. (You can overwrite the default value by copying the maxversionpolicy-context.xml file into the extensions directory and changing the value of the maxVersion property)

` `

` `

`</p>
<pre>	public void setMaxVersions(int maxVersions) {
		this.maxVersions = maxVersions;
	}</pre>
<p>`

Next we want to remove any version that puts us over the max

` `

` `

`</p>
<pre> 	@Override
	public void afterCreateVersion(NodeRef nodeRef, Version version) {

		VersionHistory versionHistory = versionService
				.getVersionHistory(nodeRef);

		// If the current number of versions in the VersionHistory is greater
		// than the maxVersions limit, remove the root/least recent version
		if (versionHistory.getAllVersions().size() > maxVersions) {
			logger.debug("Removing Version: "
					+ versionHistory.getRootVersion().getVersionLabel());
			versionService.deleteVersion(nodeRef, versionHistory
					.getRootVersion());
		}
	}</pre>
<p>`

We first get the [versionHistory][9] for the node and check it against the maxVersion property. If we have more versions than the limit we delete the [least recent][10] [version][11] from the version history.

<div id="attachment_363" style="width: 803px" class="wp-caption aligncenter">
  <img class="size-full wp-image-363" src="http://jared.ottleys.net/files/2010/07/Screen-shot-2010-07-23-at-12.45.06-AM1.png" alt="10 Version Limt" width="793" height="292" /><p class="wp-caption-text">
    Notice: 10 total versions; least recent version is 1.1; no pagination!
  </p>
</div>

> **Q:** Why are you using an if statement, instead of looping through all of the versions.  I have a few (many) more versions over the limit I want to impose?
> 
> **A:** A while statement would be more efficient in removing everything above the limit, but the versionHistory object isn&#8217;t being updated with each delete in a loop (I tried) until the policy has completely run.  Luckily, as you will find as you implement custom behaviors, they get called a lot!  So while a single update will result in a single version added, you will actually see that the behavior is called multiple times for that one action (In testing I saw the behavior being called a minimum of 7 times).  While you might not clear out every version over the limit with a single update, you will see a good many of them removed.  If you want to bring every down to the limit you could create a &#8220;cleaner&#8221; extension that could go in and remove all of the versions over the limit (this would be a very intensive/costly operation depending on the amount of content in your repository and the size of your version history  &#8211;  There several different strategies for this).  You could then use this extension to enforce that limit.

> **Q:** Does this affect all versioned content in the repository?  What if I only want it to work on some content?
> 
> **A:** Yes, by default this policy is being applied to all versioned content, but you could add in checks to look for a specific aspect, parent space name, property, etc. before passing through the delete code.

> **Q:** Compliance regulations/laws don&#8217;t allow me to delete the versions, but require me keep them for X years. How can I archive versions?
> 
> **A:** There are probably several different ways to handle this.  One way could be to use Alfresco&#8217;s [content store selector][12].  The content store selector allows you to configure multiple underlying filesystem locations for your content.  To the end user all of the content appears to come from the same location, while the content itself lives in different disc systems. The idea would be to set up a secondary store that would act as the archive.  When your trigger is reached, be it a property (status [property or aspect], date, etc.), a version number/count, etc. have the policy move the content to the archive space structure (The archive will need to have the cm:storeSelector aspect set to mark it to use the secondary store). Because we are working from the least recent version, the 1.0 version of the content, it becomes the first version in the archive, each subsequent version, is added as a version for the content, version numbers are then maintained in the archive. I would also add an aspect to the original node that contains a pointer/aspect to the archived version, for reference. Then delete the least recent version of the original versioned node.

Google Code Project: <http://code.google.com/p/alfresco-maxversion-policy/>

Download [Alfresco Max Version Policy AMP][13]

Tested on Alfresco (Enterprise) 3.2 &#8211; 3.3.1

 [1]: http://ecmarchitect.com/
 [2]: http://ecmarchitect.com/images/articles/alfresco-behavior/behavior-article.pdf
 [3]: http://blogs.alfresco.com/wp/pmonks/2009/11/03/version-baselining/
 [4]: http://blogs.alfresco.com/wp/pmonks/
 [5]: http://twitter.com/pmonks
 [6]: http://dev.alfresco.com/resource/docs/java/repository/org/alfresco/repo/version/VersionServicePolicies.OnCreateVersionPolicy.html
 [7]: http://dev.alfresco.com/resource/docs/java/repository/org/alfresco/repo/policy/PolicyComponent.html
 [8]: http://dev.alfresco.com/resource/docs/java/repository/org/alfresco/repo/policy/JavaBehaviour.html
 [9]: http://dev.alfresco.com/resource/docs/java/repository/org/alfresco/service/cmr/version/VersionHistory.html
 [10]: http://dev.alfresco.com/resource/docs/java/repository/org/alfresco/service/cmr/version/VersionService.html
 [11]: http://dev.alfresco.com/resource/docs/java/repository/org/alfresco/service/cmr/version/Version.html
 [12]: http://wiki.alfresco.com/wiki/Content_Store_Selector
 [13]: http://code.google.com/p/alfresco-maxversion-policy/downloads/detail?name=alfresco-maxversion-policy.amp