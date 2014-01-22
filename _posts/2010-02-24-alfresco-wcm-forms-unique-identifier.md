---
title: 'Alfresco WCM Forms: Unique Identifier'
author: jared
layout: post
permalink: /alfresco/alfresco-wcm-forms-unique-identifier
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - Java
  - WCM
  - web script
---
From time to time I&#8217;ve seen the need to provide a unique identifier in content created through the Alfresco WCM Forms Service.  The reason why has varied from needing an unique identifier across renditions to needing a way to tie the content to User Generated Content (UGC) or some other database managed application. A simple solution is to leverage Java generated [Universally Unique Identifiers][1] (UUID) imported into the [Schema Definition of your form][2] via a [Java Backed Web Script][3]. **Note: **These are not the UUIDs used by Alfresco DM.  UUIDs are not available/part of the AVM/WCM model used by Alfresco.

Java Backed Web Scripts provide a powerful way to include complex business process logic, enabling integrations through the [Alfresco Web Script framework][4] using a simple HTTP based API.  In this case, we are using a simple Java class to generate a UUID and then returning that UUID wrapped in a XML Scheme Definition imported through a <xs:include> into a Schema Definition used to generate a form used to capture content in Alfresco&#8217;s WCM service.

First let&#8217;s tackle the Java portion.  This is a simple Java class that extends the Alfresco [DeclarativeWebScript][5] class. A Declarative Web Script allows you to mix Java classes, Freemarker templates (FTL) and Javascript.  In this case we a want to leverage a FTL, which generates the XSD for the return.  Here is the meat of our class:

