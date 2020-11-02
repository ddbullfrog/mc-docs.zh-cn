---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/26/2020
title: 图形帧用户指南 - Scala - Azure Databricks
description: 了解如何在 Azure Databricks 中通过 Scala 使用图形帧执行图形分析。
ms.openlocfilehash: d83ff79262492d0d9d9899ce0b8de9dcab45fb46
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472892"
---
# <a name="graphframes-user-guide---scala"></a>GraphFrames 用户指南 - Scala

本文演示了[图形帧用户指南](https://graphframes.github.io/graphframes/docs/_site/user-guide.html)中的示例。

```scala
import org.apache.spark.sql._
import org.apache.spark.sql.functions._
import org.graphframes._
```

## <a name="creating-graphframes"></a>创建图形帧

可以从顶点和边数据帧创建图形帧。

* 顶点数据帧：顶点数据帧应当包含一个名为 `id` 的特殊列，它指定图形中每个顶点的唯一 ID。
* 边数据帧：边数据帧应包含两个特殊列：`src`（边的源顶点 ID）和 `dst`（边的目标顶点 ID）。

这两个数据帧都可以包含任意其他列。 这些列可以表示顶点和边属性。

创建顶点和边

```scala
// Vertex DataFrame
val v = sqlContext.createDataFrame(List(
  ("a", "Alice", 34),
  ("b", "Bob", 36),
  ("c", "Charlie", 30),
  ("d", "David", 29),
  ("e", "Esther", 32),
  ("f", "Fanny", 36),
  ("g", "Gabby", 60)
)).toDF("id", "name", "age")
// Edge DataFrame
val e = sqlContext.createDataFrame(List(
  ("a", "b", "friend"),
  ("b", "c", "follow"),
  ("c", "b", "follow"),
  ("f", "c", "follow"),
  ("e", "f", "follow"),
  ("e", "d", "friend"),
  ("d", "a", "friend"),
  ("a", "e", "friend")
)).toDF("src", "dst", "relationship")
```

让我们从这些顶点和这些边创建图形：

```scala
val g = GraphFrame(v, e)
```

```scala
// This example graph also comes with the GraphFrames package.
// val g = examples.Graphs.friends
```

## <a name="basic-graph-and-dataframe-queries"></a>基本图形和数据帧查询

图形帧提供简单的图形查询，例如节点度。

另外，由于图形帧将图形表示为成对的顶点和边数据帧，因此很容易直接对顶点和边数据帧进行强大的查询。 那些数据帧在图形帧中作为顶点字段和边字段提供。

```scala
display(g.vertices)
```

```scala
display(g.edges)
```

顶点的传入度：

```scala
display(g.inDegrees)
```

顶点的传出度：

```scala
display(g.outDegrees)
```

顶点的度：

```scala
display(g.degrees)
```

可以直接对顶点数据帧运行查询。 例如，我们可以在图形中找到最年轻人员的年龄：

```scala
val youngest = g.vertices.groupBy().min("age")
display(youngest)
```

同样，你可以对边数据帧运行查询。 例如，让我们计算一下图形中的“关注”关系的数目：

```scala
val numFollows = g.edges.filter("relationship = 'follow'").count()
```

## <a name="motif-finding"></a>装饰图形查找结果

使用装饰图形构建涉及边和顶点的更复杂的关系。 下面的单元将查找其间的两个方向上都有边的顶点对。 结果是一个数据帧，其中的列名为装饰图形键。

有关 API 的更多详细信息，请查看[图形帧用户指南](https://graphframes.github.io/graphframes/docs/_site/user-guide.html#motif-finding)。

```scala
// Search for pairs of vertices with edges in both directions between them.
val motifs = g.find("(a)-[e]->(b); (b)-[e2]->(a)")
display(motifs)
```

由于结果是一个数据帧，因此你可以在装饰图形的基础上构建更复杂的查询。 让我们找出一个人年龄大于 30 岁的所有相互关系：

```scala
val filtered = motifs.filter("b.age > 30")
display(filtered)
```

### <a name="stateful-queries"></a>有状态查询

大多数装饰图形查询是无状态的，并且简单明了，如以上示例所示。 后续示例演示了更复杂的查询，这些查询在装饰图形中沿着某个路径携带状态。 通过将图形帧装饰图形查找结果与针对结果的筛选器组合使用来表示这些查询，其中，筛选器使用序列运算来构造一系列的数据帧列。

例如，假设你要标识一个由 4 个顶点组成的链，该链具有由一系列函数定义的某些属性。 也就是说，在由 4 个顶点组成的链 `a->b->c->d` 中，标识与此复杂筛选器匹配的链子集：

* 在路径上初始化状态。
* 基于顶点 a 更新状态。
* 基于顶点 b 更新状态。
* 为 c 和 d 更新状态。
* 如果最终状态与某个条件匹配，则筛选器会接受该链。

下面的代码片段演示了此过程，其中，我们确定了由 4 个顶点构成的链，使得 3 条边中至少有 2 条是“朋友”关系。 在此示例中，状态是“朋友”边的当前计数；一般情况下，它可以是任何数据帧列。

```scala
// Find chains of 4 vertices.
val chain4 = g.find("(a)-[ab]->(b); (b)-[bc]->(c); (c)-[cd]->(d)")

// Query on sequence, with state (cnt)
//  (a) Define method for updating state given the next element of the motif.
def sumFriends(cnt: Column, relationship: Column): Column = {
  when(relationship === "friend", cnt + 1).otherwise(cnt)
}
//  (b) Use sequence operation to apply method to sequence of elements in motif.
//      In this case, the elements are the 3 edges.
val condition = Seq("ab", "bc", "cd").
  foldLeft(lit(0))((cnt, e) => sumFriends(cnt, col(e)("relationship")))
//  (c) Apply filter to DataFrame.
val chainWith2Friends2 = chain4.where(condition >= 2)
display(chainWith2Friends2)
```

### <a name="subgraphs"></a>子图形

图形帧提供了用于通过对边和顶点进行筛选来构建子图形的 API。 这些筛选器可以组合在一起。 例如，下面的子图形仅包含是朋友且年龄超过 30 的人员。

```scala
// Select subgraph of users older than 30, and edges of type "friend"
val g2 = g
  .filterEdges("relationship = 'friend'")
  .filterVertices("age > 30")
  .dropIsolatedVertices()
```

#### <a name="complex-triplet-filters"></a>复杂的三元组筛选器

下面的示例展示了如何基于对某个边及其“src”和“dst”顶点进行操作的三元组筛选器来选择子图形。 使用更复杂的装饰图形将此示例扩展为超过三元组的操作非常简单。

```scala
// Select subgraph based on edges "e" of type "follow"
// pointing from a younger user "a" to an older user "b".
val paths = g.find("(a)-[e]->(b)")
  .filter("e.relationship = 'follow'")
  .filter("a.age < b.age")
// "paths" contains vertex info. Extract the edges.
val e2 = paths.select("e.src", "e.dst", "e.relationship")
// In Spark 1.5+, the user may simplify this call:
//  val e2 = paths.select("e.*")

// Construct the subgraph
val g2 = GraphFrame(g.vertices, e2)
```

```scala
display(g2.vertices)
```

```scala
display(g2.edges)
```

## <a name="standard-graph-algorithms"></a>标准图形算法

本部分介绍了内置于图形帧中的标准图形算法。

### <a name="breadth-first-search-bfs"></a>广度优先搜索 (BFS)

从“Esther”中搜索年龄 < 32 的用户。

```scala
val paths: DataFrame = g.bfs.fromExpr("name = 'Esther'").toExpr("age < 32").run()
display(paths)
```

搜索还可以限制边筛选器和最大路径长度。

```scala
val filteredPaths = g.bfs.fromExpr("name = 'Esther'").toExpr("age < 32")
  .edgeFilter("relationship != 'friend'")
  .maxPathLength(3)
  .run()
display(filteredPaths)
```

### <a name="connected-components"></a>连接的组件

计算每个顶点的已连接组件成员，并返回一个图形，其中的每个顶点均分配有一个组件 ID。

```scala
val result = g.connectedComponents.run() // doesn't work on Spark 1.4
display(result)
```

### <a name="strongly-connected-components"></a>强连接的组件

计算每个顶点的强连接组件 (SCC)，并返回一个图形，其中的每个顶点都分配到包含该顶点的 SCC。

```scala
val result = g.stronglyConnectedComponents.maxIter(10).run()
display(result.orderBy("component"))
```

### <a name="label-propagation"></a>标签传播

运行静态标签传播算法，以检测网络中的社区。

网络中的每个节点最初都已分配给自己的社区。 在每一个超级步骤中，节点向所有邻居发送其社区隶属关系，并将其状态更新为传入消息的模式社区隶属关系。

LPA 是一个用于图形的标准社区检测算法。 这在计算方面成本较低，但具有以下特点：(1) 不能保证收敛；(2) 最终可能会得到一些微不足道的解决方案（所有节点都标识为一个社区）。

```scala
val result = g.labelPropagation.maxIter(5).run()
display(result.orderBy("label"))
```

### <a name="pagerank"></a>PageRank

基于连接确定图形中的重要顶点。

```scala
// Run PageRank until convergence to tolerance "tol".
val results = g.pageRank.resetProbability(0.15).tol(0.01).run()
display(results.vertices)
```

```scala
display(results.edges)
```

```scala
// Run PageRank for a fixed number of iterations.
val results2 = g.pageRank.resetProbability(0.15).maxIter(10).run()
display(results2.vertices)
```

```scala
// Run PageRank personalized for vertex "a"
val results3 = g.pageRank.resetProbability(0.15).maxIter(10).sourceId("a").run()
display(results3.vertices)
```

### <a name="shortest-paths"></a>最短的路径

计算给定路标顶点集的最短路径，其中的路标是由顶点 ID 指定的。

```scala
val paths = g.shortestPaths.landmarks(Seq("a", "d")).run()
display(paths)
```

### <a name="triangle-counting"></a>三角形计数

计算通过每个顶点的三角形的数量。

```scala
import org.graphframes.examples
val g: GraphFrame = examples.Graphs.friends  // get example graph

val results = g.triangleCount.run()
results.select("id", "count").show()
```