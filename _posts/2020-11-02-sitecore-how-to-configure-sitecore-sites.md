---
layout: post
title: "Sitecore How-To: Configure Sitecore Sites"
date:   2020-11-02 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Whether you are modifying the default website or setting up multiple sites, here is the best way to accomplish that.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Whether you are modifying the default website or setting up multiple sites, here is the best way to accomplish that.

## Step-by-step guide

For a single-site instance, only modify the attributes you need to for **website**.

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:set="http://www.sitecore.net/xmlconfig/set/">
  <sitecore>
    <sites>
      <site name="website" set:enableTracking="false" set:hostName="" set:targetHostName="www.mysite.com" set:scheme="https" set:rootPath="/sitecore/content/Sites/Main" set:startItem="/home" set:mvcArea="Main" />
    </sites>
  </sitecore>
</configuration>
```

For a multi-site instance, only modify the attributes you need to for **website**, add your other sites before **website**, inherit from **website**, then only add the attributes you need to modify.

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:set="http://www.sitecore.net/xmlconfig/set/">
  <sitecore>
    <sites>
      <site patch:after="site[@name='modules_website']" name="myothersite1" inherits="website" hostName="www.myothersite1.com" targetHostName="www.myothersite1.com" rootPath="/sitecore/content/Sites/Other Site 1" startItem="/home" mvcArea="OtherSite" />
      <site patch:after="site[@name='modules_website']" name="myothersite2" inherits="website" hostName="www.myothersite2.com" targetHostName="www.myothersite2.com" rootPath="/sitecore/content/Sites/Other Site 2" startItem="/home" mvcArea="OtherSite" />
      <site name="website" set:enableTracking="false" set:hostName="" set:targetHostName="www.mysite.com" set:scheme="https" set:rootPath="/sitecore/content/Sites/Main" set:startItem="/home" set:mvcArea="Main" />
    </sites>
  </sitecore>
</configuration>
```

> Note: Do not use **patch:instead**. It increases the chances of messing up the config and does not work well when upgrading Sitecore, as site attributes are added/changed over time.