<p style="padding-left: 30px">
  <code>public class UUIDWebScript extends DeclarativeWebScript&lt;br />
{</code>
</p>

<p style="padding-left: 60px">
  <code>@Override&lt;br />
protected Map executeImpl(WebScriptRequest req,&lt;br />
Status status, Cache cache)&lt;br />
{</code>
</p>

<p style="padding-left: 90px">
  <code> Map model = new HashMap();</code>
</p>

`</p>
<p style="padding-left: 90px">UUID uuid = UUID.randomUUID();</p>
<p style="padding-left: 90px">model.put("uuid", uuid.toString());</p>
<p style="padding-left: 90px">return model;</p>
<p style="padding-left: 60px">}</p>
<p>`

<p style="padding-left: 30px">
  <code>}</code>
</p>

Basically,

*   Create a `map` of properties you want to return (by coding conventions this is called a model).
*   Perform your business logic, in our case generate a UUID.
*   Add the values you want to return to the template with a unique name to identify it in the template
*   Return the map of properties

Next we want to format our return.  This is done using a Freemarker Template . ([Freemarker][6] is a Java Based markup language.) Here is ours:

<p style="padding-left: 30px">
  <code> &lt;xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"&lt;br />
xmlns:alfresco="http://www.alfresco.org/alfresco"&lt;br />
elementFormDefault="qualified"&gt;</code>
</p>

<p style="padding-left: 60px">
  <code>&lt;xs:complexType name="uuid"&gt;</code>
</p>

<p style="padding-left: 90px">
  <code>&lt;xs:sequence&gt;</code>
</p>

<p style="padding-left: 120px">
  <code> &lt;xs:element name="uuid" fixed="${uuid}" type="xs:normalizedString"/&gt;</code>
</p>

<p style="padding-left: 90px">
  <code> &lt;/xs:sequence&gt;</code>
</p>

<p style="padding-left: 60px">
  <code> &lt;/xs:complexType&gt;</code>
</p>

<p style="padding-left: 30px">
  <code> </code>
</p>

<p style="padding-left: 30px">
  <code>&lt;/xs:schema&gt;&lt;br />
</code>
</p>

The FTL, as you can see , is a simple XSD.  The value returned from our Java class, in the model, named uuid, is declared as ${uuid}.

Now let&#8217;s declare the web script so that it can be registered in Alfresco:

<p style="padding-left: 30px">
  <code>&lt;webscript&gt;</code>
</p>

<p style="padding-left: 60px">
  <code> &lt;shortname&gt;Generate UUID&lt;/shortname&gt;&lt;br />
&lt;description&gt;Returns a UUID&lt;/description&gt;&lt;br />
&lt;url&gt;/util/uuid&lt;/url&gt;&lt;br />
&lt;authentication&gt;none&lt;/authentication&gt;&lt;br />
&lt;format default="xml"&gt;argument&lt;/format&gt;</code>
</p>

<p style="padding-left: 30px">
  <code> &lt;/webscript&gt;</code>
</p>

And since we are using a Java class in our Web Script, we also need to declare it so that the Web Script framework can use it:

<p style="padding-left: 30px">
  <code>&lt;?xml version="1.0" encoding="UTF-8"?&gt;&lt;br />
&lt;beans xmlns="http://www.springframework.org/schema/beans"&lt;br />
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"&lt;br />
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"&gt;</code>
</p>

`</p>
<p style="padding-left: 60px"><bean id="webscript.org.alfresco.extension.uuid.uuid.get"<br />
class="org.alfresco.extension.uuid.UUIDWebScript"<br />
parent="webscript"><br />
</bean></p>
<p>`

<p style="padding-left: 30px">
  <code>&lt;/beans&gt;</code>
</p>

It is important to note here that the bean id for a Java backed Web Script requires a very specific format:

*   Start by declaring this bean a webscript with: webscript.
*   Next add the full Java package declaration: org.alfresco.extension.uuid
*   Followed by the name used in your FTL and XML declaration with the method used to access the web script: uuid.get
*   When packaged, the FTL and XML declaration must be in a folder structure that matches the Java package declaration.

This puts together all of the pieces on the Web Script side.  It is easiest to manage this through an [AMP file][7], which we won&#8217;t discuss here how to create or install, but [code][8] and [amp file][9] are available at the project page: <http://code.google.com/p/alfresco-uuid-webscript/>

Once installed we can verify that it works by going to http://localhost:8080/alfresco/service/util/uuid in your browser (replace localhost and port as needed for your install).  What you should see returned is an XSD that looks something like this:

<p style="padding-left: 30px">
  <code>&lt;xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"&lt;br />
xmlns:alfresco="http://www.alfresco.org/alfresco"&lt;br />
elementFormDefault="qualified"&gt;</code>
</p>

`</p>
<p style="padding-left: 60px"><xs:complexType name="uuid"></p>
<p style="padding-left: 90px"><xs:sequence></p>
<p style="padding-left: 120px"><xs:element name="uuid" fixed="9c2fce39-3351-47c9-bb05-4b002967a00d" type="xs:normalizedString"/></p>
<p style="padding-left: 90px"></xs:sequence></p>
<p style="padding-left: 60px"></xs:complexType></p>
<p>`

<p style="padding-left: 30px">
  <code>&lt;/xs:schema&gt;</code>
</p>

It is our template but with are declaration of ${uuid} replaced with aUUID: 9c2fce39-3351-47c9-bb05-4b002967a00d.  With each call to this  Web Script you will see a uniquely generated ID.

Now let&#8217;s integrate this with a simple XSD:

<p style="padding-left: 30px">
  <code>&lt;?xml version="1.0" encoding="UTF-8"?&gt;&lt;br />
&lt;xs:schema&lt;br />
xmlns:alf="http://www.alfresco.org"&lt;br />
xmlns:xs="http://www.w3.org/2001/XMLSchema"&lt;br />
xmlns:sample="http://www.alfresco.org/alfresco/sample"&lt;br />
elementFormDefault="qualified"&lt;br />
targetNamespace="http://www.alfresco.org/alfresco/sample"&gt; </code>
</p>

<p style="padding-left: 30px">
  <code>&lt;xs:include schemaLocation="webscript://util/uuid" /&gt;</code>
</p>

<p style="padding-left: 30px">
  <code> </code>
</p>

<p style="padding-left: 30px">
  <code>&lt;xs:element name="sample"&gt;</code>
</p>

<p style="padding-left: 30px">
  <p>
    <code>&lt;/p>
&lt;p style="padding-left: 30px">&lt;xs:complexType&gt;&lt;/p>
&lt;p style="padding-left: 60px">&lt;xs:sequence&gt;&lt;/p>
&lt;p style="padding-left: 90px">&lt;xs:element name="sample" type="xs:normalizedString" minOccurs="1" maxOccurs="1"/&gt;&lt;br />
&lt;xs:element name="key" type="sample:uuid"/&gt;&lt;/p>
&lt;p style="padding-left: 60px">&lt;/xs:sequence&gt;&lt;/p>
&lt;p style="padding-left: 30px">&lt;/xs:complexType&gt;&lt;/p>
&lt;p style="padding-left: 30px">&lt;/xs:element&gt;&lt;/p>
&lt;p></code>
  </p>
  
  <p style="padding-left: 30px">
    <code>&lt;/xs:schema&gt;</code>
  </p>
  
  <p>
    The important thing here is how we reference the Web Script, and you can do this with any custom Web Script designed to render XML Schema to be included in a form,  by referencing the call with the declaration of webscript:
  </p>
  
  <p style="padding-left: 30px">
    <code>&lt;xs:include schemaLocation="webscript://util/uuid" /&gt;</code>
  </p>
  
  <p>
    Add this XSD to your Web Project and give it a run! In the form you will see the UUID, as a read-only field.
  </p>
  
  <p style="text-align: center">
    <img class="size-full wp-image-299 aligncenter" src="http://jared.ottleys.net/files/2010/02/Picture-1.png" alt="UUID in Web Form" width="472" height="128" />
  </p>
  
  <p style="text-align: left">
    Now in your XML output you will have a unique key that you can reference in your templates.
  </p>

 [1]: http://en.wikipedia.org/wiki/Universally_Unique_Identifier
 [2]: http://wiki.alfresco.com/wiki/Forms_Authoring_Guide
 [3]: http://wiki.alfresco.com/wiki/Web_Scripts#Java-Backed_Implementations
 [4]: http://wiki.alfresco.com/wiki/Web_Scripts
 [5]: http://dev.alfresco.com/resource/docs/java/web-client/org/alfresco/web/scripts/DeclarativeWebScript.html
 [6]: http://freemarker.org/
 [7]: http://wiki.alfresco.com/wiki/AMP_Files
 [8]: http://code.google.com/p/alfresco-uuid-webscript/source/checkout
 [9]: http://alfresco-uuid-webscript.googlecode.com/files/alfresco-uuid-webscript.amp