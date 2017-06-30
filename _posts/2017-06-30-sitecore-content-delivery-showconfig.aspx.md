---
layout: post
title: "Sitecore Content Delivery ShowConfig.aspx"
date:   2017-06-30 00:00:00 -0500
categories: Sitecore
tags: ShowConfig.aspx CD
author: Mike Skutta
---

* content
{:toc}

## Overview

I just wanted to share a quick shortcut I recently used.  I had the need to view the Sitecore configuration on a content delivery server.  The problem was the content delivery server was already hardened and all non-essential files were removed, including ShowConfig.aspx. I created a temporary .aspx file that can be dropped anywhere on the content delivery server. A request can be made against this page to return the contents of the typical show config. 




## Show Config .aspx File

The name of the file should be scrambled so it is not easily discoverable.  The file could be named after a random GUID such as: **327bedd00aaf4288abe6bd521aae2ff8.aspx**

To add an extra layer of security, an API Key needs to be registered in the .aspx and passed in the header when requesting the page. This prevents crawlers and others from requesting the page unless the API Key is known.  The API Key must be randomly generated and added to the .aspx.  [PostMan](https://www.getpostman.com/) can be used to send the API Key in the header.

> NOTE: The .aspx is meant to be temporary.  Once you requested the configuration, you should delete the temporary .aspx to prevent others from requesting the config.  There is sensitive data in the configuration.


``` c#
<%--
    When naming this file, please use a hard to discover name, such as {Random GUID}.aspx
    Please update the API_KEY below with something unique before using.
    Use PostMan to include the apikey in the header when requesting this page.
    This is intended to be a temporary file that must be removed when finished.    
--%>

<%@ Page Language="C#" AutoEventWireup="true" %>
<%@ Import Namespace="Sitecore.Configuration" %>
<%@ Import Namespace="System" %>
<%@ Import Namespace="System.Xml" %>

<script language="C#" runat="server">
    private const string API_KEY = "c9f1eab8b35944a8aa297277b9667758"; // Replace with a new unique key

    public void Page_Load(object sender, EventArgs e)
    {
        if (Request.Headers["apikey"] == API_KEY) 
        {
            XmlDocument configuration = Factory.GetConfiguration();
            this.Response.ContentType = "text/xml";
            this.Response.Write(configuration.OuterXml);
            return;
        }
        throw new HttpException(401, "Auth Failed");
    }
</script>
```

## Usage

When you are directly on the Content Delivery server, you can use notepad to create an empty .aspx with a unique, hard to discover, name. Paste the contents from above into that file.  Update the **API_KEY** with a new unique API Key.  This is a quick way to get a functioning ShowConfig.aspx on a server that does not normally have it.

> NOTE: You MUST delete this file immediately after using to further prevent unauthorized access.

## PostMan

To request the .aspx, a request needs to be made with the **apikey** in the header.  You can use [PostMan](https://www.getpostman.com/) for this.

![PostMan](/images/sitecore-content-delivery-showconfig/postman.png)

