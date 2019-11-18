---
layout: post
title: "Sitecore How-To: Enforce Required Alt Text on Image Items"
date:   2019-11-18 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: George West, Pete Amundson, and Mike Skutta
excerpt: It's generally a good idea to include alt text for web images, since it's good for SEO and accessibility. This article shows you how to use Validation Rules and Workflows force Sitecore content editors to fill out the Alt field on Image items, and otherwise prevent them from saving or publishing Image items with an empty Alt field.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

It's generally a good idea to include alt text for web images, since it's good for SEO and accessibility. This article shows you how to use Validation Rules and Workflows force Sitecore content editors to fill out the Alt field on Image items, and otherwise prevent them from saving or publishing Image items with an empty Alt field.

> Before making any change to the behavior of Sitecore, make sure to review the implications with the client.  While Alt Text can be informative to users and crawlers, it can also be distracting or irrelevant.  For example, only imagery that is considered content (like a bio photo, infographic, or other content-supporting images) should be announced to screen readers.  In cases where an image's use case is exclusive to the design of the site (e.g. presentation), the recommendation is to have an empty Alt tag.

By default, Sitecore provides out-of-the-box validation for the Alt field on Image items. When you upload a new Image to the Media Library, the Alt field is empty. Also, when viewing the Image item in Content Editor, a red marker appears to the left of the field and in the Validation Bar, indicating that the Alt field's current value raises a validation Error.

![Image Alt Field](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/image-alt-field.png "Image Alt Field")


![Image Alt Field](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/image-alt-field-validation.png "Image Alt Field")

## Validation Rules

Sitecore comes installed with a slew of predefined Validation Rules, found at:

* **/sitecore/system/Settings/Validation Rules/Field Rules**
* **/sitecore/system/Settings/Validation Rules/Item Rules**

![Validation Rules](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/validation-rules.png "Validation Rules")

In our case, Alt field validation is already enabled via a predefined field-level rule, called *Alt Required*. It's very similar to another predefined Validation Rule, *Required*, except that *Alt Required* has more specific Title, Description, and Text. Both the Required and Alt Required Validation Rules are handled by the same code; *Sitecore.Data.Validators.FieldValidators.RequiredFieldValidator,Sitecore.Kernel*. The code which defines a Validation Rule can either be set–as in this case–with the Validation Rule's Type field, or with a script entered into the Script section of Content Editor.

A Validation Rule's code definition can return one of many Validator Results, ranging from Valid to Fatal Error. Out of the box, the Alt Required validation rule throws an Error-level Validator Result.

![Validator Result](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/validator-result.png "Validator Result")

## Preventing Save

When attempting to save an item, Sitecore will check the validation on all fields. Suggestions and Warnings will allow you to save the item without complaint. Errors and Critical Errors will still allow you to save the item, but a dialog will appear, asking you to confirm before doing so. Fatal Errors actually prevent you from saving the item, until they are resolved. This is what we want!

![Validation Message](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/validation-message.png "Validation Message")

To get this for our Alt Required Validator Rule, we can override the Validator Result returned by its code definition.

1. Open **/sitecore/system/Settings/Validation Rules/Field Rules/System/** Alt Required in Content Editor
1. Go to the Parameters field under the Data section
1. Set the "Result" parameter to "FatalError"

    ![Validation Message Text](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/validation-message-text.png "Validation Message Text")

1. Save

Now, when attempting to save an Image with empty Alt text, Sitecore will instead show the dialog shown above.

(Optional) Showing Validation in Other UI Areas

As mentioned earlier, the Validator Results for an Image's Alt field show up in Content Editor and the Validation Bar. We can show Validator Results in other areas as well. This can be configured by editing template of the Field you're using the Validation Rule on (in this case, **/sitecore/templates/System/Media/Unversioned/Image/Image/Alt**), and selecting your Validation Rule for the fields in the Validation Rules section.

### Quick Action Bar

![Quick Action Bar](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/quick-action-bar.png "Quick Action Bar")

### Validate Button

![Validate Button](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/validate-button.png "Validate Button")

### Validation Bar

![Validation Bar](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/image-alt-field-validation.png "Validation Bar")

### Workflow

This field allows the Field to be subject to validation when using a *Validation Action* in a Workflow Command. We will use this later on, in "Preventing Publish".

## Preventing Publish

