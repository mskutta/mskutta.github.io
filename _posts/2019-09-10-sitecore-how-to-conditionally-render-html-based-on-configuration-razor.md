---
layout: post
title: "Sitecore How-To: Conditionally Render HTML Based on Configuration (Razor) Site"
date:   2019-09-10 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: George West and Mike Skutta
excerpt: This article explains how to change your output markup of your Sitecore site according to its configuration.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

This article explains how to change your output markup of your Sitecore site according to its configuration.

Sometimes, you may want to render something differently in development versus production.

A good example of this is scripts loaded via CDN. [Vue, for example](https://vuejs.org/v2/guide/deployment.html#Turn-on-Production-Mode):

> **Turn on Production Mode**
>
> During development, Vue provides a lot of warnings to help you with common errors and pitfalls. However, these warning strings become useless in production and bloat your app’s payload size. In addition, some of these warning checks have small runtime costs that can be avoided in production mode.

## Step-by-step guide

To do this, we can define and use an HtmlHelper extension method which checks the Sitecore configuration for a feature-specific (in this case, HTML rendering) value.

**z.OneNorth.Html.config**
```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <settings>
      <setting name="OneNorth.HtmlEnvironment" value="dev"/>
    </settings>
  </sitecore>
</configuration>
```

**HtmlHelperExtensions.cs**
```csharp
using System.Web.Mvc;
 
public static class HtmlHelperExtensions
{
    public static bool IsDebug(this HtmlHelper htmlHelper)
    {
        return Sitecore.Configuration.Settings.GetSetting("OneNorth.HtmlEnvironment") == "dev";
    }
}
```

**DefaultLayout.cshtml**
```cshtml
@if (Html.IsDebug())
{
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
} else
{
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
}
```
