---
title: 'Alfresco: Property Decorators &#8212; cont.'
author: jared
layout: post
permalink: /alfresco/alfresco-property-decorators-cont
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
---
In my post on [Property Decorators][1] I pointed out that there was an [issue][2] adding a customer property decorator: you couldn&#8217;t create a custom bean to perform the mapping and had to overwrite the out of the box bean. Well [Mike Hatfield][3] has found a solution which is just awesome: [Map Merge][4].

It is possible to merge maps of a child bean to a parent bean. Here is how it is done: Create a new bean, where the parent is the `applicationScriptUtils` bean. Add a single property of `decoratedProperties` and define a map in the property. When you define the map add the attribute `merge="true"`. Finally, add your custom property to decorator bean mappings. Done!

Here is an example:

{% highlight xml %}
<bean id="customApplicationScriptUtils" parent="applicationScriptUtils">
   <property name="decoratedProperties">
      <map merge="true">
         <entry key="alf:propertyHolder">
            <ref bean="customDecoratorBean"/>
         </entry>
      </map>
   </property>
  </bean>
{% endhighlight %}

 [1]: http://jared.ottleys.net/alfresco/alfresco-property-decorators "Alfresco: Property Decorators"
 [2]: https://issues.alfresco.com/jira/browse/ALF-13038
 [3]: http://twitter.com/#!/mikehatfield
 [4]: http://forum.springsource.org/showthread.php?53358-Simple-merge-of-util-map