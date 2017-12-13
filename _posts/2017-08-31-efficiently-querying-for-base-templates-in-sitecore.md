---
layout: post
title: "Efficiently Querying for Base Templates in Sitecore"
date:   2017-08-31 00:00:00 -0500
categories: sitecore
tags: sitecore base-template 
author: Mike Skutta
excerpt: I was conducting code reviews of Sitecore sites and noticed in some cases inefficient code was being used to determine if an Item had a base template.  I saw this code pattern more than once and figured it would be beneficial to share a more performant approach.
---

* content
{:toc}

## Overview

I was conducting code reviews of Sitecore sites and noticed in some cases inefficient code was being used to determine if an Item had a base template.  I saw this code pattern more than once and figured it would be beneficial to share a more performant approach. 

## What is the Issue?

Typically helper methods may be created to return if an Item has a base template.  These helper methods may be called **HasTemplate** or **HasBaseTemplate**.  Typically, I saw coding patterns similar to the below:

``` csharp
public bool HasTemplate(TemplateItem template, params Guid[] templateIds)
{
    Assert.ArgumentNotNull(template, "template");
    Assert.ArgumentNotNull(templateIds, "templateIds");

    return templateIds.Any(x => x == template.ID.Guid) || template.BaseTemplates.Any(baseTemplate => HasTemplate(baseTemplate, templateIds));
}
```

I have also seen this:

``` csharp
public static bool HasBaseTemplate(this TemplateItem template, ID templateId, bool includeSelf = false, bool recursive = true)
{
    if (includeSelf && template.ID == templateId)
    {
        return true;
    }

    if (recursive)
    {
        foreach (TemplateItem baseTemplate in template.BaseTemplates)
        {
            if (baseTemplate.HasBaseTemplate(templateId, includeSelf, recursive))
            {
                return true;
            }
        }
    }

    return false;  
}
```

In each example above the code is iterating the current item and its base templates to determine if a template exists.  It will keep going until all inherited template is exhausted.  Both of the above code samples result in higher CPU.

## The Better Way

A more performant way to implement this code block is to leverage **Sitecore.Data.Managers** as follows:

``` csharp
using Sitecore.Data.Managers;
  
public static bool HasBaseTemplate(this Item item, ID templateID)
{
    if (item == null) return false;

    if (TemplateManager.GetTemplate(item).GetBaseTemplates().Any(template => template.ID == templateID))
        return true;

    return false;
}
```

In one case, applying the more efficient code reduced the CPU by 40% and the total request time by 30%.