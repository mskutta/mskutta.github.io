---
layout: post
title: "Sitecore How-To: Generate a Link to a Sitecore Item"
date:   2018-10-02 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Quick Reference for generating links to Sitecore items.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

**Generate Link with Default Options**

```c#
string url = Sitecore.Links.LinkManager.GetItemUrl(item);
```

**Generate Link with Modified Options**

```c#
Sitecore.Links.UrlOptions urlOptions = Sitecore.Links.LinkManager.GetDefaultUrlOptions();
urlOptions.AlwaysIncludeServerUrl = true;
 
string url = Sitecore.Links.LinkManager.GetItemUrl(item, urlOptions);
```
