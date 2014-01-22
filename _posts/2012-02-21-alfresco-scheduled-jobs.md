---
title: 'Alfresco: Scheduled Jobs'
author: jared
layout: post
permalink: /alfresco/alfresco-scheduled-jobs
aktt_notify_twitter:
  - yes
aktt_tweeted:
  - 1
categories:
  - Alfresco
---
**Note:** *@tommoz has pointed out to me a blog post that I&#8217;d never seen before from @Ixxus which is a [complete example of writing a scheduled job][1] (spring beans and all). You should really check out their excellent example if you need a full example of writing scheduled jobs for Alfresco.*

One of the things that is easy to forget when adding/writing a new scheduled jobs is to wrap your code in a RetryingTransactionHelper \_and\_ a RunAs. When executing the scheduled job we need to make sure that we provide a transaction in which to perform your custom code and a user to perform the code under.

Example:

<pre class="brush: java; title: ; notranslate" title="">public void execute()
    {
        AuthenticationUtil.runAs(new AuthenticationUtil.RunAsWork&lt;Object&gt;()
        {
            @Override
            public Object doWork()
                throws Exception
            {
                RetryingTransactionCallback&lt;;Object&gt;; txnWork = new RetryingTransactionCallback&lt;Object&gt;()
                {
                    public Object execute()
                        throws Exception
                    {
                        //Add logic here
                        return null;
                    }
                };

                transactionService.getRetryingTransactionHelper().doInTransaction(txnWork, false);
                return null;
            }
        }, user_authority);

    }
</pre>

Hopefully this template will prove usefully to anyone writing a custom scheduled actions.

 [1]: http://t.co/RAOFy23l