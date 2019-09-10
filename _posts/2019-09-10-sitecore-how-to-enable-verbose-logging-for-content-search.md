---
layout: post
title: "Sitecore How-To: Enable Verbose Logging for Sitecore Content Search"
date:   2019-09-10 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: When debugging index crawling and searching, it may be helpful to enable verbose logging.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

When debugging index crawling and searching, it may be helpful to enable verbose logging.

> Enabling this will generate a lot of logging. I would not suggest enabling this if you have a lot of content to index. If you are debugging index rebuilds, I suggest limiting the amount of content that is indexed.

## Step-by-step guide

1. Drop the following file into the Website\App_Config\Include\zzz folder:
    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:set="http://www.sitecore.net/xmlconfig/set/" >
      <sitecore>
        <settings>
          <!--  VERBOSE LOGGING
                This feature is designed to facilitate search index configuration and provide the necessary insight
                in troubleshooting scenarios. For example, if a particular item is not getting indexed,
                the VerboseLogger can provide more context and help you to figure out the problem.

                It is important that you only enable the VerboseLogger component in special circumstances
                and never run it for long periods in a production environment.
                Otherwise, this would result in an extremely large log file, which may have performance implications.
          -->
          <setting name="ContentSearch.VerboseLogging" set:value="true" />
        </settings>
        <log4net>
          <logger name="Sitecore.Diagnostics.Search">
            <!-- <level set:value="DEBUG"/> -->
          </logger>
          <logger name="Sitecore.Diagnostics.Crawling">
            <!-- <level set:value="DEBUG"/> -->
          </logger>
      </log4net>
      </sitecore>
    </configuration>
    ```
1. Uncomment the log4net changes depending on what you are debugging.
1. View the Crawling.log.* and Search.log.* files.