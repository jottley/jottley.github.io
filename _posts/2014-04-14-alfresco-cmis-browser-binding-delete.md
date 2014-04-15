---
layout: post
title: 'Alfresco&#58; CMIS Browser Binding DELETE example'
author: jared
permalink: /alfresco/alfresco-cmis-browser-binding-delete
categories:
  - Alfresco
tags:
  - Alfresco
  - CMIS
  - example
---
I'm in the midst of rewriting a CMIS client. We are moving from CMIS 1.0 atompub bindings to CMIS 1.1 browser bindings. I've worked out how to do a DELETE and wanted to document an example that others could extrapolate from.
<br /><br />
A browser binding DELETE should follow the model of a form POST according to the documentation. Playing around with the provided examples (there were no DELETE examples that I found doing a quick google search) I found it to be a simpler than a form POST. I should also note that these calls are governed by the [same origin policy][1] so you'll need to use either JSON-P[2] or a CORS[3] enabled server[4] to enable building applications against the CMIS enabled repository.
<br /><br />
*I'm using curl for my examples.*
<br /><br />
There are two ways to address the object that you want to delete: using the objectId or by path.

### Using objectId
To delete by objectId we need to include the 'objectId=' parameter along with cmis objectId (ex. 7ab93c6a-9520-40dd-be74-bae52f4bd6ca;1.0).
{%highlight bash %}
&objectId=7ab93c6a-9520-40dd-be74-bae52f4bd6ca;1.0
{% endhighlight %}

### By Path
To delete by path you need to include the full path in the URL. There is no need to include the objectId in the by path call as the path has precedence over the objectId
{% highlight bash %}
Sites/internal2/documentLibrary/test.txt
{% endhighlight %}
Now with the basics of addressing the document or folder under our belt we can pull together the rest of the call.
<br /><br />
With the atompub binding we made a delete using the HTTP DELETE method. With the browser binding a delete is made using the HTTP POST method. To indicate our call is a delete we add the 'cmisaction=' parameter with a value of delete to our call.
{% highlight bash %}
cmisaction=delete
{% endhighlight %}
A full objectId call (made against Alfresco Cloud) would use a URL like the following:
{% highlight bash %}
https://api.alfresco.com/ottleys.net/public/cmis/versions/1.1/browser/root?cmisaction=delete&objectId=804f6300-38cd-48a4-86db-499f58497ae5;1.0
{% endhighlight %}
The same call using a path:
{% highlight bash %}
https://api.alfresco.com/ottleys.net/public/cmis/versions/1.1/browser/root/Sites/internal2/documentLibrary/test.txt?cmisaction=delete
{% endhighlight %}
Using the objectId example from above a call using curl against the Alfresco Cloud Public API would look like:
{% highlight bash %}
curl -X POST "https://api.alfresco.com/ottleys.net/public/cmis/versions/1.1/browser/root?cmisaction=delete&objectId=804f6300-38cd-48a4-86db-499f58497ae5;1.0" --header "Authorization:Bearer 42b8c319-344c-4273-b94c-b4c23482be6b" -IL
{% endhighlight %}
(For those unfamiliar with curl I've added the -IL options to the command so that I can see details about the response (not including the body).

### What about response codes?
With atompub a successful delete would have resulted in a 204 response code and no response body. With the browser bindings a 200 response code is returned still with no body. (My preference would be that it returned a 204. This would keep consistency across the bindings and would match what I believe is proper DELETE response.)
<br /><br />
And response codes for Errors? You still see a 401 when a user needs to authenticate and a 404 if the object can't be found. All other errors that I saw came back as 409s. This includes locked documents and folders containing children. If you receive a 409 result there are two json values you can look at to find the reason for the error: exception (the short description) and message (a fuller description of the error).




[1]: http://en.wikipedia.org/wiki/Same-origin_policy "Same Origin Policy"
[2]: http://en.wikipedia.org/wiki/JSONP "JSONP"
[3]: http://en.wikipedia.org/wiki/Cross-origin_resource_sharing "Cross-origin resource sharing"
[4]: http://www.slideshare.net/jottley/cors-enable-alfresco-for-cors "CORS enable Alfresco"