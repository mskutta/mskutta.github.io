---
layout: post
title: "Sitecore How-To: Disable Analytics (xDB)"
date:   2018-09-25 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Sitecore Analytics/Experience Database (xDB) is something you may not need locally or on QA, so disabling it may increase Sitecore's performance.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sitecore Analytics/Experience Database (xDB) is something you may not need locally or on QA, so disabling it may increase Sitecore's performance.  One reason you may want to disable xDB locally is if you just need to fix front end look and feel issues, xDB is not required in this case.  xDB uses MongoDB, so if you do not have MongoDB setup locally and want to avoid a bunch of errors in the Sitecore log, disable xDB.

## Step-by-step guide

1. Add DisableAnalytics.config to your Sitecore instance (the following is a good location for custom configs: Website\App_Config\Include\zzz)

    ```xml
    <configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:set="http://www.sitecore.net/xmlconfig/set/">
      <sitecore>
        <settings>
          <setting name="Xdb.Enabled" set:value="false"/>
          <setting name="Xdb.Tracking.Enabled" set:value="false"/>
        </settings>
      </sitecore>
    </configuration>
    ```