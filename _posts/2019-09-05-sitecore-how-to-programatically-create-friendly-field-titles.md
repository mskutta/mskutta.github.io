---
layout: post
title: "Sitecore How-To: Programmatically Create Friendly Field Titles"
date:   2019-09-05 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Alex Pershteyn and Mike Skutta
excerpt: Here's a script to create friendly field titles in Sitecore. It will convert all field names in Camel Case and Lower Camel Case format. You need to provide guid of the top-level node under Templates and it would recursively go through every template and update field names as needed.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Here's a script to create friendly field titles in Sitecore. It will convert all field names in Camel Case and Lower Camel Case format. You need to provide guid of the top-level node under Templates and it would recursively go through every template and update field names as needed.  

## Step-by-step guide

**CreateFieldTitles.aspx**
```
<%@ Page Language="C#" AutoEventWireup="true" Debug="true" Inherits="Website.Logic.Common.Pages.AdminPage" %>
<%@ Import Namespace="Sitecore.Data" %>
<%@ Import Namespace="Sitecore.Data.Items" %>
<%@ Import Namespace="Website.Logic.Common.Extensions" %>
<script runat="server">
    
    Guid templateFieldID = new Guid("{455A3E98-A627-4B40-8035-E683A0331AC7}");
    protected void Page_Load(object sender, EventArgs e)
    {
        if (!IsPostBack)
            DataBind();
    }
    
    void UpdateFriendlyTitles_Click(object sender, EventArgs e)
    {
        var rootItemId = new ID(new Guid(RootItemId.Text));
        var item = Database.GetDatabase("master").GetItem(rootItemId);
        if (item == null)
            throw new ApplicationException(string.Format("Cannot find item with id {0}", RootItemId.Text));
        AddFriendlyFieldTitles(item);
    }

    private void AddFriendlyFieldTitles(Item currentItem)
    {
        if (!currentItem.HasChildren)
            return;

        foreach (Item child in currentItem.Children)
        {
            if (child.TemplateID.ToGuid() == templateFieldID)
            {
                var titleField = child.GetValueOrDefault<string>("Title");

                if (string.IsNullOrEmpty(titleField))
                {
                    var nodeName = child.Name;
                    if (nodeName.IndexOf(" ") == -1)
                    {
                        //split string into words if CamelCase (or lowerCamelCase) is used
                        var nodeTitle = Regex.Replace(nodeName, "([a-z](?=[A-Z])|[A-Z](?=[A-Z][a-z]))", "$1 ");
                        child.Editing.BeginEdit();
                        child["Title"] = nodeTitle;
                        child.Appearance.DisplayName = nodeTitle;
                        child.Editing.EndEdit();
                    }
                }
              
            }
            AddFriendlyFieldTitles(child);
        }
    }
   

</script>
<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
<head id="Head1" runat="server">
    <title></title>
</head>
<body>
    <form id="form1" runat="server">
    <div>
        
        Root ID: <asp:TextBox Width="400px" ID="RootItemId" Text="" runat="server"></asp:TextBox>
        <br/><br/>
        <asp:Button ID="Update" OnClick="UpdateFriendlyTitles_Click" Text="Update Friendly Field Titles" runat="server" />
    </div>
    </form>
</body>
</html>
```