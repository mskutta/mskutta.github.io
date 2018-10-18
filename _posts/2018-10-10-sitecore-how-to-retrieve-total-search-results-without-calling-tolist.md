---
layout: post
title: "Sitecore How-To: Retrieve total search results without calling .ToList() in Sitecore"
date:   2018-10-10 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Alex Hawley and Mike Skutta
excerpt: Calling .ToList() on an entire Lucene result set is not feasible from a performance perspective when there are thousands of potential results getting returned. However, getting the total number of results may be required on a professional search page for instance. "Total Results = 1,334" when only 10 bios are displayed at a time is a common use case, and calling .ToList() to get the total number of results is not optimal. ".GetResults()" (under the Sitecore.ContentSearch.Linq namespace) allows you to retrieve both the filtered results and the total number of results without serializing the entire result set.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Calling .ToList() on an entire Lucene result set is not feasible from a performance perspective when there are thousands of potential results getting returned. However, getting the total number of results may be required on a professional search page for instance. "Total Results: 1,334" when only 10 bios are displayed at a time is a common use case, and calling .ToList() to get the total number of results is not optimal. ".GetResults()" (under the Sitecore.ContentSearch.Linq namespace) allows you to retrieve both the filtered results and the total number of results without serializing the entire result set.

## Step-by-step guide

1. Include a using for: Sitecore.ContentSearch.Linq
1. Create your Sitecore Lucene query (e.g. via PredicateBuilder)
1. Create a Lucene search context for executing your query
1. Call .GetResults() on the IQueryable<T> that has been built
1. The returned object SearchResults<T> will contain a property for "TotalSearchResults" for a count of all results as well as "Hits" which will contain result items, even if the result items are a subset of the total number of results.

```c#
string filter = serviceId.GetSitecoreParsedId();
 
// Build query
var query = PredicateBuilder.True<ProfessionalSearchResultItem>();
query = query.And(p => p.TemplateName == "Professional"
                  && p.LatestVersion == "1"
                  && p.LanguageFallBack.Contains(currentLanguage)
                  && p.RelatedServices.Contains(filter));
 
 
// Apply sorting and skip/take
query = query.OrderBy(p => p.Name).Skip(10).Take(10);
 
// Create search context for executing query
using (var context = QueryHelper.SearchIndex.CreateSearchContext())
{
    // Executes Lucene query via .GetResults()
    SearchResults<ProfessionalSearchResultItem> results = context.GetQueryable<ProfessionalSearchResultItem>().Where(query).GetResults();
     
    // Contains total count of results, regardless of skip and take amounts
    int totalCount = results.TotalSearchResults;
     
    // Contains 10 professional search result items after skipping 10
    List<ProfessionalSearchResultItem> resultSet = results.Hits.Select(p => p.Document).ToList();
}
```