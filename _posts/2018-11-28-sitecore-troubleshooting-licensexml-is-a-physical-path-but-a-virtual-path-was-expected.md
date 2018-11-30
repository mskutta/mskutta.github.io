---
layout: post
title: "Sitecore Troubleshooting: license.xml is a physical path, but a virtual path was expected"
date:   2018-11-28 00:00:00 -0500
categories: sitecore
tags: sitecore-troubleshooting
author: Erik Carron and Mike Skutta
excerpt: license.xml is a physical path but a virtual path was expected.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore troubleshooting articles. These articles are meant to be quick guides to troubleshoot various issues within Sitecore. The troubleshooting articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## Problem

You received the following error:

```
Server Error in '/' Application.

'.../Data/license.xml' is a physical path, but a virtual path was expected.

Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code.

Exception Details: System.Web.HttpException: '.../Data/license.xml' is a physical path, but a virtual path was expected.

Source Error:

An unhandled exception was generated during the execution of the current web request. Information regarding the origin and location of the exception can be identified using the exception stack trace below.
```

![Exception](/images/sitecore-troubleshooting-licensexml-is-a-physical-path-but-a-virtual-path-was-expected/image2017-5-18_14-36-48.png)



## Solution

Forward slashes were used instead of backslashes when specifying the path to the data folder.

Correct the value specified in **Website\App_Config\Include\DataFolder.config** (this may vary)

Correct: ```<patch:attribute name="value">C:\...\Data</patch:attribute>``` (use backslashes)