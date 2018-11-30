---
layout: post
title: "Sitecore Troubleshooting: Attach Media Application Access Denied Error"
date:   2018-11-26 00:00:00 -0500
categories: sitecore
tags: sitecore-troubleshooting
author: Erik Carron and Mike Skutta
excerpt: When a non-admin tries to update an existing media item by clicking the Attach link and they get a popup with an Application access denied error.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore troubleshooting articles. These articles are meant to be quick guides to troubleshoot various issues within Sitecore. The troubleshooting articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## Problem

When a non-admin tries to update an existing media item by clicking the Attach link and they get a popup with an Application access denied error:

```text
Application access denied.
Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code.

Exception Details: Sitecore.Exceptions.AccessDeniedException: Application access denied.
```

![Exception](/images/sitecore-troubleshooting-attach-media-application-access-denied-error/image2018-8-17_9-22-40.png)

Starting in Sitecore 8.2, the default Sitecore security revoked Inheritance for the Everyone role for the /sitecore/content/Applications/Dialogs/Upload item. This prevents users from using the Attach dialog.

![Access Rights](/images/sitecore-troubleshooting-attach-media-application-access-denied-error/image2018-8-17_9-27-22.png)

## Solution

Grant Read access to specific roles that need access to attach media items, such as Sitecore\Client Content Editor. 

![Access Rights](/images/sitecore-troubleshooting-attach-media-application-access-denied-error/image2018-8-17_9-33-26.png)