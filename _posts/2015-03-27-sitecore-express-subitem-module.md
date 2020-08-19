---
layout: post
title: "Sitecore: Express Subitem Module"
date:   2015-03-27 00:00:00 -0500
categories: sitecore
tags: sitecore data field
author: Mike Skutta
excerpt: We had a need to support relationships in Sitecore that also contained metadata or supplemental information about a relationship or multiple relationships. Examples of these types of relationships are... 1. The organizations a person belongs to and the roles they play within the organization. 2. The offices an employee belongs to including their contact information for that office. 3. A person’s education with the School, Degree, Description, and year obtained. This information is specific to the item (parent) that owns it. No other item has the same information. In these particular cases the information is part of the parent item. The information only exists if the parent exists. If the parent is deleted, this information is also deleted.
---

* content
{:toc}

## Overview
We had a need to support relationships in Sitecore that also contained metadata or supplemental information about a relationship or multiple relationships. Examples of these types of relationships are:

1. The organizations a person belongs to and the roles they play within the organization.
1. The offices an employee belongs to including their contact information for that office.
1. A person&rsquo;s education with the School, Degree, Description, and year obtained.

This information is specific to the item (parent) that owns it. No other item has the same information. In these particular cases the information is part of the parent item. The information only exists if the parent exists. If the parent is deleted, this information is also deleted.

## Sitecore

Sitecore supports this concept via parent and child items. The child items only exist if the parent exists. If the parent is deleted, the children are also deleted. Using the above examples we can see how this may work:

1. A **Person** would be the parent item. The child items would be an **Organization Information** item. The Organization Information would contain the relationship to the organization as well as a role field.
1. An **Employee** would be the parent item. The child items would be an **Office Information** item. The Office Information item would contain the relationship to the office as well as supplemental Phone number fields and maybe desk location fields.
1. A **Person** would be the parent item. The child would be an **Education** item. The education item would contain relationships to the School and Degree. Additional fields would exist for the description and the year the degree was obtained.

In simple cases the children can be direct children of the parent. In cases where more than one relationship type exists, the children can be placed in sub-folders. In the above example the education records and the organization records could be placed into sub-folders under the same person for better separation. Insert Options can be configured to limit the types of children that can be added.

Using the above approach, the content administrator needs to work with multiple items (parent and children) at a time, even though the information is really part of a single record. We wanted to try to improve this user experience by removing the need to go to many individual items to update a single record. We wanted the content administrator to see and edit all the information on a record all at once. What we came up with was the** Express Subitem Module**.

## Express Subitem Module
The Express Subitem Module brings the experience of editing child items within the context of the parent item. The parent and child items still exist, but the child items and folders are hidden so the content administrator does not need to be concerned maintaining separate child items.

The child items are actually editable within the same view as the parent item. Links are provided to add and delete the child items.

![Express Subitem Collapsed](/images/sitecore-express-subitem-module/express_subitem_collapsed_image_1.jpg)

![Express Subitem Expanded](/images/sitecore-express-subitem-module/express_subitem_expanded_image_2.jpg)

Because the child items are hidden from the content administrator we had to implement the following:

* The child items are automatically versioned along with the parent item. If the parent is at version 7, the child items will also be at version 7.
* Publishing settings are mirrored from the parent to the child items
* Workflow settings are mirrored from the parent to the child items.
* Locks are also mirrored from the parent to the child items.
* The child items are automatically named via configurable replacement tokens.
* Validation errors on the child items are visible when editing within the parent.
* The Express Subitem was implemented as a custom field type. The child item ids are stored within the field as if the field was a Multilist field. This allows easy look-up of the managed child items.

> Note: The child items are hidden. When manually publishing it is important to include sub-items. One way to prevent the content administrator from forgetting to publish the sub-items, is to create a custom workflow.

Our clients have found this module to be very useful, improving the overall user experience, and we hope you do too. We have made this module available for everyone to use. The source code and releases are located at [GitHub](https://github.com/onenorth/express-subitem).

This module is also available on the [Sitecore Marketplace](https://marketplace.sitecore.net/Modules/Express_Subitem.aspx).
