---
layout: post
title: "Sitecore How-To: Dynamically Get the Home Item for the Current Site"
date:   2018-09-26 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: In some cases you may need to get the home item of the current site. Data from the home item may be required based on the site context. When you need to dynamically get the home item of the current site, perform the following.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

In some cases you may need to get the home item of the current site. Data from the home item may be required based on the site context. When you need to dynamically get the home item of the current site, perform the following:

```c#
Sitecore.Context.Database.GetItem(Sitecore.Context.Site.StartPath)
```