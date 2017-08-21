---
layout: post
title: "Sitecore Content Anonymizer"
date:   2016-06-16 00:00:00 -0500
categories: sitecore
tags: sitecore content anonymizer
author: Mike Skutta
excerpt: I was tasked to create a sales demo from an existing Sitecore training site. The existing training site was completely functional and had a good amount of data. There were a couple items that I needed to address to convert the training site into a sales demo. First, I needed to update the look and feel of it so it wouldn’t resemble the training site. Secondly, and most importantly, I needed to scramble the data so it was unrecognizable.
---

* content
{:toc}

## Introduction

I was tasked to create a sales demo from an existing Sitecore training site. The existing training site was completely functional and had a good amount of data. There were a couple items that I needed to address to convert the training site into a sales demo. First, I needed to update the look and feel of it so it wouldn’t resemble the training site. Secondly, and most importantly, I needed to scramble the data so it was unrecognizable.

At first, I thought I could manually go through the items and replace the text with Lorem Ipsum text. I quickly came to the realization that there were more than just generic text blocks within the site. How would I handle names, addresses, phone numbers, email addresses and dates? This quickly started to feel very tedious.

I recalled using “Faker” to generate random, but realistic, data for a Node.js application that I had worked on. Faker is a library for generating fake data such as names, addresses and phone numbers. The library has been ported to many languages including, Perl, Ruby, PHP, NodeJS and .Net. I thought to myself, I could take the data from Faker to anonymize the existing data and still have it look realistic.

