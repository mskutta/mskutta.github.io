---
layout: post
title: "Sitecore How-To: Generate a Link to a Sitecore Media Item"
date:   2018-10-03 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Quick Reference for generating links to Sitecore media items.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

**Generate Link with Default Option**

```c#
string url = Sitecore.Resources.Media.MediaManager.GetMediaUrl(mediaItem);
```

**Generate Safe Link (Sitecore 8+)**

https://doc.sitecore.net/sitecore_experience_platform/setting_up_and_maintaining/security_hardening/configuring/protect_media_request
```c#
string unsafeUrl = Sitecore.Resources.Media.MediaManager.GetMediaUrl(mediaItem);
string safeUrl = Sitecore.Resources.Media.HashingUtils.ProtectAssetUrl(unsafeUrl);
```

**Generate Link with Modified Options**

```c#
Sitecore.Resources.Media.MediaUrlOptions mediaUrlOptions = new Sitecore.Resources.Media.MediaUrlOptions { AbsolutePath = true };
string url = Sitecore.Resources.Media.MediaManager.GetMediaUrl(item, mediaUrlOptions);
```

**Generate Link within Content with Modified Options**
Sometimes you may want to deviate from the default media url options for media items within content (i.e. absolute urls for images for PDF generation). In this case you need to use ```Sitecore.Configuration.SettingsSwitcher```. 

> Not confirmed, but this setting changes should be scoped to the request context.

```c#
Sitecore.Configuration.SettingsSwitcher mediaSettingsSwitcher = new Sitecore.Configuration.SettingsSwitcher("Media.AlwaysIncludeServerUrl", "true");
```