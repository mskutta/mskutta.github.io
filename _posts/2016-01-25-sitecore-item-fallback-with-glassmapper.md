---
layout: post
title: "Sitecore: Item Fallback with Glass.Mapper"
date:   2016-01-25 00:00:00 -0500
categories: sitecore
tags: sitecore glass.mapper localization
author: Mike Skutta
excerpt: If you’re building sites with a global presence and multiple languages/regions, you may need some kind of language fallback functionality. Language fallback is the process of determining what language to serve when the content has not been translated in the requested language. In Sitecore, language fallback can occur at the item level and the field level. At the item level, if there is no version in the requested language, a fallback language version can be served instead. At the field level, if a field does not have content for the requested language, content for that field can be pulled from the fallback language version of the item.
---

* content
{:toc}

## Overview

If you're building sites with a global presence and multiple languages/regions, you may need some kind of language fallback functionality. Language fallback is the process of determining what language to serve when the content has not been translated in the requested language. In Sitecore, language fallback can occur at the item level and the field level. At the item level, if there is no version in the requested language, a fallback language version can be served instead. At the field level, if a field does not have content for the requested language, content for that field can be pulled from the fallback language version of the item.

Sitecore 8.1 now includes language fallback features such as field fallback and item fallback. Prior to Sitecore 8.1, third party modules like the Field Fallback module had to be installed to achieve language fallback functionality. This fallback can be defined at the site, template or item level. Each language can have one fallback language specified. Spanish can be set up to always fallback to English. Catalan, a language of northeastern Spain, can be set up to fallback to Spanish. If a user requested an item in Catalan, the language fallback would be Catalan &gt; Spanish &gt; English. Language fallback takes the path defined on each language.

There may be cases in which item language fallbacks should not take the same path. One example could be for websites targeted at a Canadian audience. In Canada, the two official languages are English and French. Quebec&rsquo;s official language is French whereas Ontario&rsquo;s official language is English. Many Canadians still speak their immigrant language at home while English and French is spoken outside the home. Someone with a mother tongue of Spanish might prefer to view content in Spanish. If they live in Quebec, they may prefer falling back to French if a Spanish version does not exist. In Ontario, they may prefer falling back to English.

One way we can achieve falling back to a language based on business rules is through the use of Glass.Mapper. There are other ways to achieve this, but I will be specifically focusing on [Glass.Mapper](http://www.glass.lu/Mapper/) in this blog post.


## Custom Fallback

In the Canadian example, item language fallback is based on the context of the user and other business rules. We need the ability to specify the fallback rules on a per-use basis. Something like the following would give us the most flexibility

*ItemFallbackController.cs*
``` csharp
public class GeneralController : GlassController
{
    public ActionResult Content()
    {
        var english = LanguageManager.GetLanguage("en");
        using (new LanguageItemFallback(english))
        {
            var model = GetContextItem<IContent>(inferType:true);
            return View(model);
        } 
    }
}
```

Each time we need a specific fallback language, we can create a new "LanguageItemFallback" context with the language(s) that fallback. If the current item does not exist in the context language, the languages specified in the context will be used as the fallback language.

We can create the LanguageItemFallback context with the following code:

*LanguageItemFallback.cs*
``` csharp
public class LanguageItemFallback : Switcher<LanguageItemFallbackState>
{
    private readonly VersionCountDisabler _versionCountDisabler;

    public LanguageItemFallback(params Language[] languages) : this(new LanguageItemFallbackState { Languages = languages } )
    {

    }

    public LanguageItemFallback(LanguageItemFallbackState state) : base(state)
    {
        _versionCountDisabler = new VersionCountDisabler();
    }

    public override void Dispose()
    {
        if (_versionCountDisabler != null)
            _versionCountDisabler.Dispose();
        base.Dispose();
    }
}
```

The context takes in a&nbsp;LanguageItemFallbackState object. This state holds an array of languages to fall through. Either the state object can be passed into the constructor or the array of languages. If you noticed, we are also creating a new VersionCountDisabler context. In order for Glass.Mapper to allow items with missing language versions through its pipeline, we need to disable the version count.

Here is the LanguageItemFallbackState used by the LanguageItemFallback:

*LanguageItemFallbackState.cs*
``` csharp
public class LanguageItemFallbackState
{
    public IEnumerable<Language> Languages { get; set; }
}
```

Now that we have&nbsp;the LanguageItemFallback context, we need to create a task to plug into the Glass.Mapper pipeline. The task takes the array of fallback languages from the LanguageItemFallback context and performs the fallback. It also makes sure the custom fallback does not interfere with the normal Sitecore processing, including the Page Editor.

*ResolveItemTask.cs*
``` csharp
public class ResolveItemTask : IConfigurationResolverTask
{
    public void Execute(ConfigurationResolverArgs args)
    {
        // Don't impact out of the box functionality for the page editor.
        if (Sitecore.Context.PageMode.IsPageEditorEditing)
            return;

        var context = (SitecoreTypeCreationContext)args.AbstractTypeCreationContext;

        // Item does not exist or language version already exists, bail out.
        if (context.Item == null || context.Item.Versions.Count > 0)
            return;

        var languageFallbackState = Switcher<LanguageItemFallbackState>.CurrentValue;
        if (languageFallbackState == null || languageFallbackState.Languages == null)
            return;

        var service = (SitecoreService)args.Service;
        var id = context.Item.ID;
        foreach (var language in languageFallbackState.Languages)
        {
            var item = service.Database.GetItem(id, language);

            // Bail out if no item.  In this case item should be null regardless of language. (you will always get a language version back regardless of translations)
            if (item == null)
                break;

            // No language version exists.  Check the next language
            if (item.Versions.Count == 0)
                continue;

            context.Item = item;
            break;
        }
    }
}
```

Now that we have our task created, we need to register it in the Glass.Mapper pipeline.

*GlassMapperScCustom.cs*
``` csharp
public static  class GlassMapperScCustom
{
  public static IDependencyResolver CreateResolver()
  {
    var config = new Glass.Mapper.Sc.Config();

    var resolver = new DependencyResolver(config);

    // Add ResolveItemTask at the beginning of the Configuration Resolver.
    resolver.ConfigurationResolverFactory.Insert(0, () => new ResolveItemTask());

    return resolver;
  }
}
```

## Usage

To use the custom language fallback, all we need to do is wrap any Glass.Mapper calls with the LanguageItemFallback context passing the list of languages. Here is an example of falling back from Catalan to Spanish to English where Catalan is the context language:

*Controller.cs*
``` csharp
public ActionResult Content()
{
    var spanish = LanguageManager.GetLanguage("es");
    var english = LanguageManager.GetLanguage("en");
    using (new LanguageItemFallback(spanish, english))
    {
        var model = GetContextItem<IContentGlass>();
        return View(model);
    } 
}
```

The array of languages can be changed based on the business requirements.

## Summary

As you can see, it is pretty easy to create a custom item language fallback based on business rules using Glass.Mapper. Good luck!