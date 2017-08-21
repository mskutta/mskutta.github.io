---
layout: post
title: "How to Quickly Determine License Information in Sitecore 8"
date:   2016-06-23 00:00:00 -0500
categories: sitecore
tags: sitecore license
author: Mike Skutta
image:
    url: /images/how-to-quickly-determine-license-information-in-sitecore-8/login.png
    height: 729
    width: 1157
excerpt: The Sitecore 8 login screen has a new look and feel, and by default, no longer contains the license information. 
---

## Overview

The Sitecore 8 login screen has a new look and feel, and by default, no longer contains the license information.

![Sitecore Login](/images/how-to-quickly-determine-license-information-in-sitecore-8/login.png)

Sitecore does provide a way to enable the display of the license information by modifying the following setting:

<setting name=”Login.DisableLicenseInfo” value=”true” /> (Please see [Sitecore 8, help where’s my license information?](http://www.newguid.net/sitecore/2014/sitecore-8-help-wheres-license-information/) for further information on this approach.) 

This requires access to the server in order to change the setting, which is not always possible. However, there is actually a quicker way to determine the license information.

## Quick Way

The license information is present in the HTML source of the page. By default, the display of this information is set to none.

To view the license info, simply view the source of the page. In most browsers, you can view the source of the page by right clicking and selecting View Page Source. In the source, do a search for “license.” You will find the following:

![Source](/images/how-to-quickly-determine-license-information-in-sitecore-8/source.png)