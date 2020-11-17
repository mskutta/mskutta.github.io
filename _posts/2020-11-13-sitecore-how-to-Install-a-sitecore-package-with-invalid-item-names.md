---
layout: post
title: "Sitecore How-To: Install a Sitecore Package with Invalid Item Names"
date:   2020-11-13 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Sometimes when installing a Sitecore package, the install will fail because there is an item name with an invalid character or the item name is too long. 
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sometimes when installing a Sitecore package, the install will fail because there is an item name with an invalid character or the item name is too long. Many of our sites have modified the default Sitecore settings for item name restrictions, which is why we sometimes see these errors from popular Sitecore packages (i.e. WFFM). You may see the following error:

``` text
An item name cannot contain any of the following characters: /:?"<>|[]- (controlled by the setting InvalidItemNameChars)
```

## Step-by-step guide

The following is a Sitecore config file that you can drop into your site to “reset” the item name restrictions. You will want to delete this config once you have successfully installed the package.

1. Copy ResetItemNameRestrictions.config into your Sitecore site.
    1. `/App_Config/Include/zzz/` for Sitecore 8
    1. `/App_Config/Environment/` for Sitecore 9+

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:set="http://www.sitecore.net/xmlconfig/set/">
  <sitecore>
    <settings>
      <setting name="InvalidItemNameChars" set:value="\/:?&quot;&lt;&gt;|[]" />
      <setting name="ItemNameValidation" set:value="^[\w\*\$][\w\s\-\$]*(\(\d{1,}\)){0,1}$" />
      <setting name="MaxItemNameLength" set:value="100" />
    </settings>
  </sitecore>
</configuration>
```