---
layout: post
title: "Sitecore How-To: Automatically Copy Files from Project Source to Sitecore Instance"
date:   2019-08-30 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: George West, Kyle Mattimore, and Mike Skutta
excerpt: Team Development for Sitecore can automatically watch the files in your project and copy them to your Sitecore instance when modifications are made. This setting is disabled by default, but can easily be turned on via Visual Studio Options.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Team Development for Sitecore can automatically watch the files in your project and copy them to your Sitecore instance when modifications are made. This setting is disabled by default, but can easily be turned on via Visual Studio Options. 

## Step-by-step guide

1. Open Visual Studio
1. Go to **Tools > Options > TDS Options > General**
1. Set **Content File Sync** to True
1. Restart Visual Studio

![TDS Options](/images/sitecore-how-to-automatically-copy-files-from-project-source-to-sitecore-instance/tds-options.png "TDS Options")

Because TDS already knows where your instance is, no extra configuration is required. TDS will copy any files in your solution with a “Content” Build Action into your Sitecore Deploy Folder configured in the TDS Build window on the properties of your TDS Classic project. This works even if files are modified externally—with Git or Gulp, for example.

![Visual Studio](/images/sitecore-how-to-automatically-copy-files-from-project-source-to-sitecore-instance/visual-studio.png "Visual Studio")

> **Using with QA Build Configuration**
> Because the copy destinations are configured in the TDS Build options, changing your build environment will also change the copy destination!
>
> ![Configuration](/images/sitecore-how-to-automatically-copy-files-from-project-source-to-sitecore-instance/configuration.png "Configuration")

## Nuances

* If you update your web.config, or any other file with transforms applied in the build process, it will copy it over and potentially remove any transforms in web.debug.config. Rebuild after editing web.config to avoid potential problems.

## Troubleshooting

*Content File Sync* logs to the Visual Studio output window for Team Development for Sitecore. Every time a content file is synced, it will be logged here.

*Content File Sync* doesn't properly watch files on Visual Studio startup (VS 15.9.x)

Sometimes the "autocopy" process doesn't start with Visual Studio. To fix this, simply toggle **Tools > Options > TDS Options > General > Content File Sync**, which should manually stop/start the process. When the process is started or stopped, it will show a log in the output window. This fix works with TDS 5.8.