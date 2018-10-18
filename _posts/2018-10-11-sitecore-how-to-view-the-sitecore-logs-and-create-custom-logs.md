---
layout: post
title: "Sitecore How-To: View the Sitecore Logs and Create Custom Logs"
date:   2018-10-11 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: When you want to view the Sitecore logs or create custom logs.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

When you want to view the Sitecore logs or create custom logs.

> New log files are created every time Sitecore cycles. This is done with a log4net file appender. 

## Step-by-step guide

Navigate to the\Data\logs folder. The Data folder should be in the same location as the Website folder.

* log.yyyymmdd.txt
    * This is the main Sitecore log
    * It will contain error details
* Crawling.log.yyyymmdd.txt
    * The indexing log
    * Check this if rebuilding an index is taking too long or failing
* Search.log.yyyymmdd.txt
    * The Lucene log
    * See what Lucene queries were ran
* Publishing.log.yyyymmdd.txt
    * Publishing log
* Client.log.yyyymmdd.txt
    * Starting with Sitecore 8.1 support has been added to log client side events. You can now track javascript errors in your application. All configuration for client logging is contained within Sitecore.JSNLog.config within App_Data/Include. Here's more information and instructions for setting it up.
* Fxm.log.yyyymmdd.txt
    * Log file for the Federated Experience Manager - a sitecore module using xDB
* WebDAV.log.yyyymmdd.hhmmss.txt
    * Logs WebDAV activity. It is usually disabled for us, so logs would be empty (https://doc.sitecore.net/sitecore_experience_platform/setting_up_and_maintaining/security_hardening/configuring/disable_webdav)

### Adding Custom Log Files

There are instances where you want to capture specific events in a separate log file to help with debugging or just to not overload main log. For example, if you have any complex integrations. Setup is very straight-forward

1. Add new custom config file with additional log4net section that includes information about your appender. In example below it's IntegrationLogFileAppender that would create a file Integration.log.yyyymmdd.txt in logs folder

    ```xml
    <configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
      <sitecore>
        <log4net>
          <appender name="IntegrationLogFileAppender" type="log4net.Appender.SitecoreLogFileAppender, Sitecore.Logging">
            <file value="$(dataFolder)/logs/Integration.log.{date}.txt"/>
            <appendToFile value="true"/>
            <layout type="log4net.Layout.PatternLayout">
              <conversionPattern value="%4t %d{ABSOLUTE} %-5p %m%n"/>
            </layout>
          </appender>
          <logger name="Sitecore.Diagnostics.Integration" additivity="false">
            <level value="INFO"/>
            <appender-ref ref="IntegrationLogFileAppender"/>
            <appender-ref ref="LogglyAppender"/>
          </logger>
        </log4net>
      </sitecore>
    </configuration>
    ```
1. Create a custom logger class based on the configuration above:

    ```c#
    using log4net;
 
    namespace Website.Logic.Logging
    {
        public static class IntegrationLogger
        {
            private static ILog log;
            public static ILog Log
            {
                get
                {
                    return log ?? (log = log4net.LogManager.GetLogger("Sitecore.Diagnostics.Integration"));
                }
            }
        }
    }
    ```
1. The below code shows implementation where the sitecore config is used, not web.config (which is what the above sample is assuming). Review 'Gotchas' to make sure any stand alone projects are set up correctly.

    ```c#
    //this namespace will come from the Sitecore.Logging dll, no need to include a log4net dll reference
    using log4net;
    
    namespace Website.Logic.Logging
    {
        public static class IntegrationLogger
        {
            private static ILog log;
            public static ILog Log
            {
                get
                {
                    return log ?? (log = Sitecore.Diagnostics.LoggerFactory.GetLogger("Sitecore.Diagnostics.Integration"));
                }
            }
        }
    }
    ```

1. Sample usage:

    ```c#
    IntegrationLogger.Log.Info("Integration started.");
    ```

### Methods to view log files

**Sitecore Instance Manager (SIM) Log Viewer**

Select the project in SIM and view the specific log file. 

**Sitecore Log Analyzer (SCLA)**

Download as a separate executable or also available under 'Entire Log files' in SIM. 

**Tail**

Stream changes from a specific file - execute it like `tail -F C:/projects/client/Data/logs/log.20170712.165114.txt`. Git bash and other bash-like shells for windows include tail. Note that -F is used instead of the traditional -f because it seems to behave better on windows. 

**Multitail**

Available on bash like cygwin (and maybe linux for windows subsystem?). watch multiple files (including newly created ones) with multitail -Q 1 "c:/projects/client/Data/logs/*.txt"

**Baretail**

Real-time log viewer available for free download here: https://www.baremetalsoft.com/baretail/

### Gotchas

If you are implementing a custom log in its own visual studio project, make sure to include references for Sitecore.Kernel, and Sitecore.Logging. Sitecore.Logging includes Log4Net implementation, so no need to add a log4net dlls to your standalone project. 