---
title: 跨数据库和跨群集查询 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的跨数据库和跨群集查询。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 962f7bd69a7ce9c00544222d8530c184aa6696c5
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104215"
---
# <a name="cross-database-and-cross-cluster-queries"></a>跨数据库和跨群集查询

::: zone pivot="azuredataexplorer"

每个 Kusto 查询都在当前群集的上下文中以及在默认数据库中运行。
* 在 [Kusto 资源管理器](../tools/kusto-explorer.md)中，默认数据库是在[“连接”面板](../tools/kusto-explorer.md#connections-panel)中选择的数据库，当前群集是包含该数据库的连接。
* 当使用[客户端库](../api/netfx/about-kusto-data.md)时，当前群集和默认数据库由[连接字符串](../api/connection-strings/kusto.md)的 `Data Source` 和 `Initial Catalog` 属性分别指定。

## <a name="queries"></a>查询
若要访问除默认值以外的任何其他数据库中的表，必须使用“限定的名称”语法。

访问当前群集中的数据库。

```kusto
database("<database name>").<table name>
```

远程群集中的数据库。
```kusto
cluster("<cluster name>").database("<database name>").<table name>
```

“数据库名称”区分大小写

“集群名称”不区分大小写且可以是以下形式之一：
   * 格式标准的 URL，例如 `http://contoso.kusto.chinacloudapi.cn:1234/`。 仅支持 HTTP 和 HTTPS 方案。
   * 完全限定的域名 (FQDN)，例如 `contoso.kusto.chinacloudapi.cn`。 该字符串等效于 `https://`**`contoso.kusto.chinacloudapi.cn`** `:443/`。
   * 短名称（不包含域部分的主机名 [和区域]），例如 `contoso` 或 `contoso.chinaeast2`。 这些字符串转译为 `https://`**`contoso`** `.kusto.chinacloudapi.cn:443/` 和 `https://`**`contoso.chinaeast2`** `.kusto.chinacloudapi.cn:443/`。

> [!NOTE]
> 跨数据库访问需遵守常规权限检查。
> 若要执行查询，必须对默认数据库和查询中引用的每个其他数据库（在当前和远程群集中）具有读取权限。

“限定的名称”可以在任何可使用表名的上下文中使用。

以下均为有效选项。

```kusto
database("OtherDb").Table | where ...

union Table1, cluster("OtherCluster").database("OtherDb").Table2 | project ...

database("OtherDb1").Table1 | join cluster("OtherCluster").database("OtherDb2").Table2 on Key | join Table3 on Key | extend ...
```

“限定的名称”显示为 [联合运算符](./unionoperator.md) 的操作数时，可以使用通配符来指定多个表和多个数据库。 群集名称中不允许使用通配符。

```kusto
union withsource=TableName *, database("OtherDb*").*Table, cluster("OtherCluster").database("*").*
```

> [!NOTE]
> * 默认数据库的名称也是潜在匹配项，因此数据库 ("&#42;") 指定所有数据库（包括默认数据库）的所有表。
> * 要详细了解架构更改如何影响跨群集查询，请参阅[跨群集查询和架构更改](../concepts/crossclusterandschemachanges.md)

## <a name="access-restriction"></a>访问限制

限定的名称或模式也可以包含在[限制访问](./restrictstatement.md)语句中，不允许在群集名称中使用通配符。

```kusto
restrict access to (my*, database("MyOther*").*, cluster("OtherCluster").database("my2*").*);
```

以上将限制对以下实体的查询访问：

* 从默认数据库中的“my...”开始的任何实体名称。 
* 当前群集的所有名为“MyOther...”的数据库中的任何表。
* “OtherCluster.kusto.chinacloudapi.cn”集群中所有名为“my2...”的数据库中的任何表 。

## <a name="functions-and-views"></a>函数和视图

函数和视图（永久的和内联创建的）可以跨数据库和群集边界引用表。 以下代码是有效的。

```kusto
let MyView = Table1 join database("OtherDb").Table2 on Key | join cluster("OtherCluster").database("SomeDb").Table3 on Key;
MyView | where ...
```

可以从同一集群中的另一个数据库访问永久函数和视图。

`OtherDb` 中的表格函数（视图）。

```kusto
.create function MyView(v:string) { Table1 | where Column1 has v ...  }  
```

`OtherDb` 中的标量函数。

```kusto
.create function MyCalc(a:double, b:double, c:double) { (a + b) / c }  
```

在默认数据库中。

```kusto
database("OtherDb").MyView("exception") | extend CalCol=database("OtherDb").MyCalc(Col1, Col2, Col3) | limit 10
```

## <a name="limitations-of-cross-cluster-function-calls"></a>跨群集函数调用的限制

可以跨群集引用的表格函数或视图。 以下限制适用：

* 远程函数必须返回表格架构。 标量函数仅能在同一个集群中访问。
* 远程函数仅能接受标量参数。 获取一个或多个表参数的函数只能在同一群集中访问。
* 远程函数的架构必须是已知的，并且参数不变。 有关详细信息，请参阅[跨群集查询和架构更改](../concepts/crossclusterandschemachanges.md)。

以下跨群集调用有效。

```kusto
cluster("OtherCluster").database("SomeDb").MyView("exception") | count
```

以下查询调用远程标量函数 `MyCalc`。
此调用违反规则 #1，因此无效。

```kusto
MyTable | extend CalCol=cluster("OtherCluster").database("OtherDb").MyCalc(Col1, Col2, Col3) | limit 10
```

以下查询调用远程函数 `MyCalc` 并提供表格参数。
此调用违反规则 #2，因此无效。

```kusto
cluster("OtherCluster").database("OtherDb").MyCalc(datatable(x:string, y:string)["x","y"] )
```

以下查询调用远程函数 `SomeTable`，该函数具有基于参数 `tablename` 的变量架构输出。
此调用违反规则 #3，因此无效。

`OtherDb` 中的表格函数。

```kusto
.create function SomeTable(tablename:string) { table(tablename)  }  
```

在默认数据库中。

```kusto
cluster("OtherCluster").database("OtherDb").SomeTable("MyTable")
```

以下查询调用远程函数 `GetDataPivot`，该函数具有基于数据（[pivot() plugin](pivotplugin.md) 具有动态输出）的可变架构输出。
此调用违反规则 #3，因此无效。

`OtherDb` 中的表格函数。

```kusto
.create function GetDataPivot() { T | evaluate pivot(PivotColumn) }  
```

默认数据库中的表格函数。

```kusto
cluster("OtherCluster").database("OtherDb").GetDataPivot()
```

## <a name="displaying-data"></a>显示数据

返回的记录数将隐式限制将数据返回到客户端的语句，即使没有特定使用 `take` 运算符。 若要提升此限制，请使用 `notruncation` 客户端请求选项。

要以图形形式显示数据，请使用 [呈现运算符](renderoperator.md)。

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 中不支持跨数据库查询和跨群集查询。

::: zone-end
