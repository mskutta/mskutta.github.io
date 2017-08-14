---
layout: post
title: "Sitecore Telerik Patch"
date:   2017-08-11 00:00:00 -0500
categories: sitecore
tags: sitecore telerik patch
author: Mike Skutta
excerpt: Sitecore recently announced a critical security vulnerability with the Telerik Rich Text editor. It is highly encouraged that all Sitecore systems receive the related hotfix. We went through the process of applying the hotfix for all of our Sitecore clients.  We found that the patch provided by Sitecore introduced a breaking change to the Rich Text Editor.
---

* content
{:toc}

## Overview

Sitecore recently announced a **critical** security vulnerability with the Telerik Rich Text editor.  By default, Sitecore uses the Telerik Rich Text editor for the editing of Rich Text fields.  Here was the announcement that Sitecore made: [https://kb.sitecore.net/articles/978654](https://kb.sitecore.net/articles/978654).  The vulnerability impacts Sitecore versions 6.5 to 8.2 update 4.  More details about the vulnerability are on the Telerik site [http://www.telerik.com/support/kb/aspnet-ajax/details/cryptographic-weakness](http://www.telerik.com/support/kb/aspnet-ajax/details/cryptographic-weakness).  It is highly encouraged that all Sitecore systems receive the related hotfix.

We went through the process of applying the hotfix for all of our Sitecore clients.  We found that the patch provided by Sitecore introduced a breaking change to the Rich Text Editor.

## What is the Issue?

When editing a specific data scenario in the Rich Text Editor, an error will occur that will prevent updates to the system. The error reads: *Error while executing filter XHTML - TypeError: Cannot read property 'parentNode' of null*

![Error while executing filter XHTML](/images/sitecore-telerik-patch/error-message.png)

The data scenario is when you have a bulleted list **<ul>** nested inside a **<p>** tag as follows:
``` html
<p>
    <ul>
        <li>Lorem Ipsum</li>
    </ul>
</p>
```

This is technically invalid html, as a <ul> element should not be nested inside a <p> element.  The Telerik rich text editor does allow this markup.  We had cases where this markup is present, so we needed to address this.

## Who Is Impacted?

1. Any clients that are on version 6.5 to 8.0 of Sitecore.  This does not affect 8.1+ customers.

1. Any clients updating content with the data scenario described above.  Please note your client may never see this issue if they don’t have this data scenario in the Rich Text Editor

## How to Fix?

It appears that Sitecore upgraded Telerik.Web.UI.dll from version 2012 to 2014 as part of the patch.  Any upgrade comes with the potential of breaking changes.  This is what appears to have happened in this case.  Instead of upgrading the version of Telerik.Web.UI.dll, we can apply the Telerik patch for the original version of the Telerik.Web.UI.dll.  Patch versions do exist for the 2012 version of the Telerik.Web.UI.dll assembly.  The patched versions can be obtained from Telerik directly.

Assuming you already applied the Sitecore patch, here is how to apply the fix:
1. Revert the following 2 files back to the version that came with the base install of Sitecore.
    1. Telerik.Web.UI.Skins.dll
    1. Telerik.Web.UI.xml
1. Update the Telerik.Web.UI.dll with the patch version from Telerik that applies to the version that came with the base install of Sitecore.  Note: The patched version must be obtained from Telerik.
1. Remove the assembly redirect in the web.config.
``` xml
<dependentAssembly>
    <assemblyIdentity name="Telerik.Web.UI" publicKeyToken="121fae78165ba3d4" />
    <bindingRedirect oldVersion="2012.2.607.35" newVersion="2014.1.403.35" /> 
</dependentAssembly>
```

We reached out to Sitecore asking why they upgraded from 2012 to 2014.  We also asked if we can use the patched versions of the assemblies instead.  They replied saying "We intended to upgrade to the latest version possible. This was also recommended by Telerik.  If you don't have any issues with patched 2012 dlls, feel free to use them."

I hope this helps anyone else running into similar issues.

