---
title: 'Alfresco: JMX from the command line'
author: jared
layout: post
permalink: /alfresco/alfresco-jmx-from-the-command-line
aktt_notify_twitter:
  - yes
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - JMX
---
[JMX][1] is great. [Alfresco][2] and [JMX][3] is awesome. I&#8217;ve written before about [configuring Alfresco to use tunneling to connect JMX & debuggers][4] to servers that don&#8217;t allow access to the higher numbered ports (or to only a few ports).  Let&#8217;s add another cool tool. (JMX is an Enterprise only feature of Alfresco.)

The [Alfresco wiki][5] covers a few of the [clients][6] that are available out there.  Let&#8217;s add another type to the list:  JMX from the command line.  There are a [couple of options][7] for us to choose from.   I am partial to j[mxterm][8] from [CyclopsGroups.org][9]

Jmxterm is an opensource  JMX Client ([download][10]).  It supports auto completion, history browsing and scripting.  In a word: cool.

Let&#8217;s jump in&#8230;

First thing you need to do with the client is connect

With jmxterm you can pass a connection string via the jar to connect to Alfresco

<p style="padding-left: 30px">
  <code>java -jar jmxterm-1.0-alpha-4-uber.jar -l \ service:jmx:rmi:///jndi/rmi://&lt;host&gt;:50500/alfresco/jmxrmi -p &lt;password&gt; -u &lt;user&gt;</code>
</p>

Or, you can connect via the interactive shell

<p style="padding-left: 30px">
  <code>java -jar jmxterm-1.0-alpha-4-uber.jar</code>
</p>

Next, using the `open` command connect to Alfresco

<p style="padding-left: 30px">
  <code>$&gt;open service:jmx:rmi:///jndi/rmi://localhost:50500/alfresco/jmxrmi -p change_asap -u \ controlRole&lt;br />
#Connection to service:jmx:rmi:///jndi/rmi://localhost:50500/alfresco/jmxrmi is opened</code>
</p>

Or, using the `jvms` command (with the appropriate user permissions you can list the running java processes on the machine jmxterm is running on).

<p style="padding-left: 30px">
  <code>$&gt;jvms&lt;br />
94822    ( ) - jmxterm-1.0-alpha-4-uber.jar&lt;br />
13157    (m) - org.apache.catalina.startup.Bootstrap start</code>
</p>

And once again use the `open` command to connect

<p style="padding-left: 30px">
  <code>$&gt;open 13157 -u controlRole -p change_asap&lt;br />
#Connection to 13157 is opened</code>
</p>

Now that you are connected  let&#8217;s perform a simple operation, finding out who is logged into Alfresco and then disconnect one of the users.

First you&#8217;ll list the domains that are available

<p style="padding-left: 30px">
  <code>$&gt;domains&lt;br />
#following domains are available&lt;br />
Alfresco&lt;br />
Catalina&lt;br />
JMImplementation&lt;br />
Users&lt;br />
axis&lt;br />
com.sun.management&lt;br />
connector&lt;br />
java.lang&lt;br />
java.util.logging&lt;br />
log4j</code>
</p>

The bean you need is in the `Alfresco` domain

<p style="padding-left: 30px">
  <code>$&gt;domain Alfresco&lt;br />
#domain is set to Alfresco</code>
</p>

You can look up the beans within the domain by using the `beans` command.  There are quite a few beans in the `Alfresco` domain, so let&#8217;s connect directly to the one that you need: `RepoServerMgmt`. (Don&#8217;t forget jmxterm support auto completion.)

<p style="padding-left: 30px">
  <code>$&gt;bean Alfresco:Name=RepoServerMgmt&lt;br />
#bean is set to Alfresco:Name=RepoServerMgmt</code>
</p>

Now that you have the bean, you can now find out what it does by using the `info` command

<p style="padding-left: 30px">
  <code>$&gt;info&lt;br />
