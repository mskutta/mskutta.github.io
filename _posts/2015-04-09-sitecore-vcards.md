---
layout: post
title: "Sitecore vCards"
date:   2015-04-09 00:00:00 -0500
categories: sitecore
tags: sitecore vcard
author: Mike Skutta
excerpt: A vCard is essentially an electronic business card. Like a standard business card, vCards contain business related information about an individual or company. Usually this includes contact information such as the company name, the person’s name, address, phone numbers, email addresses and social media links. vCards can be attached to e-mail messages, shared through instant messaging, downloaded from web pages and distributed through various other forms of media.
---

* content
{:toc}

## Background

A [vCard](http://en.wikipedia.org/wiki/VCard) is essentially an electronic business card. Like a standard business card, vCards contain business related information about an individual or company. Usually this includes contact information such as the company name, the person&rsquo;s name, address, phone numbers, email addresses and social media links. vCards can be attached to e-mail messages, shared through instant messaging, downloaded from web pages and distributed through various other forms of media.

Applications that maintain contact information, such as Microsoft Outlook, can import data from vCards into address books. vCards provide a way to electronically distribute contact information without the need for end users to key in the contact information manually.

**A vCard is a file.**

The file has a format specification that indicates how applications should create and read from the file. This gives the ability for a website to provide links to vCard files that client applications can open and understand.

When building websites that include contact information for people, it is usually beneficial to provide a link to download the associated vCard. As mentioned above, this saves the extra steps a user would need to go through to manually key in the contact information into their favorite electronic address book. It provides an overall better user experience.

## Implementation on Sitecore

We wanted to make it as easy as possible to implement vCards on our Sitecore sites. We found that the source of the data contained within the vCards varies across our Sitecore implementations. This is typically due to differences in the data templates that represent a person. The best way for us to make vCard creation easier in Sitecore was to create a reusable API for the generation of the vCard file.

The API consists of a vCard object with properties that represent the data in the vCard file. Calling .ToString() on the object takes all of the data within the properties and returns a string that represents the vCard file in the proper format. We found this approach to be very flexible. The fields from the Sitecore item that represent a person are programmatically mapped to the properties on the vCard object. An ORM/Mapper can be used to make the mapping easier. The resulting string is then returned in a way that is compatible with the implementation. A Sitecore Web Form implementation may return the data differently than an MVC implementation.

We released this API to the community in [GitHub](https://github.com/onenorth/vcards). Feel free to use this API to help with your implementation of vCards.
