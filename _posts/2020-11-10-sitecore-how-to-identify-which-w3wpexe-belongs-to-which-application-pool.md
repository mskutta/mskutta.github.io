---
layout: post
title: "Sitecore How-To: Identify which w3wp.exe Belongs to which Application Pool"
date:   2020-11-10 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: George West and Mike Skutta
excerpt: When debugging with Visual Studio, you need to attach to a w3wp process. If you have multiple Application Pools running, however, you will get multiple instances of w3wp.exe running, and it isn't immediately obvious which one you should attach to to debug a given website. There are steps you can follow to find out which process belongs to your website's Application Pool.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

When debugging with Visual Studio, you need to attach to a w3wp process. If you have multiple Application Pools running, however, you will get multiple instances of w3wp.exe running, and it isn't immediately obvious which one you should attach to to debug a given website. There are steps you can follow to find out which process belongs to your website's Application Pool.

## Step-by-step guide

### With Task Manager

1. Open **Task Manage**r
   1. Shortcut `Ctrl + Shift + Esc`
1. Show the **Command Line** and **PID** columns by right-clicking the header row and selecting the appropriate item from the multi-list.
    ![Command Prompt](/images/sitecore-how-to-identify-which-w3wpexe-belongs-to-which-application-pool/task-manager.png)
1. Sort by **Name** to find processes named "IIS Worker Process"
1. Find the IIS Worker Process that has your website's name in the **Command Line** column and take note of the **PID**.
    ![Command Prompt](/images/sitecore-how-to-identify-which-w3wpexe-belongs-to-which-application-pool/task-manager-2.png)

### With the command line

1. Run **cmd.exe** as Administrator
1. Run the following command: `C:\Windows\System32\inetsrv\appcmd.exe list wp`
1. Find your website's Application Pool in the output and take note of the corresponding PID, which is the w3wp.exe process you are looking for.
    ![Command Prompt](/images/sitecore-how-to-identify-which-w3wpexe-belongs-to-which-application-pool/command-prompt.png)