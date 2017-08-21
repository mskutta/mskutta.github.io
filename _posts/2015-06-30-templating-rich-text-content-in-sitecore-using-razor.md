---
layout: post
title: "Templating Rich Text Content in Sitecore using Razor"
date:   2015-06-30 00:00:00 -0500
categories: sitecore
tags: sitecore templating razor
author: Mike Skutta
image:
    url: /images/templating-rich-text-content-in-sitecore-using-razor/confirmation-email.png
    height: 426
    width: 668
excerpt: Contact us forms are common on the sites I work on. Many times, the end result of a user filling out a contact us form is sending a confirmation e-mail to that user. The confirmation e-mail generally includes information that the user filled out on the contact us form. This information is intermixed with other text content, such as labels and other messaging. Content managers may have a need to modify the text and layout within the confirmation e-mails on an ongoing basis.
---

* content
{:toc}

## Overview

Contact us forms are common on the sites I work on. Many times, the end result of a user filling out a contact us form is sending a confirmation e-mail to that user. The confirmation e-mail generally includes information that the user filled out on the contact us form. This information is intermixed with other text content, such as labels and other messaging. Content managers may have a need to modify the text and layout within the confirmation e-mails on an ongoing basis.

I just wanted to share one approach I used to support confirmation e-mails with dynamic content. The formatting, layout, and supporting text is all editable by a Content Manager. This approach can also be applied to other areas.

## Approach

One way to support the above requirements is to store the various components of a confirmation e-mail in a Sitecore Item. Fields can be defined to contain the Subject, Body, and other required fields. The body would contain the entire body content of the e-mail. Dynamic content could be injected into the body before the e-mail is sent. The same concept can be applied to the subject if dynamic content is required.

A common approach to support dynamic content is to use tokens within the text that are replaced. A token could follow the format #FIRSTNAME#, #LASTNAME#, #EMAIL#, or #PHONE#. All the content administrator would need to do is place tokens within the body’s text content where desired. The tokens are replaced with content from the Contact Us form when the e-mail is to be sent.

``` text
Hi #FIRSTNAME# #LASTNAME#

Thank you for filling out our contact us form. A representative will be reaching out to you shortly using #PHONE# or #EMAIL#.

Thank You
```

While this approach supports basic content, it does not support display logic. What happens if only e-mail or phone are required and only one of the two were entered. We would only want to display the one entered and exclude the “or”. We need some way to support this display logic.

## Template Engines

[Template Engines](https://en.wikipedia.org/wiki/Comparison_of_web_template_engines) allow you to build dynamic templates that generate and format text.   We commonly see template engines in use with MVC, where the view uses a template engine to generate the final content.  Web Form’s .aspx and .ascx also use a template engine to generate the final content.  You may use T4 templates to generate code.

We can satisfy our requirements using a template engine.  The open source Razor Engine uses the Razor template parser from Dot Net’s MVC views and made it available for use outside of MVC.  We can use the Razor Engine to generate our confirmation email content.

## Implementation

The Razor Engine can be installed via NuGet.  The package name is: **RazorEngine**.

Using the Razor Engine is simple to use. All you need to do is call RunCompile, pass the template, template key, model type, and model in, and you get the generated content out.

``` csharp
var result = Engine.Razor.RunCompile(template, "TemplateKey", model.GetType(), model);
```

In our example, the model would be:

``` csharp
public class ContactInfo {
	public string FirstName { get; set; }
	public string LastName { get; set; }
	public string Email { get; set; }
	public string Phone { get; set; }
}
```

The template would be:

``` text
Hi @Model.FirstName @Model.LastName

Thank you for filling out our contact us form. A representative will be reaching out to you shortly using @string.Join(" or ", new[] {Model.Email, Model.Phone}.Where(x => !string.IsNullOrEmpty(x))).

Thank you!
```

As you can see, the template supports complex logic. This is powerful when there is a need for more complicated layouts. I found that the Razor syntax is generally supported by the Rich Text fields in Sitecore. Most of the time, the Razor syntax can be entered directly into the Design view of the Rich Text editor.

![Confirmation Email](/images/templating-rich-text-content-in-sitecore-using-razor/confirmation-email.png)

A Complete example using a Sitecore Item where the above template is stored in the Body field:

``` csharp
public string GetConfirmationEmailBody(ContactInfo model, Item email)
{
  var template = email["Body"];
  return Engine.Razor.RunCompile(template, "Body", model.GetType(), model);
}
```

## Other Considerations

The possibilities of using the Razor Engine to process templates stored within Sitecore rich text fields opens up a bunch of possibilities. There are a few things to keep in mind:

1. Doing anything other than accessing model properties off a model object may be complex for content administrators. Know your content administrators. You may want to limit editing the templates to people more experienced with Razor.
1. There is no intellisense when editing the Razor syntax from within a Sitecore rich text field. Document all the model properties you want the content administrators to be aware of.
1. Provide appropriate error handling in case the template cannot be compiled due to syntax or other errors.
1. Generally, the Razor Engine supports whatever you can do when using Razor in MVC. This is very powerful, but can also be very dangerous. Malicious code can be added to the templates. Editing of the templates should be locked down to people you trust. When in doubt, do not use the approach.

## Conclusion

I hope you found that it is easy to create Razor based templates within Sitecore Rich Text fields. Most commonly this can be used to generate email content, but it can be expanded to other areas.