I proceeded down the path of building the Sitecore Content Anonymizer. I used the concepts of “Faker” in the implementation for the data. I needed this to work with any item / template. It is difficult to detect what data is stored in which field, so I also needed a way to specify the type of data that goes into each field. The following is an overview of what I implemented and how to use it. I also released the implementation as a Sitecore Marketplace Module. The link to the marketplace module is here: [https://marketplace.sitecore.net/Modules/C/Content_Anonymizer.aspx?sc_lang=en](https://marketplace.sitecore.net/Modules/C/Content_Anonymizer.aspx?sc_lang=en).

## Overview

The Sitecore Content Anonymizer allows anonymization of the field values on items. Anonymization is performed per template type on a field-by-field basis. The Content Anonymizer allows administrators to anonymize data within Sitecore's content tree. A few examples of how data can be anonymized are as follows:
1. Paragraph text replaced with Lorem Ipsum text.
1. First and last names replaced with randomly selected names.
1. Email addresses replaced with new addresses based on the randomly generated names.
1. Dates replaced with randomly generated dates.

There may be several reasons to anonymize data:
1. Data from an existing implementation might need to be anonymized so it can be used for a demo.
1. Data from live should not be brought down to development due to confidentiality.
1. Data needs to be sent to Sitecore for support, but needs to be anonymized for confidentiality.
1. Inappropriate test data from dev should not make it to QA

## Features

Here are the key features of the Sitecore Content Anonymizer:
1. Anonymize the values of fields on items based on the underlying template.
1. Anonymization is applied to selected items of the selected template type.
1. Only the latest version of the items are anonymized. **All prior versions of an item will be removed**.
1. All language versions are anonymized. Currently English-based content will be used for some anonymization types.
1. Only fields with an anonymization type selected will be anonymized.
1. Only fields with inner values are anonymized. Fields containing the standard value, default value, fallback value or inherited value are not.
1. 30+ out-of-the-box field value anonymization formats
1. Basic custom field formats are supported. This allows combining one or more field values into a string for use on another field.
1. Item renaming based on custom fields is supported. Items names will be updated to follow the configured Sitecore naming conventions.
1. Global search and replace can be performed on all fields that are not flagged to be anonymized.

> **Note:** Anonymization is a permanent action to your data. It is recommended to back up the database(s) before anonymizing.

> **Cautions:** Only content you choose to be anonymized will be anonymized. This only includes content in the content tree. Other content and data will NOT be anonymized.

## Installation

I implemented the Sitecore Content Anonymizer as a Sitecore Module. The module is located at: [https://marketplace.sitecore.net/Modules/C/Content_Anonymizer.aspx?sc_lang=en](https://marketplace.sitecore.net/Modules/C/Content_Anonymizer.aspx?sc_lang=en).

Please follow the directions at the GitHub repository [https://github.com/onenorth/content-anonymizer](https://github.com/onenorth/content-anonymizer) under **Installation** and **Configuration** to install the module.

## Usage

The Content Anonymizer lives under the Sitecore Admin folder. To run the Anonymizer, navigate to:

http://site.com/sitecore/admin/contentanonymizer

You will be required to sign in if you are not already signed in.

### Select Template

To start, you first need to select a template. Data is anonymized based on the template selected. Only items based on the chosen template can be anonymized. To select a template, start typing the template name in the type-ahead.

A list of templates should appear that contain the text you typed. Select the template you desire.

![Select Template](/images/sitecore-content-anonymizer/select-template.png)

Once the template is chosen, additional options appear.

### Configure Search and Replace

You can define a global search and replace for text-based fields that are not chosen to be anonymized. If you need to add a search and replace, click the Add button. Click the Add button as many times as you need search and replace. The content entered in the Replace field is replaced with the content entered in the With field. The replacement is case sensitive.

![Configure Search and Replace](/images/sitecore-content-anonymizer/configure-search-and-replace.png)

### Configure Item Names

You can optionally rename the items that are anonymized. Currently, Custom Format and Lorem Replace anonymization types are supported. If renaming, choose an anonymization type.

For Custom Format, type the name of the custom format to use and choose the item in the type-ahead. The type-ahead will search the names provided in the Custom Format section. You can define a custom format by clicking the "...". The Configure Custom Formats dialog will appear. You will need to specify the language that the custom format will use to populate the name.

![Configure Item Names](/images/sitecore-content-anonymizer/configure-item-names.png)

### Configure Custom Formats

Custom formats allow you to combine one or more value(s) into a single string format. The resulting string can be used for text-based fields or the name of the item.

If you need a custom format, click the **Add** button. Click Add as many times as you need custom formats.

Each entry needs a **name**. The name is used when specifying the Item Name or Field format. The name can be anything you desire.

Each entry also needs at least one **token**. The token is tied to a field and represents the chosen field. The first field, will be assigned "$0" as the token. The second "$1", third "$2" and so on. The tokens are used in the format string.

A **format** string also needs to be specified. The format string should contain the tokens and any desired surrounding text. The result shows the interpreted format.

An example may be as follows for an email address related to person template:
* **Name:** Email
* **Tokens:** $0&#124;FirstName $1&#124;LastName
* **Format:** $0_$1@domain.com
* **Result:** FirstName_LastName@domain.com

![Configure Item Names](/images/sitecore-content-anonymizer/configure-custom-formats.png)

### Configure Fields

Choose which fields you want to anonymize by specifying the type of anonymization. Fields that do not have a selected anonymization type will not be anonymized. Relationships remain as-is (Anonymize relationships by anonymizing the target template).

![Configure Item Names](/images/sitecore-content-anonymizer/configure-fields.png)

### Select Items

Choose which items you want to anonymize. You can optionally select all with the all items checkbox.

![Configure Item Names](/images/sitecore-content-anonymizer/select-items.png)

### Running

To run the anonymization, click the Anonymize button. Note: The anonymize button appears disabled if required fields are not populated. Please make sure all required fields have been filled out. You will see a confirmation dialog appear that summarizes the selections.

![Configure Item Names](/images/sitecore-content-anonymizer/confirm.png)

If everything looks okay, click the **Anonymize** button on the dialog.

## Summary

Hopefully, the Sitecore Content Anonymizer proves to be useful.  Please let me know if you have any suggestions or feedback.

In the future, I hope to add better multilingual support.  A data generator is also a natural next step…
