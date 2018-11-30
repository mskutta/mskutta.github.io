---
layout: post
title: "Sitecore Troubleshooting: Sitecore Hangs when Creating a Language"
date:   2018-11-30 00:00:00 -0500
categories: sitecore
tags: sitecore-troubleshooting
author: Kyle Mattimore and Mike Skutta
excerpt: After trying to create a language, the interface hangs as if it's preparing a confirmation window.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore troubleshooting articles. These articles are meant to be quick guides to troubleshoot various issues within Sitecore. The troubleshooting articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## Problem

After trying to create a language, the interface hangs as if it's preparing a confirmation window.

The language item you are trying to create may contain characters blacklisted in the config. To confirm this, open the network inspector and look at the last few responses (you may have to try again with dev tools open) and see if any of them complain about the item name. 

## Solution

Change (temporarily or permanently) the InvalidItemNameChars setting in the config. In most cases it will be the '-' character due to names like ja-JP. 