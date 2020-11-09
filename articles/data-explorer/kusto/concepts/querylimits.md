---
title: 查询限制 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的查询限制。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/12/2020
ms.date: 10/29/2020
ms.openlocfilehash: 3ad0689407623a6d6533bbb464b3d22e81e041ac
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103554"
---
# <a name="query-limits"></a>查询限制

Kusto 是一个即席查询引擎，它承载着大型数据集，并尝试通过在内存中保留所有相关数据来满足查询。
存在一个固有的风险，即查询会无限地独占服务资源。 Kusto 以默认查询限制的形式提供了许多内置保护。 如果要考虑消除这些限制，请首先确定这样做实际上是否会带来任何价值。

## <a name="limit-on-query-concurrency"></a>有关查询并发性的限制

**查询并发性** 是指群集针对同时运行的查询数施加的限制。

* 查询并发限制的默认值取决于运行它的 SKU 群集，计算方式如下：`Cores-Per-Node x 10`。
  * 例如，对于在 D14v2 SKU 上设置的群集（其中每台计算机都有 16 个 vCore），默认查询并发限制为 `16 cores x10 = 160`。
* 可以通过配置[查询限制策略](../management/query-throttling-policy.md)来更改默认值。 
  * 可以在群集上并发运行的实际查询数取决于各种因素。 最主要的因素包括群集 SKU、群集的可用资源和查询模式。 可以根据对类似生产的查询模式执行的负载测试来配置查询限制策略。

## <a name="limit-on-result-set-size-result-truncation"></a>有关结果集大小的限制（结果截断）

**结果截断** 是针对查询返回的结果集默认设置的限制。 Kusto 将返回给客户端的记录数限制为 500,000，将这些记录的总数据大小限制为 64 MB。 当超出其中任一限制时，查询将失败并显示“部分查询失败”。 超出总数据大小将生成包含以下消息的异常：

```
The Kusto DataEngine has failed to execute a query: 'Query result set has exceeded the internal data size limit 67108864 (E_QUERY_RESULT_SET_TOO_LARGE).'
```

超出记录数则会失败，并会显示以下异常：

```
The Kusto DataEngine has failed to execute a query: 'Query result set has exceeded the internal record count limit 500000 (E_QUERY_RESULT_SET_TOO_LARGE).'
```

有许多策略可用于处理此错误。

* 将查询修改为仅返回令人感兴趣的数据，从而减小结果集的大小。 当初始的失败查询范围太广时，此策略非常有用。 例如，查询未抛弃不需要的数据列。
* 通过将查询后处理（例如聚合）切换到查询本身中来减小结果集的大小。 当查询输出被馈送到另一个处理系统并且该系统随后执行其他聚合时，此策略非常有用。
* 要从服务中导出大量数据时，请从查询切换到使用[数据导出](../management/data-export/index.md)。
* 使用下面列出的 `set` 语句或[客户端请求属性](../api/netfx/request-properties.md)中的标志来指示服务取消此查询限制。

用于减小查询生成的结果集大小的方法包括：

* 使用 [summarize 运算符](../query/summarizeoperator.md)对查询输出中的相似记录进行组合和聚合。 可以使用[任何聚合函数](../query/any-aggfunction.md)对某些列采样。
* 使用 [take 运算符](../query/takeoperator.md)对查询输出采样。
* 使用 [substring 函数](../query/substringfunction.md)来剪裁宽的自由文本列。
* 使用 [project 运算符](../query/projectoperator.md)从结果集中删除任何不感兴趣的列。

你可以通过使用 `notruncation` 请求选项来禁用结果截断。
建议你仍然实施某种形式的限制。

例如：

```kusto
set notruncation;
MyTable | take 1000000
```

还可以通过设置 `truncationmaxsize`（以字节为单位的最大数据大小，默认为 64 MB）和 `truncationmaxrecords`（最大记录数，默认为 500,000）的值，对结果截断进行更精确的控制。 例如，下面的查询将结果截断设置为在 1105 条记录或 1MB 时发生，以先超出的值为准。

```kusto
set truncationmaxsize=1048576;
set truncationmaxrecords=1105;
MyTable | where User=="UserId1"
```

去除结果截断限制意味着你打算将大量数据移出 Kusto。

你可以去除结果截断限制，以方便使用 `.export` 命令进行导出，或方便以后进行聚合。 如果选择稍后进行聚合，请考虑使用 Kusto 进行聚合。

Kusto 提供的许多客户端库能够以将“无限大的”结果流式传输给调用方的方式来处理这些结果。 请使用这些库中的一个，并将其配置为流式传输模式。 例如，使用 .NET Framework 客户端 (Microsoft.Azure.Kusto.Data)，并将连接字符串的 streaming 属性设置为 *true* ，或使用始终会流式传输结果的 *ExecuteQueryV2Async()* 调用。

默认情况下，结果截断不仅仅应用于返回给客户端的结果流。 默认情况下，它还应用于跨群集查询中一个群集向另一个群集发出的任何子查询，并具有类似的效果。

## <a name="limit-on-memory-per-iterator"></a>每个迭代器的内存限制

**每个结果集迭代器的最大内存** 是 Kusto 用于防范“失控”查询的另一个限制。 此限制由请求选项 `maxmemoryconsumptionperiterator` 表示，用于设置单个查询计划结果集迭代器可以容纳的内存量的上限。 此限制适用于本来就不能进行流式传输的特定迭代器，例如 `join`。下面是发生这种情况时将返回的一些错误消息：

