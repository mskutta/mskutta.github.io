---
layout: post
title: "Sitecore patch:instead in config"
date:   2017-08-09 00:00:00 -0500
categories: sitecore
tags: sitecore config
author: Mike Skutta
excerpt: During an exercise with our Sitecore configuration we notice a bug with the patch instead action that is described here  http://sitecore.stackexchange.com/questions/2049/patchinstead-removes-an-element-with-no-attributes
---

* content
{:toc}

## Overview

During an exercise with our Sitecore configuration we notice a bug with the patch:instead action that is described here:  [http://sitecore.stackexchange.com/questions/2049/patchinstead-removes-an-element-with-no-attributes](http://sitecore.stackexchange.com/questions/2049/patchinstead-removes-an-element-with-no-attributes)

## What is the Issue?

When using the **patch:instead** action and the replacement value is the same, it will cause the xml element to be removed.  Here is an example:

Sample patch config contains:
``` xml
<setting patch:instead="*[@name='Caching.DisableCacheSizeLimits']" name="Caching.DisableCacheSizeLimits" value="false"/>
```

Existing Sitecore.config setting is:
``` xml
<setting name="Caching.DisableCacheSizeLimits" value="false" />
```

This transform will remove the setting from Sitecore / ShowConfig.aspx.  If you run into the scenario of missing settings I would start looking here.

## Working Around the Issue

How do you work around this issue?  There are a couple ways:

1. Delete and replace in config as follows:
    ``` xml
    <setting name="Caching.DisableCacheSizeLimits">
            <patch:delete />
    </setting>
    <setting name="Caching.DisableCacheSizeLimits" value="false" />
    ```

1. Use **patch:attribute** as follows:
    ``` xml
    <setting name="Caching.DisableCacheSizeLimits">
            <patch:attribute name="value">false</patch:attribute>
    </setting>
    ```

1. Use **set:value** as follows:
    ``` xml
    <setting name="Caching.DisableCacheSizeLimits " set:value="false" />
    ```
    You will need to make sure that the configuration root node has xmlns:set="http://www.sitecore.net/xmlconfig/set/"

## Best Practice

As preference, we found the 3rd way to be the best option. The reason why we like this approach better than the patch:attribute is primarily to see the **patch:source** in the showconfig.xml.  Doing it the patch:attribute approach does not show the patch:source.  This will hide which config file is actually setting the value. 

### Example MyPatch.Config
``` xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:set="http://www.sitecore.net/xmlconfig/set/">
    <sitecore>
      <setting name="Caching.DisableCacheSizeLimits " set:value="false" />
    </sitecore>
</configuration>
```

### Example Result

This is the result of the above patch config:

``` xml
<setting name="Caching.DisableCacheSizeLimits" value="false" patch:source="MyPatch.config" />
```
> Notice the additional patch:source attribute.
