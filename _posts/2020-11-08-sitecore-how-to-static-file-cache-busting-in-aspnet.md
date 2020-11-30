---
layout: post
title: "Sitecore How-To: Static File Cache Busting in ASP.NET"
date:   2020-11-08 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: This is a guide on how to implement cache busting in Sitecore MVC.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

This is a guide on how to implement cache busting in Sitecore MVC.  A good explanation of cache busing is: "[What Is Cache Busting?](https://www.keycdn.com/support/what-is-cache-busting)".

## Step-by-step guide

### Utility Class

> Note: The following code should be updated per your project’s caching method.

``` csharp
public sealed class FingerprintUtility
{
    /// <summary>
    /// Computes the fingerprint of the provided file so it can be used for cache busting.
    /// </summary>
    /// <param name="rootRelativePath">Relative path to the file to compute the hash for.</param>
    /// <returns>The fingerprint of the file.</returns>
    public static string GetCacheBusterFingerprint(string rootRelativePath)
    {
        if (HttpRuntime.Cache[rootRelativePath] == null)
        {
            byte[] hash;
            string filePath = HostingEnvironment.MapPath("~" + rootRelativePath);
            var hasher = System.Security.Cryptography.MD5.Create();

            if (hasher == null)
            {
                filePath = null;
            }

            using (FileStream file = File.OpenRead(filePath))
            {
                hash = hasher.ComputeHash(file);
            }

            // The base64 string of the hash is cleaned up to provide a cleaner looking file version.
            HttpRuntime.Cache.Insert(rootRelativePath, Convert.ToBase64String(hash).Replace("/", "").Replace("+", "").Replace("=", "").ToLowerInvariant(), new CacheDependency(filePath));
        }

        return HttpRuntime.Cache[rootRelativePath] as string;
    }

    /// <summary>
    /// Produces a path to an asset with a cache busting value.
    /// </summary>
    /// <param name="assetPath"></param>
    /// <returns></returns>
    public static string GetCacheBusterPath(string assetPath)
    {
        string fingerprint = GetCacheBusterFingerprint(assetPath);
        int fileExtensionIndex = assetPath.LastIndexOf('.');

        // fingerprint is removed on incoming requests via URL Rewrite.
        return assetPath.Insert(fileExtensionIndex, string.Format(".v-{0}", fingerprint));
    }
}
```

### IS URL Rewrite Rule

``` xml
<rewrite>
  <rules>
    <rule name="fingerprint">
      <match url="([\S]+)(\.v-[a-zA-Z0-9]+\.)(js|css)"/>
      <action type="Rewrite" url="{R:1}.{R:3}"/>
    </rule>
  </rules>
</rewrite>
```

### Usage Example

``` html
<link href="@FingerprintUtility.GetCacheBusterPath("/Contents/css/styles.css")" rel="stylesheet" />
```

### Setting Cache-Control

This will depend on your implementation and client requirements.  On a public website without gated content, you should be able to have a Cache-Control policy of “public” with an expiration of 365 days.  How this is done can be debated.  I prefer to be specific in setting the cache control options on the specific folder versus setting this value at the root of the website.  Doing this at the root of the application, you might inadvertently make gated content or secure content publically exposed on the CDN.

There are two ways to do this, and both ways involve a web.config.  If a folder does not have a web.config, then simply dropping a web.config as follows will set the Cache-Control: public, max-age=315360000

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <staticContent>
            <clientCache cacheControlCustom="public" cacheControlMode="UseMaxAge" cacheControlMaxAge="365.00:00:00" />
        </staticContent>
    </system.webServer>
</configuration>
```

If the folder location already includes a web.config, and it can not be removed for another purpose you can update the web.config at the root with a web.debug.config or web.release.config transform as follows:

``` xml
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">	
	<!-- Doing this in the web.config due to existing web.config at root of assets -->
	<location path="assets" xdt:Locator="Match(path)" xdt:Transform="InsertIfMissing">
		<system.webServer>
			<staticContent>
				<clientCache cacheControlCustom="public" cacheControlMode="UseMaxAge" cacheControlMaxAge="365.00:00:00" />
			</staticContent>
		</system.webServer>
	</location>
</configuration>
```

### TODO:  Additional consideration

This cache busting only applies to files that are directly referenced in our views.  We would also need an additional process to rewrite all the images that are referenced in our generated CSS.  This will then complete the cache busting of our assets.

See examples:  https://stackoverflow.com/questions/35482122/cache-bust-background-images-in-my-css-with-gulp-without-having-to-edit-my-sass
