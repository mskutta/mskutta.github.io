---
layout: post
title: "Sitecore Dynamic robots.txt"
date:   2018-07-16 00:00:00 -0500
categories: sitecore
tags: sitecore nodejs express
author: Mike Skutta
excerpt: One of the development best practices we follow at One North is make the websites we build have great SEO. Public website content is crawled and properly indexed by search engines so they appear higher in Google search pages, and can be found more easily by our client’s customers looking for information. An important part of SEO that is sometimes overlooked is understanding the security balance of what to show publicly, and when to hide non-public pages from being crawled. For instance, we wouldn’t want testing environments to show up in public search engine results.
---

* content
{:toc}

## Overview

One of the development best practices we follow at One North is make the websites we build have great SEO. Public website content is crawled and properly indexed by search engines so they appear higher in Google search pages, and can be found more easily by our client’s customers looking for information. An important part of SEO that is sometimes overlooked is understanding the security balance of what to show publicly, and when to hide non-public pages from being crawled. For instance, we wouldn’t want testing environments to show up in public search engine results. 

In Sitecore, we need to serve up different robots.txt files based on environment. QA and staging environments should not be indexed and the robots.txt should indicate that. On the other hand, the production environment is public and should be indexed. To solve this problem, we created the Dynamic Robots module.

The Dynamic Robots module allows you to serve up a different robots.txt file per domain name. For example, a public facing domain would allow all URLs to be crawled, where an internal / staging domain would disallow all URLs. The robots.txt files follow a naming convention. The default robots.txt file is named robots.txt. This file is served if domain-specific robots.txt files are not provided. If the robots.txt file is not provided, then the following default is returned:

```txt
User-agent: *
Disallow: /
```

The default is done to prevent sites from accidentally being crawled.

Domain specific robots.txt files follow the convention ```robots-{domain name}.txt```. These specific files will be matched exactly by the domain name. If no match is made, the default is returned.

> NOTE: File names must be lower case.

## Native Sitecore

When running in the context of Sitecore, an HttpHander can be created to process ```robots.txt``` requests.  Below is an example of HttpHander code:

``` csharp
using System;
using System.IO;
using System.Collections.Generic;
using System.Text;
using System.Web;
using System.Reflection;
using System.Configuration;

namespace DynamicRobot
{
    public sealed class DynamicRobotModule : IHttpHandler
    {
        public void ProcessRequest(HttpContext context)
        {
            if (context != null)
            {
                if (string.Compare(context.Request.Url.PathAndQuery, "/robots.txt", true) == 0)
                {
                    HttpResponse response = context.Response;
                    response.Clear();
                    response.ContentType = "text/plain";

                    string currentHost = context.Request.Url.Host;
                    string hostMatchFileName = context.Server.MapPath(string.Format("robots-{0}.txt", currentHost));
                    string defaultFileName = context.Server.MapPath("robots.txt");

                    if (File.Exists(hostMatchFileName))
                    {
                        // found a host match between the request's host and a file on the hard drive, eg: /robots-myminisite.myfirm.com.txt

                        // Note: Read all text into memory to naturally remove the Byte Order Mark
                        response.Write(File.ReadAllText(hostMatchFileName));
                    }  
                    else if (File.Exists(defaultFileName))
                    {
                        // found /robots.txt at the root.

                        // Note: Read all text into memory to naturally remove the Byte Order Mark
                        response.Write(File.ReadAllText(defaultFileName));
                    }
                    else 
                    {
                        response.Write("User-agent: *\nDisallow: /");
                    }

                    response.Flush();
                    response.End();
                }
            }
        }

        public bool IsReusable
        {
            get { return false; }
        }
    }
}
```

The module can then be registered in the ```web.config```.

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>					
	<system.webServer>		
		<handlers>			
			<add name="DynamicRobot" path="/robots.txt" verb="*" type="DynamicRobot.DynamicRobotModule" />
		</handlers>		
	</system.webServer>	
</configuration>
```

## Headless Sitecore with NodeJS

When running Sitecore headless, one technology the public-facing site can run is [NodeJS](https://nodejs.org/) using [ExpressJS](https://expressjs.com/).  The following can be created to process ```robots.txt``` requests from a NodeJS / Express app.

**dynamic-robots.js**

``` js
const fs = require('fs')
const path = require('path')

var App = function () {
  var self = this

  self.init = function (app, configPath) {
    // Routes
    app.get('/robots.txt', async (req, res) => {
      const fallback = [
        'User-agent: *',
        'Disallow: /'
      ].join('\n')

      res.set({
        'Content-Type': 'text/plain'
      })

      var host = req.hostname.split(',', 1)[0]
      var filePath = null

      // Check for host only file
      if (!filePath) {
        var hostFilePath = path.join(configPath, 'robots-' + host + '.txt')
        if (fs.existsSync(hostFilePath)) { filePath = hostFilePath }
      }

      // Check for default file
      if (!filePath) {
        var defaultPath = path.join(configPath, 'robots.txt')
        if (fs.existsSync(defaultPath)) { filePath = defaultPath }
      }

      // Return the contents
      if (filePath) {
        res.sendFile(filePath, {}, function (err) {
          if (err) { res.send(fallback) }
        })
      } else {
        res.send(fallback)
      }
    })
  }
}

module.exports = new App()
```

The custom NPM module should then be registered and referenced within the application ```app.js``` file.
``` js
const robots = require('./dynamic-robots')
...
// Create app
const app = express()
const router = express.Router()
...
robots.init(app, __dirname)
```

> NOTE: ```__dirname``` is the path to the location of the config files.

This can optionally be converted to a custom NPM package; I will leave that as an exercise for the reader.  Please see [Private NPM Packages in BitBucket](/2018/05/25/private-npm-packages-in-bitbucket/) for information on hosting the NPM privately in BitBucket.

