---
title: 'UPDATED: Tunneling Debug and JMX for Alfresco'
author: jared
layout: post
permalink: /alfresco/updated-tunneling-debug-and-jmx-for-alfresco
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
tags:
  - Alfresco
  - JMX
  - Putty
  - SSH
  - Windows
---
Back in February I wrote a post on [Tunneling Debug and JMX with Alfresco][1].  Here are a few updates to that post:

0/ For JMX: From Alfresco 3.2 sp1 (enterprise release) on you no longer need to add the custom-core-services-context.xml file.  Instead in the alfresco-global.properties file add monitor.rmi.services.port=50508 to set the static port.

1/ How can I get this work on Windows?  I&#8217;ve tested with two options: [Cygwin][2] or [Putty][3].

<p style="padding-left: 30px">
  Cygwin: this is option is the most straight forward.  Install Cygwin.  On the Select Packages screen do a search for openssh and select it for install.
</p>

<p style="padding-left: 30px">
  The same ssh commands will work for creating tunnels under a Cygwin shell that you can use with Linux or Max OS X.
</p>

<p style="padding-left: 30px">
  Putty: The only way I&#8217;ve been successful with connecting JMX with putty was by using the <a href="http://the.earth.li/~sgtatham/putty/0.60/htmldoc/Chapter7.html#plink">plink</a> command line executable.  The graphical interface doesn&#8217;t allow you set the first hostname that is needed for the second tunnel.  plink gets this done.
</p>

<p style="padding-left: 30px">
  Let&#8217;s take a look at some real examples&#8230;
</p>

<p style="padding-left: 30px">
  For debug:
</p>

<p style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 60px">
  On the server:
</p>

<p style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 60px">
  Add the following to your <code style="padding: 0px;margin: 0px">alfresco.sh</code> file:
</p>

<p style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 60px">
  <code style="padding: 0px;margin: 0px">export JAVA_OPTS="${JAVA_OPTS} -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8082"</code>
</p>

<p style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 60px">
  The <code style="padding: 0px;margin: 0px">address</code> parameter sets the port you want the debugger to listen on.  Adding this line, will require you to restart Alfresco/tomcat for it to become active.
</p>

<p style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 60px">
  On the client:
</p>

<p style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 60px">
  Using standard port forwarding with ssh will enable Java debugging using plink
</p>

<code style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 60px">plink &lt;user&gt;@&lt;ip/host/dns address&gt; -L 2000:localhost:8082 -N</code>

<p style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 60px">
  The debugger is then set to connect to <code>localhost</code> using port <code>2000</code>
</p>

<p style="margin-top: 0px;margin-right: 0px;margin-bottom: 15px;margin-left: 0px;line-height: 18px;padding-top: 0px;padding-right: 0px;padding-bottom: 0px;padding-left: 30px">
  For JMX:
</p>

<p style="padding-left: 60px">
  On the server:
</p>

<p style="padding-left: 60px">
  Modify <code>alfresco.sh</code>:  set <code>-Dcom.sun.management.jmxremote</code> equal to <code>true</code>.  Append <code>-Djava.rmi.server.hostname=dummyhost</code> to the <code>JAVA_OPTS</code>.
</p>

<p style="padding-left: 60px">
  Add <code>monitor.rmi.services.port=50508</code> to <code>alfresco-global.properties</code>
</p>

<p style="padding-left: 60px">
  On the client:
</p>

<p style="padding-left: 60px">
  Add <code>127.0.0.1 dummyhost</code> to <code>C:\Windows\System32\drivers\etc\hosts</code>
</p>

<p style="padding-left: 60px">
  Using plink from the command line, run the following two commands:
</p>

<p style="padding-left: 60px">
  <code>plink &lt;user&gt;@&lt;ip/dns of server&gt; -L 2001:localhost:50500 -N</code>
</p>

<p style="padding-left: 60px">
  <code>plink &lt;user&gt;@&lt;ip/dns of server&gt; -L dummyhost:50508:localhost:50508 -N</code>
</p>

<p style="padding-left: 60px">
  Now you can connect to Alfresco using a jmx console, like jconsole (ships with jre/jdk).
</p>

<p style="padding-left: 60px">
  the connection string will look like this:
</p>

<p style="padding-left: 60px">
  <code>service:jmx:rmi:///jndi/rmi://localhost:2001/alfresco/jmxrmi</code>
</p>

<p style="padding-left: 60px">
  Unless you&#8217;ve changed them, the default username and password are <code>controlRole</code> and <code>change_asap</code>
</p>

 [1]: http://jared.ottleys.net/alfresco/tunneling-debug-and-jmx-for-alfresco
 [2]: http://cygwin.com/
 [3]: http://www.chiark.greenend.org.uk/~sgtatham/putty/