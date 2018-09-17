---
layout: post
title: "Sitecore How-To: Configure Media Item Links to Open or Download"
date:   2018-09-17 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Kyle Mattimore and Mike Skutta
excerpt: Configure Sitecore to encourage the browser to open or download a type of file when linking to it.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Configure Sitecore to encourage the browser to open or download a type of file when linking to it.

> This is only a suggestion to the browser, not a rule. Certain users' browsers may be set to 'always download/open files of this type,' or have specific applications configured.

A configuration setting per file type can be set to force a download.  Add a Sitecore config file in App_Config/Include for forceDownload. 

This example is for PDF files: 

```xml
<?xml version="1.0"?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <mediaLibrary>
      <mediaTypes>
        <mediaType name="PDF file" extensions="pdf">
          <mimeType>application/pdf</mimeType>
          <forceDownload>false</forceDownload>
          <sharedTemplate>system/media/unversioned/pdf</sharedTemplate>
          <versionedTemplate>system/media/versioned/pdf</versionedTemplate>
        </mediaType>
      </mediaTypes>
    </mediaLibrary>
  </sitecore>
</configuration>
```

After adding the patch, verify the forcedownload value has changed in showconfig.aspx.

Look for the Content-Disposition HTTP header on the file's response in the network tab in dev tools. If there is no header, forceDownload is likely false. If the header value contains 'attachment', this indicates forceDownload is true. 

> CACHING: The Sitecore media cache is stored on disk and is valid for 90 days. If anyone has recently accessed these media items, they will not reflect the config changes. Delete the MediaCache folder in the Website directory.