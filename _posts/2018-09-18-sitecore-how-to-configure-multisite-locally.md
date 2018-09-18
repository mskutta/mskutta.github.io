---
layout: post
title: "Sitecore How-To: Configure Sitecore Multisite Locally"
date:   2018-09-18 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Kyle Mattimore and Mike Skutta 
excerpt: Sitecore can host multiple websites in the same environment/databases. They appear as other sites besides the primary site in the content tree.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge.  I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sitecore can host multiple websites in the same environment/databases. They appear as other sites besides the primary site in the content tree.

![Content Tree](/images/how-to-configure-sitecore-multisite-locally/image.jpg)

### Local setup

1. Open notepad++ or another text editor as an admin and open the file: **C:\Windows\System32\drivers\etc\hosts**. Inside this file, add on a new line: `127.0.0.1 {host name}`
1. Open IIS and navigate to your site on the left hand panel. Right click your site and click edit bindings. From there, hit the add button and add the host name.
1. Create/Open the config file for your project located in app Config/Include and bind the hostname of your local site to a site.

Other notes: 

The local domain names will be defined in an App Config/Include file. Each site will have a definition including the accessible host names, e.g. `hostName="mainsite|minisite"`. You can add to or edit this list, and/or give IIS another binding to access multiple sites at the same time.

The *targetHostName* does not support multiple hostnames, and is used for generating absolute URLS, etc. 

The site bindings are processed in the order they are defined. Note that the hostname binding will match somewhat loosely, so if you have `hostName="myminisite"` and `hostName="myminisite2"`, the myminisite hostname will catch both, and you will be miserable wondering why you can't hit the second minisite.

### Example

If I setup "mainsite" normally through the Sitecore installer, the installer created a hosts file entry and IIS binding for "mainsite". The installer doesn't know about the second site, so I must do these steps manually. Add `127.0.0.1 minisite` to the HOSTS file, and add a binding in the site in IIS for minisite. Now http://mainsite gives you the main site, and http://minisite gives you the second site. The mainsite and minisite hostnames must be registered in the hostName fields of the sites they belong.

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:set="http://www.sitecore.net/xmlconfig/set/">
  <sitecore>
    <sites>
      <site name="website" set:hostName="mainsite" />
      <site name="minisite" set:hostName="minisite" />
    </sites>
  </sitecore>
</configuration>

```