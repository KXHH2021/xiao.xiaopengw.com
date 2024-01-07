---
title: Kernel Memory 入门系列：生成并获取文档摘要
date: 2023-12-25 20:00:00
categories:
  - Kernel Memory
tags:
  - Summary流程
  - RAG
  - 文档预处理
description: Kernel Memory 入门系列：生成并获取文档摘要
cover: https://s2.loli.net/2024/01/07/8Woi3YV5MQwTj1m.png
---
![image.png](https://s2.loli.net/2024/01/07/8Woi3YV5MQwTj1m.png)

## Kernel Memory 入门系列：生成并获取文档摘要

前面在RAG和文档预处理的流程中，我们得到一个解决方案，可以让用户直接获取最终的问题答案。

但是实际的业务场景中，仍然存在一些基础的场景，不需要我们获取文档的所有详情的，而只是了解的文档的大概信息，得到文章整体的摘要或者总结，此时仍然可以使用Kernel Memory来处理。

## 生成摘要

我们依然使用Kernel Memory的文件导入方法，不过此时不需要指定默认的处理流程，而只需要指定Summary流程即可。
```undefined
await memory.ImportDocumentAsync(new Document("doc1")
        .AddFile("file4-SK-Readme.pdf")
        .AddFile("file5-NASA-news.pdf"),
    steps: Constants.PipelineOnlySummary);
```
其中PipelineOnlySummary 包含了一下步骤：

- extract
- summarize
- gen_embeddings
- save_records

  
相比较默认的流程，仅是将partition变更为了summarize, 但是实际存储的记录将不再是源文档的分片，而是经过LLM总结之后的内容摘要。

## 获取摘要

获取的摘要的方法更加直接，使用SearchSummariesAsync方法，通过文档过滤条件过滤需要获取文档摘要即可。
```undefined
// Fetch the list of summaries. The API returns one summary for each file.
var results = await memory.SearchSummariesAsync(filter: MemoryFilters.ByDocument("doc1"));
 
// Print the summaries!
foreach (var result in results)
{
    Console.WriteLine($"== {result.SourceName} summary ==\n{result.Partitions.First().Text}\n");
}
```
## 检索生成数据
摘要的生成和检索在Kernel Memory中实际是数据类型标记和自定义筛选筛选的过程。

在生成摘要的过程中，将摘要内容作为生成内容，通过添加__synth:summary标记进行存储，筛选的时候也是类似。文档的标记和筛选，将会在后续【文档管理】中的详细讲解。

而摘要的检索的过程SearchSummariesAsync实际上也是调用SearchSyntheticsAsync过程，指定了__synth:summary标记的段落进行检索。

同理，生成摘要的过程也可以进行自定义的过程，例如文章分类，关键词提取，实体提取，题图生成等任何的文章处理流程。后续也会详细介绍【自定义流程】的处理。

**参考**

- Summarizing documents:

**https://github.com/microsoft/kernel-memory/tree/main/examples/106-dotnet-retrieve-synthetics**

- kernel-memory/service/Abstractions/KernelMemoryExtensions.cs
```undefined
﻿// Copyright (c) Microsoft. All rights reserved.

using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace Microsoft.KernelMemory;

/// <summary>
/// Kernel Memory API extensions
/// </summary>
public static class KernelMemoryExtensions
{
    /// <summary>
    /// Return a list of synthetic memories of the specified type
    /// </summary>
    /// <param name="memory">Memory instance</param>
    /// <param name="syntheticType">Type of synthetic data to return</param>
    /// <param name="index">Optional name of the index where to search</param>
    /// <param name="filter">Filter to match</param>
    /// <param name="filters">Filters to match (using inclusive OR logic). If 'filter' is provided too, the value is merged into this list.</param>
    /// <param name="cancellationToken">Async task cancellation token</param>
    /// <returns>List of search results</returns>
    public static async Task<List<Citation>> SearchSyntheticsAsync(
        this IKernelMemory memory,
        string syntheticType,
        string? index = null,
        MemoryFilter? filter = null,
        ICollection<MemoryFilter>? filters = null,
        CancellationToken cancellationToken = default)
    {
        if (filters == null)
        {
            filters = new List<MemoryFilter>();
            if (filter == null) { filters.Add(new MemoryFilter()); }
        }

        if (filter != null)
        {
            filters.Add(filter);
        }

        foreach (var x in filters)
        {
            x.ByTag(Constants.ReservedSyntheticTypeTag, syntheticType);
        }

        SearchResult searchResult = await memory.SearchAsync(
            query: "",
            index: index,
            filters: filters,
            cancellationToken: cancellationToken).ConfigureAwait(false);

        return searchResult.Results;
    }

    /// <summary>
    /// Return a list of summaries matching the given filters
    /// </summary>
    /// <param name="memory">Memory instance</param>
    /// <param name="index">Optional name of the index where to search</param>
    /// <param name="filter">Filter to match</param>
    /// <param name="filters">Filters to match (using inclusive OR logic). If 'filter' is provided too, the value is merged into this list.</param>
    /// <param name="cancellationToken">Async task cancellation token</param>
    /// <returns>List of search results</returns>
    public static Task<List<Citation>> SearchSummariesAsync(
        this IKernelMemory memory,
        string? index = null,
        MemoryFilter? filter = null,
        ICollection<MemoryFilter>? filters = null,
        CancellationToken cancellationToken = default)
    {
        return SearchSyntheticsAsync(memory, Constants.TagsSyntheticSummary, index, filter, filters, cancellationToken);
    }
}
```
