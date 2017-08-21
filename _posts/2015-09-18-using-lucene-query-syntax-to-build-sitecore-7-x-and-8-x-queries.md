---
layout: post
title: "Using Lucene Query Syntax to build Sitecore 7.x and 8.x Queries"
date:   2015-09-24 00:00:00 -0500
categories: sitecore
tags: sitecore lucene-query-syntax
author: Mike Skutta
excerpt: Sitecore 7 introduced many improvements to Lucene Search. Among those improvements was the added LINQ support for Lucene queries. Developers can now use LINQ-based queries to retrieve data from the Lucene indexes. Prior to Sitecore 7, developers generally had 2 ways to build the query. The first way was to build up a query using the Field Query Object Model. This provided an object model to create complex queries. A second way was to form a string-based representation of a query using the Lucene Query Parser to interpret the string.
---

* content
{:toc}

## Overview

Sitecore 7 introduced many improvements to Lucene Search. Among those improvements was the added LINQ support for Lucene queries. Developers can now use LINQ-based queries to retrieve data from the Lucene indexes. Prior to Sitecore 7, developers generally had 2 ways to build the query. The first way was to build up a query using the Field Query Object Model. This provided an object model to create complex queries. A second way was to form a string-based representation of a query using the Lucene Query Parser to interpret the string.

We built several sites on Sitecore 6.6 using the string-based queries. It is now time to upgrade those sites to newer versions of Sitecore to take advantage of the new features and improvements. In some cases we want to continue to use the string based queries so we don’t have to rework them into the new LINQ syntax, simply to save time.

For Sitecore 6.6, we abstracted the logic of executing the queries into helper methods. The details of how the query was executed and how the results were returned was contained within the helper methods. This makes it easy for us to swap the logic in the helper methods with the new Sitecore 7+ support.

# Sitecore 7+

The following example shows how a typical Lucene query may be executed in Sitecore 7+ using LINQ.

``` csharp
var index = ContentSearchManager.GetIndex("sitecore_web_index");
using (var context = index.CreateSearchContext())
{
    var results = context.GetQueryable<SearchResultItem>()
            .Where(item => item.TemplateName == "Sample Item")
            .Where(item => item.Language == "en")
            .Take(10)
            .ToList();
}
```

The string representation of the above LINQ query is: **_language:en AND _templatename:“sample item”**.

How can we use the string representation of the query instead?  Again, we are looking to support backward compatibility of older implementations.  Here is a link to the syntax we need to support: [https://lucene.apache.org/core/2_9_4/queryparsersyntax.html](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html).

When performing a search in Sitecore 7+, you first get a reference to the index, then you create a search context from the index.  The search context is of type **IProviderSearchContext**.  When using Lucene, the underlying type is **LuceneSearchContext**.  The LuceneSearchContext has the ability to parse string queries into a **Lucene.Net.Search.Query**.  The LuceneSearchContext also exposes the underlying **Lucene.Net.Search.Searcher**.  The Searcher can take in the parsed Query, along with the number of items to return and optional sort fields, returning the matching documents IDs.  The IDs can then be used to retrieve the actual documents from the Lucene index.  From the documents, we can use the **DefaultLuceneDocumentTypeManager** to create **SearchResultItems**.

> Note: In this implementation, it is more efficient to perform the paging and sorting prior to performing the mapping. Performing the paging and sorting via LINQ afterwards is not as efficient.

This logic can be formed into an extension method for ease of use:

``` csharp
public static IEnumerable<TSearchResultItem> Query<TSearchResultItem>(this IProviderSearchContext context, string query, int skip = 0, int take = 0, SortField[] sortFields = null)
    where TSearchResultItem : SearchResultItem
{
    var luceneSearchContext = context as LuceneSearchContext;
    if (luceneSearchContext == null)
        throw new ApplicationException("Lucene Search Context does not exist.");

    skip = (skip > 0) ? skip : 0;
    take = (take > 0) ? take : int.MaxValue;

    // Parse the string based query
    var parsedQuery = luceneSearchContext.Parse(query);

    // Log the Lucene query 
    SearchLog.Log.Info(string.Concat("ExecuteQueryAgainstLucene : ", parsedQuery));

    // Perform the search, add sorting if applicable
    var searcher = luceneSearchContext.Searcher;
    var topDocs = (sortFields != null && sortFields.Length > 0) ?
                        searcher.Search(parsedQuery, null, skip + take, new Sort(sortFields)) :
                        searcher.Search(parsedQuery, null, skip + take);

    // Convert the results to hits.
    var hits = topDocs.ScoreDocs.Select(x => new SearchHit<Document>(x.Score, searcher.Doc(x.Doc))).ToList();

    // When paging results, get the correct page of groups
    var pagedHits = hits.Skip(skip).Take(take);

    // Convert hits to SearchResultItem and return results.
    var mapper = new DefaultLuceneDocumentTypeMapper();
    mapper.Initialize(context.Index);
    return pagedHits.Select(hit => mapper.MapToType<TSearchResultItem>(hit.Document, null, null, SearchSecurityOptions.DisableSecurityCheck));
}
```

An example of how to use the extension method is as follows:

``` csharp
var index = ContentSearchManager.GetIndex("sitecore_web_index");
using (var context = index.CreateSearchContext())
{
    var results = context.Query<SearchResultItem>(@"_language:en AND _templatename:""sample item""")
            .Take(10)
            .ToList();
}
```

As you can see, it was relatively straight forward to add Lucene query string syntax support. We were able to use a variation of the above logic to support our legacy Lucene Search Strings.

> Note: Lucene is queried as part of the call to Query method. Any LINQ expressions made on the results of the Query method call are done after Lucene was queried. Using Sitecore’s out of the box Lucene LINQ support builds the query as LINQ expressions are added. The query against Lucene is not actually executed till the results are enumerated.

I hope you find this useful.