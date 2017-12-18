---
layout: post
title: "Glass.Mapper, Rich Text Fields, and Links in Sitecore"
date:   2017-12-18 00:00:00 -0500
categories: sitecore
tags: sitecore glass.mapper links rich-text
author: Mike Skutta
excerpt: We noticed links within the content of rich text fields in Sitecore were not always rendering as we expected using Glass.Mapper.  We had various requirements where different combinations of settings resulted in different behaviors.  For example, A setting that would work for data loaders using Glass.Mapper would result in links that would not render correctly on the front end. Here are some of our findings.
---

* content
{:toc}

## Overview

We noticed links within the content of rich text fields in Sitecore were not always rendering as we expected using Glass.Mapper.  We had various requirements where different combinations of settings resulted in different behaviors.  For example: A setting that would work for data loaders using Glass.Mapper would result in links that would not render correctly on the front end. Here are some of our findings.

## Configure

There are a few ways to configure Glass.Mapper to handle Rich Text Fields.  Each approach has different outcomes.

1. No Sitecore Field Settings are specified.
    ``` csharp
    [SitecoreField(FieldName = "MyRichTextField ")]
    System.String MyRichTextField { get; set; }
    ```
    * Sitecore will use the Render Field pipeline when rendering this field. This means links within the field will be processed and converted.
    * Sitecore will not let you use this field to create/modify content. 
1. SitecoreFieldSettings.RichTextRaw is specified
    ``` cshtml
    [SitecoreField(FieldName = "MyRichTextField ", Setting = SitecoreFieldSettings.RichTextRaw)]
    System.String MyRichTextField { get; set; }
    ```
    * Sitecore will only call Render Field pipeline when rendering this field in edit mode. This means links within the field are not processed and converted when rendered on the front end. Links will have the default .ashx extension.
    * Sitecore will let you use this field to create/modify content. 
1. SitecoreFieldSettings.ForceRenderField is specified
    ``` cshtml
    [SitecoreField(FieldName = "MyRichTextField ", Setting = SitecoreFieldSettings.ForceRenderField)]
    System.String MyRichTextField { get; set; }
    ```
    * This setting will have the same functionality as not specifying any settings.
    * SitecoreFieldSettings.ForceRenderField was designed to be used with Single Line Text fields so their content could be forced to use the Render Field pipeline. See http://docs.glass.lu/html/d44739b2-5f0e-d862-f62c-86c50f0b262f.htm
    * **Due to the previous two facts, there is no need to use it on Rich Text fields.** 

## Rendering Links

To have links within your Rich Text fields to render with user-friendly URLs (not with the .ashx extension), the setting **Media.RequestExtension** needs to be blank and your Rich Text fields must not have **SitecoreFieldSettings.RichTextRaw**.

If you need to use SitecoreFieldSettings.RichTextRaw (due to a data importer using the Glass models), you will need to provide an extension method to expand links for the front end.

Expand Links Extension Method
``` csharp
public static class HtmlHelperExtensions
{
    public static IHtmlString Raw(this HtmlHelper helper, string value, bool expandLinks)
    {
        return new HtmlString(!string.IsNullOrEmpty(value) ? (expandLinks ? DynamicLink.ExpandLinks(value) : value) : value);
    }
}
```

Expand Links Usage
``` csharp
@Html.Raw(Model.MyRichTextField, true)
```

## Data Importers

If you are using Glass objects to import data, Rich Text fields need to have SitecoreFieldSettings.RichTextRaw. If they do not, you will get the following error:

*It is not possible to save data from a rich text field when the data isn't raw.Set the SitecoreFieldAttribute setting property to SitecoreFieldSettings.RichTextRaw for property MyRichTextField on type My.Domain.Object*
