---
layout: post
title: "Sitecore How-To: Setup Sitecore 9.1 Locally With Sitecore Instance Manager"
date:   2019-09-24 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Sitecore 9.1 is recommended to be installed with SIF (Sitecore Installation Framework), which is a PowerShell module, but if you would like to install it with SIM (Sitecore Instance Manager) then follow these steps.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sitecore 9.1 is recommended to be installed with SIF (Sitecore Installation Framework), which is a PowerShell module, but if you would like to install it with SIM (Sitecore Instance Manager) then follow these steps. 

## Step-by-step guide

1. You need to have .NET Framework 4.7.1 installed on your machine. For development in Visual Studio, you will also need the .NET Framework 4.7.1 Developer Pack: https://dotnet.microsoft.com/download/thank-you/net471-developer-pack
1. Open SIM and click "Get Sitecore". At the time of creating this page, Sitecore 9.1.0 is available for download, but Sitecore 9.1.1 is not. Not sure if this will change in the future, but follow the following steps to manually get Sitecore 9.1.1:
    1. Log into https://dev.sitecore.net
    1. Go to https://dev.sitecore.net/Downloads/Sitecore_Experience_Platform/91/Sitecore_Experience_Platform_91_Update1.aspx
    1. Download the ZIP archive of the Sitecore site root folder resource. (https://dev.sitecore.net/~/media/8A91A7B3AA084A66A0FF34DA57BFD04C.ashx)
    1. Place the zip file into your SIM repository folder.
1. Using SIM, install a Sitecore 9.1 site.
1. By default, Sitecore 9.1 will use the new Sitecore Identity (SI) server, which requires setting up a SI server. If you would like to use the old authentication method, follow these steps:
    1. Update the security connection string in Website\App_Config\ConnectionStrings.config so it uses the Core database (copy the core connection string to the security connection string).
    1. Copy the Website\App_Config\Include\Examples\Sitecore.Owin.Authentication.IdentityServer.Disabler.config.example file and paste it into Website\App_Config\Environment and remove the .example.
1. You should now be able to log into the backend of Sitecore using Admin / b