---
title: take 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 take 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: afee77c09e92d3cea8756ac42f1beefd20167725
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103995"
---
# <a name="take-operator"></a>take 运算符

最多返回指定数量的行。

```kusto
T | take 5
```

除非对源数据进行排序，否则无法保证返回哪些记录。

> [!NOTE]
> `take` 是一种在以交互方式浏览数据时查看小样本记录的简单、快速且高效的方法，但请注意，即使数据集没有变化，在多次执行时，也不能保证其结果的一致性。
> 即使查询返回的行数不由查询显式限制（未使用 `take` 运算符），Kusto 也会默认限制该数字。 如需更多详细信息，请参阅 [Kusto 查询限制](../concepts/querylimits.md)。

## <a name="syntax"></a>语法

`take` *NumberOfRows*
`limit` *NumberOfRows*

（`take` 和 `limit` 是同义词。）

## <a name="does-kusto-support-paging-of-query-results"></a>Kusto 是否支持对查询结果进行分页？

Kusto 不提供内置分页机制。

Kusto 是一种复杂的服务，它可不断优化其存储的数据，以便提供针对大型数据集的极佳的查询性能。 尽管分页对于资源有限的无状态客户端是一种有用的机制，但它是将负担转移到必须跟踪客户端状态信息的后端服务。 因此，后端服务的性能和可伸缩性受到严重限制。

对于分页支持，请实现以下功能之一：

* 将查询结果导出到外部存储，并以分页方式列出生成的数据。

* 编写中间层应用程序，该应用程序通过缓存 Kusto 查询的结果提供有状态分页 API。

## <a name="see-also"></a>请参阅

* [sort 运算符](sortoperator.md)
* [top 运算符](topoperator.md)
* [top-nested 运算符](topnestedoperator.md)