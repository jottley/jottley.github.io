---
title: 'Alfresco: Property Decorators'
author: jared
layout: post
permalink: /alfresco/alfresco-property-decorators
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
---
**Note**: *After fighting through some upgrade issues with MySQL&#8230;and it still not being 100% fixed&#8230;I&#8217;m finally at a point where I can comfortably write to the DB and bring to you two, thats right two!, new posts in quick succession.  I&#8217;m hoping to make this a trend to have two new posts a month (so look for 4 this month as these two should have gone live last month). So without further ado&#8230;.*

One of the exciting, at least in my opinion, new features of Alfresco 4 is the [Share Document Library Extension&#8217;s][1] (hat tip to [Mike Hatfield)][2].  In the above post Mike goes in to detail on how to use this new feature and I&#8217;m going to share an example on how I&#8217;m using it as part of integration with Dropbox I&#8217;m working on.

### **The Problem Set**

As part of the integration we need to add a status indicator (see the [above article ][1]for in depth details on status indicators) to, as the name states, indicate that the file/folder is in a specific state, ie synced to Dropbox.  Early on this was a simple tasks: does it have a specific aspect or not.  As the project has moved forward, we&#8217;ve raised the complexity: does it have a specific association and does that association have a child association of a specific value&#8230;a direct correlation between this child association and a specific user.  Now that we have moved beyond what the out of the box evaluators can do directly, what are our options?

### Options?

The Share Document Library Extension Framework provides two options for retrieving additional metadata/values from the repository tier: Custom Responses and Property Decorators.

Custom Responses allow you to return information that is not specific to any node.  Alfresco 4 uses this to return information about our Sharepoint Protocol integration.

Property Decorators allow you to return node specific information in a more usable format to the web tier (ie Share).  For example rewriting a nodeRef to a filename, or a username as first and last name. Or as in our case, a new set of properties or key/value pairs.

Since we are looking for specific information about a node, a property decorator is what we will need.

### Implemention

A property decorator needs to have the following:

A content model property to hold a map of properties. In our model we added a property similar to:

<pre class="brush: xml; title: ; notranslate" title="">&lt;property name="alf:propertyHolder"&gt;
    &lt;type&gt;d:boolean&lt;/type&gt;
    &lt;default&gt;false&lt;/default&gt;
    &lt;index enabled="false"/&gt;
&lt;/property&gt;
</pre>

This property will never be needed in the index since it will always be the same value until it is requested in Share.

Next we need to implement our logic.  This requires a new Java class that implements PropertyDecorator.

<pre class="brush: java; title: ; notranslate" title="">public class CustomProperty
    implements PropertyDecorator
{

  //Add any needed services

    @Override
    public Serializable decorate(NodeRef nodeRef, String propertyName, Serializable value)
    {
        Map&lt;String, Serializable&gt; map = new LinkedHashMap&lt;String, Serializable&gt;(4);

        //Add logic here

        //One to many
        map.put("key", value);

        return (Serializable)map;
    }

}
</pre>

This also requires a new Spring bean definition.

<pre class="brush: xml; title: ; notranslate" title="">&lt;bean id="customProperty" class="org.alfresco.extension.repo.jscript.app.CustomProperty"&gt;
   &lt;!-- Any needed service --&gt;
  &lt;/bean&gt;
</pre>

Lastly we need to add it to the list of propertyDecorators.  ***Update:** I&#8217;ve add a [new post][3] that covers how the mapping should occur and added a new bean here.* <del datetime="2012-04-02T20:26:51+00:00">Currently this requires overwriting the out of the box definition for applicationScriptUtils.  This could cause issues if you have several different AMPs that need to overwrite this bean. (I&#8217;ve opened ALF-13038 for this issue).<br /> </del>

<pre class="brush: xml; title: ; notranslate" title="">&lt;bean id="customApplicationScriptUtils" parent="applicationScriptUtils"&gt;
 &lt;property name="decoratedProperties"&gt;
    &lt;map merge="true"&gt;
       &lt;entry key="alf:propertyHolder"&gt;
          &lt;ref bean="customDecoratorBean"/&gt;
       &lt;/entry&gt;
    &lt;/map&gt;
 &lt;/property&gt;
&lt;/bean&gt;
</pre>

Now this completes the repo side of our extension.  Now we need to work with the values returned on the Share side.  We needr a status indicator so we&#8217;ll focus there.

First we&#8217;ll define the evaluator.  We need to add the evaluator to `web-extension/custom-slingshot-something-context.xml`

<pre class="brush: xml; title: ; notranslate" title="">&lt;bean id="evaluator.doclib.indicator.custom" parent="evaluator.doclib.metadata.value"&gt;
      &lt;property name="accessor" value="properties.customProperty"/&gt;
	  &lt;property name="comparator"&gt;
         &lt;bean class="org.alfresco.web.evaluator.StringEqualsComparator"&gt;
            &lt;property name="value" value="true" /&gt;
         &lt;/bean&gt;
      &lt;/property&gt;
   &lt;/bean&gt;
</pre>

It is important to note that the value of the accessor is not the content model property name, but the key value used in our map from the CustomProperty class.

And finally, we use the evaluator in our indicator config. This is added to our `share-config-custom.xml` under the `META-INF` in our custom jar file.

<pre class="brush: xml; title: ; notranslate" title="">&lt;config evaluator="string-compare" condition="DocumentLibrary"&gt;
		&lt;indicators&gt;
			&lt;indicator id="customDecorator" index="250" icon="custom-16.png"&gt;
				&lt;evaluator&gt;evaluator.doclib.indicator.custom&lt;/evaluator&gt;
			&lt;/indicator&gt;
		&lt;/indicators&gt;
&lt;/config&gt;
</pre>

In conclusion, the Share Document Library Extension Framework is a powerful new tool in the developer toolbox.

 [1]: http://blogs.alfresco.com/wp/mikeh/2011/09/26/share-document-library-extensions-in-v4-0/
 [2]: https://twitter.com/#!/mikehatfield
 [3]: http://jared.ottleys.net/alfresco/alfresco-property-decorators-cont "Alfresco: Property Decorators — cont."