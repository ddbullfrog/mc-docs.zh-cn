---
title: Azure Monitor 日志查询语言差异 | Microsoft Docs
description: Azure Monitor 使用的 Kusto 查询语言的参考信息。 包括特定于 Azure Monitor 的附加元素以及 Azure Monitor 日志查询中不支持的元素。
ms.subservice: logs
ms.topic: conceptual
author: Johnnytechn
ms.author: v-johya
ms.date: 11/02/2020
origin.date: 08/22/2019
ms.openlocfilehash: c24292a0b0ad15a38443b47ade984747e243db68
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328638"
---
# <a name="azure-monitor-log-query-language-differences"></a>Azure Monitor 日志查询语言差异

虽然 [Azure Monitor 中的日志](log-query-overview.md)是基于 [Azure 数据资源管理器](/data-explorer)构建的并使用相同的 [Kusto 查询语言](https://docs.microsoft.com/azure/kusto/query)，但该语言的版本确实有一些差异。 本文介绍了在数据资源管理器使用的语言版本与 Azure Monitor 日志查询使用的版本之间存在差异的元素。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="kql-elements-not-supported-in-azure-monitor"></a>Azure Monitor 中不支持的 KQL 元素
以下各部分介绍了 Azure Monitor 不支持的 Kusto 查询语言的元素。

### <a name="statements-not-supported-in-azure-monitor"></a>Azure Monitor 中不支持的语句

* [Alias](https://docs.microsoft.com/azure/kusto/query/aliasstatement)
* [Query parameters](https://docs.microsoft.com/azure/kusto/query/queryparametersstatement)

### <a name="functions-not-supported-in-azure-monitor"></a>Azure Monitor 中不支持的函数

* [cluster()](https://docs.microsoft.com/azure/kusto/query/clusterfunction)
* [cursor_after()](https://docs.microsoft.com/azure/kusto/query/cursorafterfunction)
* [cursor_before_or_at()](https://docs.microsoft.com/azure/kusto/query/cursorbeforeoratfunction)
* [cursor_current()、current_cursor()](https://docs.microsoft.com/azure/kusto/query/cursorcurrent)
* [database()](https://docs.microsoft.com/azure/kusto/query/databasefunction)
* [current_principal()](https://docs.microsoft.com/azure/kusto/query/current-principalfunction)
* [extent_id()](https://docs.microsoft.com/azure/kusto/query/extentidfunction)
* [extent_tags()](https://docs.microsoft.com/azure/kusto/query/extenttagsfunction)

### <a name="operators-not-supported-in-azure-monitor"></a>Azure Monitor 中不支持的运算符

* [跨群集加入](https://docs.microsoft.com/azure/kusto/query/joincrosscluster)

### <a name="plugins-not-supported-in-azure-monitor"></a>Azure Monitor 中不支持的插件

* [Python 插件](https://docs.microsoft.com/azure/kusto/query/pythonplugin)
* [sql_request 插件](https://docs.microsoft.com/azure/kusto/query/sqlrequestplugin)


## <a name="additional-operators-in-azure-monitor"></a>Azure Monitor 中的附加运算符
下列运算符支持特定的 Azure Monitor 功能并且在 Azure Monitor 外部不可用。

* [app()](app-expression.md)
* [workspace()](workspace-expression.md)

## <a name="next-steps"></a>后续步骤

- 获取[用于编写 Azure Monitor 日志查询的不同资源](/data-explorer/kusto/query/)的参考信息。
- 访问完整的 [Kusto 查询语言的参考文档](https://docs.microsoft.com/azure/kusto/query/)。

