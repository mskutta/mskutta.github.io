---
layout: post
title: "Loggly Optimization for IIS logs"
date:   2016-07-19 00:00:00 -0500
categories: sitecore
tags: sitecore
author: Rick Tham, Mike Skutta
target: https://www.onenorth.com/blog/post/loggly-optimization-for-iis-logs-1
excerpt: For two years now, we have been using Loggly at One North, and it has proven to be a valuable DevOps tool for our organization.  This service allows us to view and search our logs quickly across environments and applications such as Sitecore and other web CMS applications.
---

For two years now, we have been using [Loggly](https://www.loggly.com/docs/about-loggly/) at One North, and it has proven to be a valuable DevOps tool for our organization. This service allows us to view and search our logs quickly across environments and applications such as [Sitecore](http://www.sitecore.net/partners/find-partner/o/one-north-interactive.aspx) and other web CMS applications.

Loggly is a SaaS provider that charges by the amount of data that is being sent to them for processing. Over time, our cost naturally started to increase as we rolled out Loggly to more servers and applications that we managed. The bulk of our data consumption to Loggly was our web server logs.&nbsp;

After having an optimization session with a Loggly technical support engineer, we identified that the web traffic logs were being duplicated to their service: once as JSON and then again as unparsed key value pair data. Up until this point, we ignored the unparsed message and assumed that it was needed for Loggly.

![Loggly JSON](/images/loggly-optimization-for-iis-logs/loggly_optimization_image.png)

When we first started using Loggly, we leveraged [their source setup examples](https://www.loggly.com/docs/iis-web-server-logs/) online to configure our systems to send logs to their service. The recommended approach is to use NXLOG, an open source universal log collector, to send IIS web traffic logs. [NXLOG](http://nxlog-ce.sourceforge.net/about) is a lightweight and efficient service that sends the logs to Loggly.

After digging into this unparsed data, it was determined that it was not needed by Loggly and that the NXLOG default behavior was sending our data over in two formats.Luckily, we discovered a relatively straightforward fix.

The following are our modifications to the nxlog.config file to remove the unparsed data from the logs as they are sent to Loggly. The commented example below shows you how to do it.

*nxlog.config*
``` xml
#The following HOSTNAMELENGTH is to grab the variable length within the beginning of the message.
define HOSTNAMELENGTH size(hostname())
<Output out>
    Module om_tcp
    Host logs-01.loggly.com
    Port 514
    # The below statement was the default code from loggly
    #Exec to_syslog_ietf(); $raw_event = replace($raw_event, 'NXLOG@12345', '00000000-0000-0000-0000-000000000000@12345 tag="windows"] [', 1);
    # Building the initial message + constructing the header for Loggly + IIS JSON message
    Exec to_syslog_ietf(); $raw_event = substr($raw_event, 0, 48 + %HOSTNAMELENGTH%) + "[00000000-0000-0000-0000-000000000000@12345 tag="windows"] " + $Message;
</Output>
```

Once you make this change to NXLOG, you will no longer see the unparsed data within your server logs in Loggly.

Hopefully, you not only find this post helpful, but end up saving a few bucks, too.