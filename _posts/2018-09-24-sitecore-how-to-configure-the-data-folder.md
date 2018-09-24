---
layout: post
title: "Sitecore How-To: Configure the Sitecore Data Folder"
date:   2018-09-24 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Depending on the version of Sitecore, when setting up a new Sitecore instance, you need to configure where the data folder is located and do it in a way so it is not overwritten every time you do a build.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Depending on the version of Sitecore, when setting up a new Sitecore instance, you need to configure where the data folder is located and do it in a way so it is not overwritten every time you do a build.

> This should only be needed when the dataFolder variable is specified in web.config

## Step-by-step guide

1. Navigate to App_Config\Include and open the DataFolder.config.example file (this file is provided by the Sitecore installer. You will not find this is source control.)
1. Using **backslashes**, set the path to the data folder. Save and close.
1. Rename the file from DataFolder.config.example to DataFolder.config. (In some cases, you may need to prepend "z" to make sure it is not overwritten)

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <sc.variable name="dataFolder">
      <patch:attribute name="value">C:\Projects\My Project\Data</patch:attribute>
    </sc.variable>
  </sitecore>
</configuration>
```