```
The ClusterBy operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete E_RUNAWAY_QUERY.

The DemultiplexedResultSetCache operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The ExecuteAndCache operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The HashJoin operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The Sort operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The Summarize operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The TopNestedAggregator operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The TopNested operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).
```

默认情况下，此值设置为 5 GB。 最多可将此值增加到计算机的物理内存的一半：

```kusto
set maxmemoryconsumptionperiterator=68719476736;
MyTable | ...
```

在许多情况下，可以通过对数据集采样来避免超出此限制。 下面的两个查询展示了如何进行采样。 第一个是统计采样，它使用一个随机数生成器。 第二个是确定性采样，它是通过对数据集中的某个列（通常是某个 ID）进行哈希处理来执行的。

```kusto
T | where rand() < 0.1 | ...

T | where hash(UserId, 10) == 1 | ...
```

## <a name="limit-on-memory-per-node"></a>每个节点的内存限制

**每个节点每个查询的最大内存** 是用于防范“失控”查询的另一个限制。 此限制通过请求选项 `max_memory_consumption_per_query_per_node` 表示，用于设置可以在单个节点上用于特定查询的内存量的上限。

```kusto
set max_memory_consumption_per_query_per_node=68719476736;
MyTable | ...
```

## <a name="limit-on-accumulated-string-sets"></a>对累积的字符串集的限制

在各种查询操作中，Kusto 需要“收集”字符串值，并在内部将其置于缓冲区中，然后再开始生成结果。 这些累积的字符串集在大小和可以容纳的项目数方面都受限。 此外，每个单独的字符串不应超出特定的限制。
超出上述任一限制将导致出现以下错误之一：

```
Runaway query (E_RUNAWAY_QUERY). (message: 'Accumulated string array getting too large and exceeds the limit of ...GB (see https://aka.ms/kustoquerylimits)')

Runaway query (E_RUNAWAY_QUERY). (message: 'Accumulated string array getting too large and exceeds the maximum count of ..GB items (see http://aka.ms/kustoquerylimits)')
```

目前没有任何开关可用来增加最大字符串集大小。
解决方法是重新表述查询以减少必须缓冲的数据量。 可以抛弃不需要的列，不让 join 和 summarize 之类的运算符使用它们。 也可使用[随机执行查询](../query/shufflequery.md)策略。

## <a name="limit-execution-timeout"></a>限制执行超时

**服务器超时** 是应用于所有请求的服务端超时。
Kusto 中的多个点针对运行中的请求（查询和控制命令）强制实施了超时限制：

* 客户端库（如果使用）
* 接受请求的服务终结点
* 处理请求的服务引擎

默认情况下，查询超时设置为 4 分钟，控制命令超时设置为 10 分钟。 如果需要，可以增大该值（上限为 1 小时）。

* 如果你使用 Kusto.Explorer 进行查询，请使用“工具”&gt;“选项”_ &gt; _“连接”* &gt;“查询服务器超时”  。
* 以编程方式将 `servertimeout` 客户端请求属性设置为 `System.TimeSpan` 类型的值，最多为 1 小时。

**有关超时的说明**

* 在客户端，超时将从创建请求时开始算起，直到响应开始到达客户端为止。 在客户端上读回有效负载所花费的时间不计入超时。 这取决于调用方从流中拉取数据的速度。
* 此外，在客户端使用的实际超时值略高于用户所请求的服务器超时值。 此差异考虑了网络延迟。
* 若要自动使用允许的最大请求超时，请将客户端请求属性 `norequesttimeout` 设置为 `true`。

<!--
  Request timeout can also be set using a set statement, but we don't mention
  it here since it shouldn't be used in production scenarios.
-->

## <a name="limit-on-query-cpu-resource-usage"></a>对查询 CPU 资源使用量的限制

Kusto 允许你在运行查询时使用群集拥有的全部 CPU 资源。 如果有多个查询正在运行，它会尝试在查询之间进行公平的轮循。 对于即席查询，此方法可以实现最佳性能。
在其他时候，你可能希望限制用于特定查询的 CPU 资源。 例如，如果你运行“后台作业”，系统可能会容忍更高的延迟，以便向并发即席查询授予高优先级。

Kusto 支持在运行查询时指定两个[客户端请求属性](../api/netfx/request-properties.md)。 这两个属性是 query_fanout_threads_percent 和 query_fanout_nodes_percent。
这两个属性都是默认值为最大值 (100) 的整数，但对于特定查询，可以将其降低为某个其他值。 

第一个属性 query_fanout_threads_percent 控制线程使用的扇出因子。 如果为 100%，则群集将分配每个节点上的所有 CPU。 例如，在 Azure D14 节点上部署的群集上的 16 个 CPU。 当它为 50% 时，将使用一半的 CPU，依此类推。 数值将向上舍入为一整个 CPU，因此将其设置为 0 是安全的。 第二个属性 query_fanout_nodes_percent 控制每个子查询分发操作将使用群集中的多少个查询节点。 它以类似的方式工作。

## <a name="limit-on-query-complexity"></a>有关查询复杂性的限制

在查询执行期间，查询文本将转换为表示查询的关系运算符的树。
如果树深度超出了数千层的内部阈值，则查询会被视为太复杂，无法处理，因此会失败并显示错误代码。 此失败表示关系运算符树超出了其限制。
超出了限制是因为查询中有很长的二元运算符列表链接在一起。 例如：

```kusto
T 
| where Column == "value1" or 
        Column == "value2" or 
        .... or
        Column == "valueN"
```

对于此特定案例，请使用 [`in()`](../query/inoperator.md) 运算符来重新编写查询。

```kusto
T 
| where Column in ("value1", "value2".... "valueN")
```
