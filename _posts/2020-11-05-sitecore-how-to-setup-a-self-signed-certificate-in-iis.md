---
layout: post
title: "Sitecore How-To: Setup a Self Signed Certificate in IIS"
date:   2020-11-05 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: In order to setup a https site locally on your machine, you will need to create a self signed certificate and add SSL settings to your IIS site.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

In order to setup a https site locally on your machine, you will need to create a self signed certificate and add SSL settings to your IIS site.

## Step-by-step guide

### Create the self-signed certificate via IIS:

> Note: This method will not allow you to control the domain name(s) the certificate is for. IIS will use your machine name by default.

1. Open IIS Manager
1. Click the root node
1. Double click the Server Certificates feature
1. In Server Certificates Feature window => Actions pane: Create Self-Signed Certificate
    1. Specify friendly name for certificate. Your machine name works.
    1. Click ok

### Create the self-signed certificate via PowerShell

1. Open PowerShell as an administrator
1. Execute the following command
    ``` bash
    New-SelfSignedCertificate -DnsName "my.local.domain" -CertStoreLocation cert:\LocalMachine\My -FriendlyName "My Local Cert Name"
    ```
    
    > Note: The `-DnsName` parameter of the `New-SelfSignedCertificate` cmdlet takes an array of strings. You can add multiple domain names to a single certificate.
    ``` bash
    New-SelfSignedCertificate -DnsName "mysite", "mysite.local", "mysite.siteco.re" -CertStoreLocation cert:\LocalMachine\My -FriendlyName "My Local Cert Name"
    ```

    > Note: The `-NotAfter` parameter takes a DateTime, allowing you to set an expiration date for the certificate.
    ``` bash
    New-SelfSignedCertificate -DnsName "mysite" -CertStoreLocation cert:\LocalMachine\My -FriendlyName "My Local Cert Name" -NotAfter (Get-Date).AddYears(10)
    ```

    See the [documentation for New-SelfSignedCertificate](https://docs.microsoft.com/en-us/powershell/module/pkiclient/new-selfsignedcertificate?view=win10-ps) for more details.
Related article [here](https://babelway.zendesk.com/hc/en-us/community/posts/360027367853-HOW-TO-CREATE-SELF-SIGNED-SSL-CERTIFICATES-IN-WINDOWS-10).

#### Bind the new certificate to your website

1. Open IIS Manager
1. Click the site you want to add the certificate to.
1. Under "Edit Site" on the right-hand menu, click on Bindings
1. Click Add
    1. Type:  HTTPS
    1. IP Address: All Unassigned
    1. Port: 443
    1. Hostname:  your site name
    1. SSL Certificate:  choose your self-signed certificate you just created

> Note: If you want to force SSL, double click on SSL Settings feature => check Require SSL