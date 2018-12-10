---
layout: post
title: "Sitecore Troubleshooting: Sitecore Site Always Returns 404 - Not Found"
date:   2018-12-10 00:00:00 -0500
categories: sitecore
tags: sitecore-troubleshooting
author: Erik Carron and Mike Skutta
excerpt: When trying to setup an instance of a Sitecore site and all pages return the 404 page.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore troubleshooting articles. These articles are meant to be quick guides to troubleshoot various issues within Sitecore. The troubleshooting articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## Problem

When trying to setup an instance of a Sitecore site and all pages return the 404 page.

Another indication of this issue is you see the request is for a site other than "website": ```http://mysite:80/notfound?item=%2f&user=sitecore%5cAnonymous&site=scheduler```.  In this case the site is **scheduler**.

![Not Found](/images/sitecore-troubleshooting-site-always-returns-404-not-found/image2017-6-16_8-21-54.png)

## Solution

The hostname binding is incorrect in the Sitecore config so Sitecore defaults to the last site defined in the configuration.

1. Find the config file where the "website" site node is modified. (this varies project to project and is not Web.config!)
1. For the site "website", the hostName attribute should be blank and it should be the last site node in the sites section. This was make the "website" site the default site.

**Site configuration example**

```xml
<sites>
    <site name="shell" virtualFolder="/sitecore/shell" physicalFolder="/sitecore/shell" rootPath="/sitecore/content" startItem="/home" language="en" database="core" domain="sitecore" loginPage="/sitecore/login" content="master" contentStartItem="/Home" enableWorkflow="true" enableTracking="false" analyticsDefinitions="content" xmlControlPage="/sitecore/shell/default.aspx" browserTitle="Sitecore" htmlCacheSize="10MB" registryCacheSize="15MB" viewStateCacheSize="1MB" xslCacheSize="25MB" disableBrowserCaching="true"/>
    <site name="login" virtualFolder="/sitecore/login" physicalFolder="/sitecore/login" enableTracking="false" database="core" domain="sitecore" disableXmlControls="true"/>
    <site name="admin" virtualFolder="/sitecore/admin" physicalFolder="/sitecore/admin" enableTracking="false" enableWorkflow="true" domain="sitecore" loginPage="/sitecore/admin/login.aspx"/>
    <site name="service" enableTracking="true" virtualFolder="/sitecore/service" physicalFolder="/sitecore/service"/>
    <site name="modules_shell" virtualFolder="/sitecore modules/shell" physicalFolder="/sitecore modules/shell" rootPath="/sitecore/content" startItem="/home" language="en" database="core" domain="sitecore" content="master" enableTracking="false" enableWorkflow="true"/>
    <site name="modules_website" virtualFolder="/sitecore modules/web" physicalFolder="/sitecore modules/web" rootPath="/sitecore/content" startItem="/home" language="en" database="web" domain="extranet" allowDebug="true" cacheHtml="true"/>
    <site name="second-website" hostName="secondary.com" targetHostName="secondary.com" enableTracking="true" virtualFolder="/" physicalFolder="/" rootPath="/sitecore/content" startItem="/home" database="web" domain="extranet" allowDebug="true" cacheHtml="true" htmlCacheSize="50MB" registryCacheSize="0" viewStateCacheSize="0" xslCacheSize="25MB" filteredItemsCacheSize="10MB" enablePreview="true" enableWebEdit="true" enableDebugger="true" disableClientData="false" cacheRenderingParameters="true" renderingParametersCacheSize="10MB"/>
    <site name="website" hostName="" targetHostName="primary.com" enableTracking="true" virtualFolder="/" physicalFolder="/" rootPath="/sitecore/content" startItem="/home" database="web" domain="extranet" allowDebug="true" cacheHtml="true" htmlCacheSize="50MB" registryCacheSize="0" viewStateCacheSize="0" xslCacheSize="25MB" filteredItemsCacheSize="10MB" enablePreview="true" enableWebEdit="true" enableDebugger="true" disableClientData="false" cacheRenderingParameters="true" renderingParametersCacheSize="10MB"/>
    <site name="scheduler" enableTracking="false" domain="sitecore"/>
    <site name="system" enableTracking="false" domain="sitecore"/>
    <site name="publisher" domain="sitecore" enableTracking="false" enableWorkflow="true"/>
</sites>
```