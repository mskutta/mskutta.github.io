---
layout: post
title: "Sitecore How-To: Export Media from Sitecore 6 with Powershell"
date:   2018-09-28 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Ben DeLeon and Mike Skutta
excerpt: You may have a need to export media.  For example; if you are rebuilding a Sitecore site you may need to export existing media to then import it into the new site. Here is how you export media in Sitecore 6.6, but may be applicable to other versions as well.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

You may have a need to export media.  For example; if you are rebuilding a Sitecore site you may need to export existing media to then import it into the new site. Here is how you export media in Sitecore 6.6, but may be applicable to other versions as well.

## Step-by-step guide

1. Install “Sitecore PowerShell Extensions-2.8.zip” https://marketplace.sitecore.net/Modules/Sitecore_PowerShell_console.aspx
1. After install, when prompted restart your Sitecore Instance
1. Open Content Editor and navigate to Media Library in Content Editor Tree
1. Right Click → Scripts → Zip and Download and download will begin
1. Wait for it to finish, and you'll be able to find your zipped file in the Data Directory of the Application 

> Note- Spaces will be replace with "%20" in exported files so you may need a way to rename files to replace with spaces. Ideally with a Powershell script.