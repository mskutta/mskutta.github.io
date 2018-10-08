---
layout: post
title: "Sitecore How-To: How to Quickly Export Data from Sitecore"
date:   2018-10-08 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Alex Pershteyn and Mike Skutta
excerpt: Sitecore has a built-in tool that allows to quickly export content in XML format. It has a very basic functionality out of the box, but could easily and quickly be customized to add custom business rules as needed. 
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sitecore has a built-in tool that allows you to quickly export content in XML format. It has a very basic functionality out of the box, but could easily and quickly be customized to add custom business rules as needed. 

## Step-by-step guide

Out of the box, you can generate the export by following these steps:

1. Login as admin.
2. Navigate to Control Panel → Localization → Export Languages (in Sitecore 7.5 and below it's Control Panel → Globalization → Export Languages).
3. Select a language (most likely English) and select a top-level Sitecore Item that will serve as a 'root' element for your export. Then click 'Next', and once the export completes, you should be able to get full XML export.

There are various problems with this out-of-the-box approach: [it doesn't include any 'Shared' fields](https://sitecore.stackexchange.com/questions/2076/is-there-a-way-to-include-shared-fields-during-export-languages-feature-in-8/), since technically these fields are not 'translatable'. By default, it is exporting all translatable fields, including the ones only used by Sitecore admins (like Display name, etc). For media items, the urls may not be correct, etc.

Here is what you need to customize this export:

1. Sitecore.Support.90534 patch - certain versions of Sitecore inject css in XML (this happened to me in 7.5 and 6.6, but I believe it also affects other versions) - make sure to install it if you see css references in your XML. More info about this patch is here: https://community.sitecore.net/developers/f/8/t/8075. You can download it here: Sitecore.Support.90534-7.2.3.0 (1).zip
2. Decomplie Sitecore.Shell.Applications.Globalization.ExportLanguage.ExportLanguageForm class from Sitecore.Client assembly, create a new class from it and alter the behavior. The changes will most likely need to happen in the ExportItem method. 
3. Copy from your website folder and add this file to your project: \sitecore\shell\Applications\Globalization\ExportLanguage\ExportLanguage.xml. You will need to modify the following namespace in this file <WizardForm CodeBeside="Sitecore.Shell.Applications.Globalization.ExportLanguage.ExportLanguageForm,Sitecore.Client" Submittable="false"> - replace it with your new namespace/class class.
4. Once you make all updates, follow the steps above to generate a new export.

> We have used this export approach several times, mostly for clients that required a data dump.