To prevent publishing an empty-Alt Image, we will use a custom workflow. The Image template inherits the *Publish Item Only* Workflow as it's Default workflow from the File template, so we will base our new Workflow off of it. The only difference between Publish Item Only and our custom Workflow will be a *Validation Action* that happens during the *Publish* Command when moving from the *Draft* state to the *Publish* state, which will catch any Validation Results over a given threshold.

Create Custom Workflow with Validation

1. Copy /sitecore/system/Workflows/Publish Item Only
1. Rename the copy to something like "Publish Item with Validation"

    ![Publish Item With Validation](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/publish-item-with-validation.png "Publish Item With Validation")

1. Open the copy in Content Editor, and edit the Initial state field so it points to the correct Draft item under the copy. Save.

    ![Initial State](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/initial-state.png "Initial State")

1. Edit all Commands under your copy so their Next state fields point to the correct State items under the copy (similar to step 4). Save.

    ![States](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/states.png "States")

    ![Next State](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/next-state-1.png "Next State")

    or  

    ![Next State](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/next-state-2.png "Next State")

    or 

    ![Next State](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/next-state-3.png "Next State")

1. Under the Draft State's Publish Command, insert a new Validation Action

    ![Insert Action](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/insert-action.png "Insert Action")

    ![Validation Action](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/validation-action.png "Validation Action")

1. Fill out the Validation Action's fields.
    1. **Type**: Sitecore.Workflows.Simple.ValidatorsAction, Sitecore.Kernel
    1. **Max Result Allowed** (highest level Validator Result allowed without triggering a validation failure): Warning
    1. Results (text result which displays for each Validator Result level)
        1. **Unknown Result**: [can leave blank]
        1. **Warning Result**: [can leave blank]
        1. **Error Result**: You cannot publish an item with validation errors.
        1. **Critical Error Result**: You cannot publish an item with critical validation errors.
        1. **Fatal Error Result**: You cannot publish an item with fatal validation errors.
    1. Save

Configure Image items to Use New Custom Workflow by Default
1. Open the __Standard Values of the Image template (**/sitecore/templates/System/Media/Unversioned/Image/__Standard Values**) in Content Editor
1. Change the Default workflow to the custom workflow

    ![Standard Values](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/standard-values.png "Standard Values")

1. Save

Configure the Image Alt Field to be Subject to Validation in Workflow
1. Open **/sitecore/templates/System/Media/Unversioned/Image/Image/Alt** in Content Editor
1. Under Validation Rules, edit the Workflow field, and select the Alt Required Validation Rule (under System)

    ![Workflow Field](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/workflow-field.png "Workflow Field")

1. Save

Now, when attempting to publish a new Image with empty Alt text, Sitecore will instead show the validation dialog with the Validation Action's message.

![Validation Results](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/validation-results.png "Validation Results")

> Validation Actions are only run in Publish **Commands**, during Workflow state changes.
> ![Publish Command](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/publish-command.png "Publish Command")
> Once a new item has been published via this Workflow, it is now in a Final Workflow State, and can be published via the normal publishing process regardless of validation.
> ![Final Workflow State](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/final-workflow-state.png "Final Workflow State")
> ![Publish Item](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/publish-item.png "Publish Item")

## Preventing Publish

There are some cases in which alternate text should not be included on an image. When an image is used as a "background" or otherwise a decorative element, it is actually preferred to omit alternate text, since screen readers and crawlers would otherwise emphasize the importance of these elements.

> For decorative images, it is actually best to still add the *alt* attribute, but to set it as an empty string. This is because some screen readers will otherwise announce the *src* URL attribute instead of quietly skipping over the image. `<img src="/images/separator.gif" alt="">`

To allow a specific Image to ignore the Required Alt validation rule, you can add the rule to the suppressed validation rules. To do this,

1. Open the Image in Content Editor
1. Delete any existing Alt text (without saving, since you can't!), and wait for validation to run

    ![Alt Text](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/exception1.png "Alt Text")

1. In the Validator Bar, right-click the marker for Alt Text, and click "Suppress Validation Rule"

    ![Suppress Validation Rule](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/exception2.png "Suppress Validation Rule")

1. The Alt Required Validation Rule should now appear in the Suppressed validation rules field, under the Validation Rules section.
This is specific to this Image item.

    ![Suppressed Validation Rules](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/exception3.png "Suppressed Validation Rules")

1. You should now be able to save the Image item, even with no Alt text

    ![No Alt Text](/images/sitecore-how-to-enforce-required-alt-text-on-image-items/exception4.png "No Alt Text")
