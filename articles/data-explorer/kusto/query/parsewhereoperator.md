---
title: parse-where 运算符 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 parse-where 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/12/2020
ms.date: 10/29/2020
ms.openlocfilehash: 139b2942f845d8bc4df02ec514ceb34fdeede9a2
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105035"
---
# <a name="parse-where-operator"></a>parse-where 运算符

计算字符串表达式并将其值分析为一个或多个计算列。 得到的结果只有成功分析的字符串。 

请参阅 [parse 运算符](parseoperator.md)，该运算符对未成功分析的字符串生成 null。

```kusto
T | parse-where Text with "ActivityName=" name ", ActivityType=" type
```

## <a name="syntax"></a>语法

*T* `| parse-where` [`kind=regex` [`flags=regex_flags`] |`simple`] *Expression* `with` `*` ( *StringConstant* *ColumnName* [`:` *ColumnType* ]) `*`...

## <a name="arguments"></a>参数

* *T* ：输入表。

* *kind* ： 

    * *simple* （默认值）：StringConstant 是一个正则字符串值，匹配为严格匹配。 所有字符串分隔符都应出现在分析的字符串中，并且所有扩展列都必须与所需类型匹配。
        
    * *regex* ：StringConstant 可能是正则表达式，匹配为严格匹配。 所有字符串分隔符都应出现在分析的字符串中，并且所有扩展列都必须与所需类型匹配。 对于此模式，字符串分隔符可以是正则表达式。
    
    * *flags* ：正则表达式模式中要使用的标志：`U` (Ungreedy)、`m`（多行模式）、`s`（匹配新行 `\n`）、`i`（不区分大小写），可在 [RE2 标志](re2.md)中找到更多标志。
        
* *表达式* ：计算结果为字符串的表达式。

* *columnName* ：列名称，分配给从字符串表达式中提取的值。 
  
* *ColumnType：* 应为可选的标量类型，指示要将值转换为的类型。 默认值为字符串类型。

## <a name="returns"></a>返回

输入表，根据提供给运算符的列的列表进行扩展。

> [!Note] 
> 只有成功分析的字符串才会出现在输出中。 与模式不匹配的字符串将被筛选掉。

**提示**

* `parse-where` 以与 [parse](parseoperator.md) 相同的方式分析字符串，并筛选出未成功分析的字符串。

* 如果还希望删除或重命名某些列，请使用 [project](projectoperator.md)。

* 在模式中使用 * 可跳过垃圾值。 此值不能在字符串列后使用。

* 除 StringConstant 外，分析模式还可以 ColumnName 开头 。 

* 如果分析的表达式不是字符串类型，会将其转换为字符串类型。

* 如果使用正则表达式模式，则可以添加正则表达式标志来控制分析中使用的整个正则表达式。

* 在正则表达式模式下，分析会将模式转换为正则表达式，并使用 [RE2 语法](re2.md)以便使用在内部处理的被捕获的组（有编号）进行匹配。
  
  例如，以下分析语句：
  
    ```kusto
    parse-where kind=regex Col with * <regex1> var1:string <regex2> var2:long
    ```

    分析过程在内部生成的正则表达式将为 `.*?<regex1>(.*?)<regex2>(\-\d+)`。
        
    - `*` 转换为 `.*?`。
        
    - `string` 转换为 `.*?`。
        
    - `long` 转换为 `\-\d+`。

## <a name="examples"></a>示例

`parse-where` 运算符提供了一种简单的方法，可通过对同一 `string` 表达式使用多个 `extract` 应用程序来 `extend` 某个表。 当表中有一个 `string` 列，其中包含多个要分解为单独列的值时，这会非常有用。 例如，你可分解开发人员 trace ("`printf`"/"`Console.WriteLine`") 语句生成的列。

### <a name="using-parse"></a>使用 `parse`

在下面的示例中，`Traces` 表的 `EventText` 列包含 `Event: NotifySliceRelease (resourceName={0}, totalSlices= {1}, sliceNumber={2}, lockTime={3}, releaseTime={4}, previousLockTime={5})` 格式的字符串。 以下操作将用六个列扩展该表：`resourceName`、`totalSlices`、`sliceNumber`、`lockTime `、`releaseTime`、`previouLockTime`、`Month` 和 `Day`。 

有一些字符串未完全匹配。

如果使用 `parse`，计算列将具有 null 值。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=invalid_number, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=invalid_datetime, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=invalid_number, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces  
| parse EventText with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previouLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previousLockTime
```

|resourceName|totalSlices|sliceNumber|lockTime|releaseTime|previousLockTime|
|---|---|---|---|---|---|
|||||||
|||||||
|||||||
|PipelineScheduler|27|20|2016/02/17 08:40:01|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler|27|22|2016/02/17 08:41:01|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|

### <a name="using-parse-where"></a>使用 `parse-where` 

如果使用“parse-where”，将从结果中筛选出未成功分析的字符串。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=invalid_number, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=invalid_datetime, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=invalid_number, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces  
| parse-where EventText with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previousLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previousLockTime
```

|resourceName|totalSlices|sliceNumber|lockTime|releaseTime|previousLockTime|
|---|---|---|---|---|---|
|PipelineScheduler|27|20|2016/02/17 08:40:01|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler|27|22|2016/02/17 08:41:01|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|


### <a name="regex-mode-using-regex-flags"></a>使用正则表达式标志的正则表达式模式

若要获取 resourceName 和 totalSlices，请使用以下查询：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=non_valid_integer, sliceNumber=11, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=non_valid_integer, sliceNumber=44, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces
| parse-where kind = regex EventText with * "RESOURCENAME=" resourceName "," * "totalSlices=" totalSlices:long "," *
| project resourceName, totalSlices
```

### <a name="parse-where-with-case-insensitive-regex-flag"></a>具有不区分大小写的正则表达式标志的 `parse-where`

在上述查询中，默认模式为区分大小写，因此字符串已成功分析。 未获得结果。

若要获取所需结果，请运行包含不区分大小写的 (`i`) 正则表达式标志的 `parse-where`。

只会成功分析三个字符串，因此结果为三个记录（某些 totalSlices 包含无效整数）。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=non_valid_integer, sliceNumber=11, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=non_valid_integer, sliceNumber=44, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces
| parse-where kind = regex flags=i EventText with * "RESOURCENAME=" resourceName "," * "totalSlices=" totalSlices:long "," *
| project resourceName, totalSlices
```

|resourceName|totalSlices|
|---|---|
|PipelineScheduler|27|
|PipelineScheduler|27|
|PipelineScheduler|27|
