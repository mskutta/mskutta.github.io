---
layout: post
title: "Sitecore How-To: Setup Sitecore with Media Library as a Separate Site"
date:   2019-10-15 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Pete Amundson and Mike Skutta
excerpt: There are multiple advantages to serving imagery and assets from different domains from your main website.  This is a way that the Sitecore Media Library can be served as a distinct, media-only website on a separate domain.  This can be used as-is or a global CDN can be placed in front of this.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

There are multiple advantages to serving imagery and assets from different domains from your main website.  This is a way that the Sitecore Media Library can be served as a distinct, media-only website on a separate domain.  This can be used as-is or a global CDN can be placed in front of this. 

## Step-by-step guide

### Step 1

The first step to serving the media library from a site in Sitecore is to create a new site configuration.  This is important because it restricts the domain to only the media library content (if you do not do this, then the /sitecore/content/ data will be serve-able from this domain as well and that is bad for SEO). Note, this site will not need many of the same functions that you typically will use for editing content (like webedit or preview):

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <sites>
      <site patch:before="*[@name='website']"
            name="cdn"
            hostName="cdn.localtest.me"
            virtualFolder="/"
            physicalFolder="/"
            rootPath="/sitecore/media library"
            startItem="/default"
            database="web"
            domain="extranet"
            allowDebug="false"
            cacheHtml="false"
            registryCacheSize="0"
            viewStateCacheSize="0"
            xslCacheSize="0"
            filteredItemsCacheSize="0"
            enablePreview="false"
            enableWebEdit="false"
            enableDebugger="false"
            disableClientData="false"
            blockAnonymousAccess="false"/>
    </sites>
  </sitecore>
</configuration>
```

> **Important Patch Info** The patch:before attribute will place this domain above the standard website site node.  This is important if the website node does not have a specific domain (especially helpful when in development).

### Step 2

The next step is to configure the media link builder to automatically link to the media-only domain:

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>    
    <settings>
      <!-- Always Include Server Url must be true to bring in the Media Link Server Url -->
      <setting patch:instead="*[@name='Media.AlwaysIncludeServerUrl']" name="Media.AlwaysIncludeServerUrl" value="true" />
      <!-- Point to the FQD (including protocol) -->
      <setting patch:instead="*[@name='Media.MediaLinkServerUrl']" name="Media.MediaLinkServerUrl" value="http://cdn.localtest.me" />
    </settings>
  </sitecore>
</configuration>
```

### Step 3

If you are using Sitecore 8.2 Update 5 (170728) or higher, the following settings can also be used.  These are only recommended for 8.2u5+ because "Media.AlwaysAppendRevision" is built into those versions.  If you wire your own media link builder to include revision info (see below for example), then feel free to use the bottom two settings as well:

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>    
    <settings>
      <!-- The revision number is generated each time a media item is updated, so the unique URL can be cached for a long time without issue. -->
      <setting patch:instead="*[@name='Media.AlwaysAppendRevision']" name="Media.AlwaysAppendRevision" value="true" />
      <!-- Setting this to true will tell caching layers (like client infrastructure, browsers, cdns) to fully cache the content on the url. Default is 'private' e.g. browser. -->
      <setting patch:instead="*[@name='MediaResponse.Cacheability']" name="MediaResponse.Cacheability" value="public" />
      <!-- Tells the caching layers how long to cache the content (longer is better when the revision above is used). Default is '7.00:00:00' which is seven days. -->
      <setting patch:instead="*[@name='MediaResponse.MaxAge']" name="MediaResponse.MaxAge" value="60.00:00:00" />
    </settings>
  </sitecore>
</configuration>
```

> **AlwayAppendRevision Patch** There is a patch for multi-lingual sites that use Media.AlwaysAppendRevision: https://github.com/sitecoresupport/Sitecore.Support.219261/releases

## For Sitecore 8.2u4 and Lower

The revision appending that is available for 8.2u5+ can be manually added using a custom Media Provider.  An example of how to do that is below:

### Step 1

Create your custom Media Provider:

```C#
namespace Website.Logic.Providers
{
  public class CDNMediaProvider : MediaProvider
  {
    public override string GetMediaUrl(MediaItem item) { return GetMediaUrl(item, MediaUrlOptions.Empty); }

    public override string GetMediaUrl(MediaItem item, MediaUrlOptions options)
    {
      Sitecore.Diagnostics.Assert.ArgumentNotNull(item, "item");

      // add the "rv" query string that contains a hashed version of the revision and updated time statistics
      return Sitecore.Web.WebUtil.AddQueryString(base.GetMediaUrl(item, options), false, "rv", HashingUtils.ComputeHash(item.InnerItem.Statistics.Revision + item.InnerItem.Statistics.Updated.ToString("yyyyMMddHHmmss")));
    }
  }
}
```

### Step 2

Configure Sitecore to use the custom provider when generating media library URLs:

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>    
    <mediaLibrary>
      <mediaProvider>
        <!-- Adds ability to add revision information to media urls in versions lower than 8.2u5 -->
        <patch:attribute name="type">Website.Logic.Providers.CDNMediaProvider, Website</patch:attribute>
      </mediaProvider>
    </mediaLibrary>
  </sitecore>
</configuration>
```

## References

* https://doc.sitecore.com/developers/91/sitecore-experience-management/en/sitecore-media-library-cdn-related-configuration-reference.html
* https://doc.sitecore.com/developers/91/sitecore-experience-management/en/developer-considerations-for-the-sitecore-media-library-cdn-support.html