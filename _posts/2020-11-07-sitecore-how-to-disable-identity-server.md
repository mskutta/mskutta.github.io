---
layout: post
title: "Sitecore How-To: Disable Identity Server"
date:   2020-11-07 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Sitecore Identity Server is a new feature that started in Sitecore 9.1 and it is a separate identity provider. This feature is typically enabled by default, so if you are not using it you should disable it to prevent unwanted errors and login buttons.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

 Sitecore [Identity Server](https://doc.sitecore.com/developers/91/sitecore-experience-manager/en/sitecore-identity.html) is a new feature that started in Sitecore 9.1 and it is a separate identity provider. This feature is typically enabled by default, so if you are not using it you should disable it to prevent unwanted errors and login buttons.

## Step-by-step guide

Sitecore provides the config to disable this in `\App_Config\Include\Examples`

1. Copy the file `Sitecore.Owin.Authentication.IdentityServer.Disabler.config.example` from `\App_Config\Include\Examples\` into `\App_Config\Environment\`
1. Rename the file to `.config`

You can take it a step further and disable Owin Authentication too:

1. Copy the file `Sitecore.Owin.Authentication.Disabler.config.example` from `\App_Config\Include\Examples\` into `\App_Config\Environment\`
1. Rename the file to `.config`