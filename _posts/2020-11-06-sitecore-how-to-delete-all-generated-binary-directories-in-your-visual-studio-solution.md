---
layout: post
title: "Sitecore How-To: Delete All Generated Binary Directories in Your Visual Studio Solution"
date:   2020-11-06 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Sometimes running Clean Solution in Visual Studio does not delete everything in the bin and obj directories. This can sometimes lead to reference issues if you have old binaries lingering around.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sometimes running Clean Solution in Visual Studio does not delete everything in the `bin` and `obj` directories. This can sometimes lead to reference issues if you have old binaries lingering around.

## Step-by-step guide

Run this PowerShell script from the root of your solution. It will delete all `bin` and `obj` directories, ignoring the NuGet `packages` directory:

``` bash
Get-ChildItem .\ -Include bin,obj -Recurse | Where-Object {$_.PSParentPath -NotLike "*packages*"} | ForEach-Object -Process { Remove-Item $_.fullname -Force -Recurse }
```