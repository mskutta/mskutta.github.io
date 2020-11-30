---
layout: post
title: "Sitecore How-To: Edit Your Host File"
date:   2020-11-03 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Mike Skutta
excerpt: When running sites locally, you will want to use your own host names. The host file on Windows allows you to create these host names.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

When running sites locally, you will want to use your own host names. The host file on Windows allows you to create these host names.

## Step-by-step guide

1. Open a text editor in administration mode.
1. Open file C:\Windows\System32\drivers\etc\hosts
1. Add a new line at the end of the file to define your new host name. See comments at the top of the file for examples of how to use this file.
1. Save and close.

> Note: IP address 127.0.0.1 is your local machine, so if setting up a site for local use, you should use that IP.