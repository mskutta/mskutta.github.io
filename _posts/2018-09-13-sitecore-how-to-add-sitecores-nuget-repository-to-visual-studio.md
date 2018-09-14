---
layout: post
title: "Sitecore How-To: Add Sitecore's NuGet Repository To Visual Studio"
date:   2018-09-13 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Sitecore has their own NuGet repository at https://sitecore.myget.org/gallery/sc-packages. To restore/add Sitecore packages in Visual Studio, you need to add this repository to package sources.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sitecore has its own NuGet repository at https://sitecore.myget.org/gallery/sc-packages. To restore/add Sitecore packages in Visual Studio, you need to add this repository to package sources.

## Step-by-step guide

In Visual Studio

1. Go to Tools → Options
1. Navigate to **NuGet Package Manager → Package Sources** in the navigation tree on the left.
1. Click the plus button in the top right.
1. At the bottom of the dialog, you can set the name and source of the newly added package source.
    1. Set **Name** to whatever you want. Say **Sitecore NuGet**
    1. Set **Source** to **https://sitecore.myget.org/F/sc-packages/api/v3/index.json**

![NuGet Package Manager Package Sources](/images/how-to-add-sitecores-nuget-repository-to-visual-studio/image2018-5-8_11-18-30.png)

## Update 2018-09-14

As Corey Smith @sitecorey pointed out, you can also add a **NuGet.config** to your repo.  Using this approach, the Sitecore feed is available even if you have not configured it in VS.  Here is a good example: https://github.com/Sitecore/Habitat/blob/master/nuget.config.