#mbean = Alfresco:Name=RepoServerMgmt&lt;br />
#class name = org.alfresco.repo.admin.RepoServerMgmt&lt;br />
# attributes&lt;br />
%0   - LinkValidationDisabled (boolean, r)&lt;br />
%1   - MaxUsers (int, r)&lt;br />
%2   - ReadOnly (boolean, r)&lt;br />
%3   - TicketCountAll (int, r)&lt;br />
%4   - TicketCountNonExpired (int, r)&lt;br />
%5   - UserCountAll (int, r)&lt;br />
%6   - UserCountNonExpired (int, r)&lt;br />
# operations&lt;br />
%0   - int invalidateTicketsAll()&lt;br />
%1   - int invalidateTicketsExpired()&lt;br />
%2   - void invalidateUser(java.lang.String p1)&lt;br />
%3   - [Ljava.lang.String; listUserNamesAll()&lt;br />
%4   - [Ljava.lang.String; listUserNamesNonExpired()&lt;br />
#there's no notifications</code>
</p>

There are lots of things you can do here, you can get attributes, you can set attributes, you can run opertaions.

Looking at the returned attributes you can tell a few things: the datatype of each attribute and if you can change it.  In this case none of our attributes can be modified, they are all read only (r).  ReadWrite would be signified by rw.

Operations are just like Java functions: they can have a return type and you might be able pass parameters to them.

It can also list notifcations, but there is no mechanism (that I have found yet) in jmxterm to subscribe or unsubscribe to them.

Let's do some more work:

First let's see how many users are logged in.

<p style="padding-left: 30px">
  <code>$&gt;get UserCountNonExpired&lt;br />
#mbean = Alfresco:Name=RepoServerMgmt:&lt;br />
UserCountNonExpired = 2;</code>
</p>

And now let's find out who these two users are

<p style="padding-left: 30px">
  <code>$&gt;run listUserNamesNonExpired&lt;br />
#calling operation listUserNamesNonExpired of mbean Alfresco:Name=RepoServerMgmt&lt;br />
#operation returns:&lt;br />
[ System, admin ]</code>
</p>

So you have 2 user currently in Alfresco: System and admin

Let&#8217;s log out the admin user.  (Look back at the list from the `info` command.) You can either invalidate all of the authenticated users tickets or we can invalidate named user.  Since you just want to remove one user let&#8217;s invalidate the admin users session.  (The System user is a special account, so you will ignore it.)

<p style="padding-left: 30px">
  <code>$&gt;run invalidateUser admin&lt;br />
#calling operation invalidateUser of mbean Alfresco:Name=RepoServerMgmt&lt;br />
#operation returns:&lt;br />
null</code>
</p>

Remember that this operation does not return anything so jmxterm returns `null`.

Let&#8217;s now check and see if the admin user is still logged in.

<p style="padding-left: 30px">
  <code>$&gt;run listUserNamesNonExpired&lt;br />
#calling operation listUserNamesNonExpired of mbean Alfresco:Name=RepoServerMgmt&lt;br />
#operation returns:&lt;br />
[ System ]</code>
</p>

You&#8217;ve now run through some basic operations: connecting to Alfresco, listing and selecting the domain, listing and selecting the bean, discovering what the bean can do, getting an attribute, running an operation.

The last things to cover are scripting/non-interactive mode and setting properties.

From time to time you might see the need, to lock Alfresco down by setting it to Read Only mode. The following two commands will help us do this.

First let&#8217;s check the current state:

<p style="padding-left: 30px">
  <code>$ echo get -s -b Alfresco:Name=RepoServerMgmt ReadOnly | java -jar jmxterm-1.0-alpha-4-uber.jar -l service:jmx:rmi:///jndi/rmi://localhost:50500/alfresco/jmxrmi -p change_asap -u controlRole -v silent -n</code>
</p>

When passing commands to jmxterm, you need to echo the command into the interpreter. So the above command passes the jmxterm get command.  we pass it a `-s` since we just want the value and not the full expression (`ReadOnly = <value>;`) returned `-b` names the bean and finally the attribute we are querying. On the jmxterm side we pass the connection information (connection string or PID, username and password), set the verbosity of the execution, and tell it that it should run in non-interactive mode.

In this case the command returns the value `false`. So the value of `ReadOnly` is `false` &#8212; Alfresco is in Read/Write mode.

Now let&#8217;s put it in ReadOnly mode

<p style="padding-left: 30px">
  <code>$ echo set -b Alfresco:Category=sysAdmin,Type=Configuration,id1=default server.transaction.allow-writes false | java -jar jmxterm-1.0-alpha-4-uber.jar -l service:jmx:rmi:///jndi/rmi://localhost:50500/alfresco/jmxrmi -p change_asap -u controlRole -v silent -n</code>
</p>

This follow pretty much the same pattern.  The `ReadOnly` attribute is a read only property, so to change Alfresco into Read Only mode you set the value of `server.transaction.allow-writes` to `false`. The set will not return any value.  You can then check the current state of the repository by running the previous command and then when you are ready to put the repository back in Read/Write mode change the value of the set command on `server.transaction.allow-writes` to `true`.

 [1]: http://java.sun.com/javase/6/docs/technotes/guides/jmx/index.html
 [2]: http://www.alfresco.com
 [3]: http://wiki.alfresco.com/wiki/JMX
 [4]: http://jared.ottleys.net/alfresco/tunneling-debug-and-jmx-for-alfresco
 [5]: http://wiki.alfresco.com
 [6]: http://wiki.alfresco.com/wiki/JMX#Connecting_through_JMX_Consoles_.2F_JSR-160
 [7]: http://www.google.com/search?sourceid=chrome&ie=UTF-8&q=jmx+command+line
 [8]: http://www.cyclopsgroup.org/projects/jmxterm/index.html
 [9]: http://www.cyclopsgroup.org/
 [10]: http://sourceforge.net/projects/cyclops-group/files/jmxterm/