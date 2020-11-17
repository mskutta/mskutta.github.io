---
layout: post
title: "Sitecore How-To: Debug an IIS site with Visual Studio"
date:   2020-11-11 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: David Guan and Mike Skutta
excerpt: After you set up a site, you might want to debug the process to figure out variable information and everything else. Here’s how you’d go about setting up the debugging process.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

After you set up a site, you might want to debug the process to figure out variable information and everything else. Here’s how you’d go about setting up the debugging process.

## Step-by-step guide

1. Open up your visual studio process as an administrator
1. Click on `Debug` → `Attach to process`
1. Make sure the `Show processes from all users` checkbox is checked and search for the `w3wp.exe` process.
    ![Code](/images/sitecore-how-to-debug-an-iis-site-with-visual-studio/attach.png)
1. If there are multiple processes, you could attach to all of them by holding down Ctrl and clicking on the process. Alternatively, you could run this to find out what Application pool the `w3wp.exe` points to. [How to identify which w3wp.exe belongs to which Application Pool](/2020/11/09/sitecore-how-to-identify-which-w3wpexe-belongs-to-which-application-pool/) 
1. Set a breakpoint wherever you want the process to stop. For example: 
1. Get that portion of code to rerun, either through a page reload or a button click or however the code generates. Most code runs on page load so refreshing the page should work but for forms you can test through a button click instead.
    ![Code](/images/sitecore-how-to-debug-an-iis-site-with-visual-studio/code.png)
1. There are a few buttons here that are useful for debugging. This is defined clearly in the Microsoft documentation: https://docs.microsoft.com/en-us/visualstudio/debugger/debugger-feature-tour?view=vs-2019
1. The most useful one I’ve found is the step over function, or F10, that goes to the next line. 