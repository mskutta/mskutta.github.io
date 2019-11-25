---
layout: post
title: "Sitecore How-To: How to Debug on a Remote Machine"
date:   2019-11-25 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: David Guan and Mike Skutta
excerpt: Sometimes you set up a Sitecore site on a remote shared machine and end up needing to debug the instance on visual studio. Instead of having to set up the site again, simply set up remote debugging on the remote machine instead.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sometimes you set up a Sitecore site on a remote shared machine and end up needing to debug the instance on visual studio. Instead of having to set up the site again, simply set up remote debugging on the remote machine instead. 

## Step-by-step guide

1. Navigate to https://docs.microsoft.com/en-us/visualstudio/debugger/remote-debugging?view=vs-2019 on the remote machine and download the version of remote tools compatible with your version of Visual Studio.
1. Install said remote tools and set it up like on the site:
    ![Remote Debugging Configuration](/images/sitecore-how-to-debug-on-a-remote-machine/remote-debugging-configuration.png "Remote Debugging Configuration")
1. Run the remote debugger **as administrator** and then head to your visual studio instance.
1. Go to Tools → Attach to process and click Find... at the **Connection target**.
1. You might get a prompt from Windows Firewall to allow connections, make sure to allow it to find your remote device.
    ![Remote Debugging Configuration](/images/sitecore-how-to-debug-on-a-remote-machine/attach-to-process.png "Remote Debugging Configuration")
1. Once attached, you should be able to look for w3wp.exe as normal and start debugging.