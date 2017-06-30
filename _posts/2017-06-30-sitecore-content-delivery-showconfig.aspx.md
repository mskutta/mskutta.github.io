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

I just wanted to share a quick shortcut I recently used.  I had the need to view the Sitecore configuration on a content delivery server.  The problem was the content delivery server was already hardened and all non-essential files were removed, including ShowConfig.aspx. I created a temporary ShowConfig.aspx that can be dropped anywhere on the content delivery server. A request can be made against this page to return the contents of the typical show config. Once you have what you need, you MUST delete the ShowConfig.aspx as there are no security checks preventing an unauthenticated user from viewing the configuration.






## ShowConfig.aspx


``` c#
<%@ Page Language="C#" AutoEventWireup="true" %>
<%@ Import Namespace="Sitecore.Configuration" %>
<%@ Import Namespace="System" %>
<%@ Import Namespace="System.Xml" %>

<script language="C#" runat="server">
    public void Page_Load(object sender, EventArgs e)
    {
        XmlDocument configuration = Factory.GetConfiguration();
        this.Response.ContentType = "text/xml";
        this.Response.Write(configuration.OuterXml);
    }
</script>
```

> You MUST delete this file immediately after use as there are no security restrictions
