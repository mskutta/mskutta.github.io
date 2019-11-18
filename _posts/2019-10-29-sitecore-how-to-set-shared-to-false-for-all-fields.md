---
layout: post
title: "Sitecore How-To: Set Shared to False for All Fields"
date:   2019-10-29 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Alex Hawley, Kyle Mattimore, and Mike Skutta
excerpt: You may have the need to switch all shared fields to versioned for multi-lingual support.  Here is a script to help out.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

You may have the need to switch all shared fields to versioned for multi-lingual support.  Here is a script to help out.

## Step-by-step guide

Execute the following script:

```c#
Database masterDb = Sitecore.Configuration.Factory.GetDatabase("master");
 
using (new SecurityDisabler())
{
    try
    {
        var templates = Sitecore.Data.Managers.TemplateManager.GetTemplates(masterDb);
 
        foreach (var template in templates.Values.ToList())
        {
            if (template.FullName.StartsWith("User Defined"))
            {
                var tmpl = masterDb.GetTemplate(template.ID);
 
                foreach (var section in tmpl.GetSections())
                {
                    foreach (var templateFieldItem in section.GetFields())
                    {
                        string shared = templateFieldItem.InnerItem[TemplateFieldIDs.Shared];
 
                        if(shared == "1")
                        {
                            templateFieldItem.BeginEdit();
                            templateFieldItem.InnerItem[TemplateFieldIDs.Shared] = "0";
                            templateFieldItem.EndEdit();
                        }
                    }
                }
            }
        }
    }
    catch (Exception ex)
    {
         
    }
}
```