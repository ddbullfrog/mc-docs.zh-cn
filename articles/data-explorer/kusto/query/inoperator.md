---
title: in 和 notin 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 in 和 notin 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/18/2019
ms.date: 10/29/2020
ms.openlocfilehash: 37ca4827470bf7f63e0bf55155301398588dd94e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106010"
---
# <a name="in-and-in-operators"></a>in 和 !in 运算符

根据提供的值集筛选记录集。

```kusto
Table1 | where col in ('value1', 'value2')
```

> [!NOTE]
> * 向运算符添加“~”会使值的搜索不区分大小写：`x in~ (expression)` 或 `x !in~ (expression)`。
> * 在表格表达式中，会选择结果集的第一列。
> * 表达式列表最多可生成 `1,000,000` 个值。
> * 嵌套数组将平展为单个值列表。 例如，`x in (dynamic([1,[2,3]]))` 重命名为 `x in (1,2,3)`。
 
## <a name="syntax"></a>语法

### <a name="case-sensitive-syntax"></a>区分大小写的语法

*T* `|` `where` *col* `in` `(`*list of scalar expressions*`)`   
*T* `|` `where` *col* `in` `(`*tabular expression*`)`   
 
*T* `|` `where` *col* `!in` `(`*list of scalar expressions*`)`  
*T* `|` `where` *col* `!in` `(`*tabular expression*`)`   

### <a name="case-insensitive-syntax"></a>不区分大小写的语法

*T* `|` `where` *col* `in~` `(`*list of scalar expressions*`)`   
*T* `|` `where` *col* `in~` `(`*tabular expression*`)`   
 
*T* `|` `where` *col* `!in~` `(`*list of scalar expressions*`)`  
*T* `|` `where` *col* `!in~` `(`*tabular expression*`)`   

## <a name="arguments"></a>参数

* T：其记录待筛选的表格输入。
* col - 要筛选的列。
* list of expressions - 以逗号分隔的表格、标量或文本表达式的列表。
* tabular expression - 包含一组值的表格表达式。 如果表达式包含多个列，则使用第一列。

## <a name="returns"></a>返回

其谓词为 `true` 的 T 中的行。

## <a name="examples"></a>示例  

### <a name="use-in-operator"></a>使用“in”运算符

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where State in ("FLORIDA", "GEORGIA", "NEW YORK") 
| count
```

|计数|
|---|
|4775|  

### <a name="use-in-operator"></a>使用“in~”运算符  

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where State in~ ("Florida", "Georgia", "New York") 
| count
```

|计数|
|---|
|4775|  

### <a name="use-in-operator"></a>使用“!in”运算符

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents 
| where State !in ("FLORIDA", "GEORGIA", "NEW YORK") 
| count
```

|计数|
|---|
|54291|  


### <a name="use-dynamic-array"></a>使用动态数组

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let states = dynamic(['FLORIDA', 'ATLANTIC SOUTH', 'GEORGIA']);
StormEvents 
| where State in (states)
| count
```

|计数|
|---|
|3218|

### <a name="subquery"></a>子查询

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Using subquery
let Top_5_States = 
StormEvents
| summarize count() by State
| top 5 by count_; 
StormEvents 
| where State in (Top_5_States) 
| count
```

同一查询可以编写为：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
// Inline subquery 
StormEvents 
| where State in (
    ( StormEvents
    | summarize count() by State
    | top 5 by count_ )
) 
| count
```

|计数|
|---|
|14242|  

### <a name="top-with-other-example"></a>其他示例中的 Top

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
let Lightning_By_State = materialize(StormEvents | summarize lightning_events = countif(EventType == 'Lightning') by State);
let Top_5_States = Lightning_By_State | top 5 by lightning_events | project State; 
Lightning_By_State
| extend State = iif(State in (Top_5_States), State, "Other")
| summarize sum(lightning_events) by State 
```

| 状态     | sum_lightning_events |
|-----------|----------------------|
| ALABAMA   | 29                   |
| 威斯康星州 | 31                   |
| 德克萨斯     | 55                   |
| 佛罗里达州   | 85                   |
| 佐治亚州   | 106                  |
| 其他     | 415                  |

### <a name="use-a-static-list-returned-by-a-function"></a>使用函数返回的静态列表

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
StormEvents | where State in (InterestingStates()) | count

```

|计数|
|---|
|4775|  

函数定义。

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
.show function InterestingStates
```

|名称|parameters|正文|文件夹|DocString|
|---|---|---|---|---|
|InterestingStates|()|{ dynamic(["WASHINGTON", "FLORIDA", "GEORGIA", "NEW YORK"]) }
