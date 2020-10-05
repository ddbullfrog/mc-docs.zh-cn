---
author: orspod
ms.service: data-explorer
ms.topic: include
origin.date: 02/27/2020
ms.date: 09/24/2020
ms.author: v-tawe
ms.openlocfilehash: f8e8cd290a13d95655ec838d6d63668217c639d0
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146613"
---
### <a name="schema-mapping-examples"></a>架构映射示例

**表架构映射示例**

如果数据包含三列（`Timespan`、`Metric` 和 `Value`）并且包含的属性是 `x-opt-enqueued-time` 和 `x-opt-offset`，请使用以下命令创建或更改表架构：

```kusto
    .create-merge table TestTable (TimeStamp: datetime, Metric: string, Value: int, EventHubEnqueuedTime:datetime, EventHubOffset:string)
```

**CSV 映射示例**

运行以下命令，将数据添加到记录的开头。 记下序号值。

```kusto
    .create table TestTable ingestion csv mapping "CsvMapping1"
    '['
    '   { "column" : "Timespan", "Properties":{"Ordinal":"2"}},'
    '   { "column" : "Metric", "Properties":{"Ordinal":"3"}},'
    '   { "column" : "Value", "Properties":{"Ordinal":"4"}},'
    '   { "column" : "EventHubEnqueuedTime", "Properties":{"Ordinal":"0"}},'
    '   { "column" : "EventHubOffset", "Properties":{"Ordinal":"1"}}'
    ']'
```
 
**JSON 映射示例**

使用系统属性映射添加数据。 运行以下命令：

```kusto
    .create table TestTable ingestion json mapping "JsonMapping1"
    '['
    '    { "column" : "Timespan", "Properties":{"Path":"$.timestamp"}},'
    '    { "column" : "Metric", "Properties":{"Path":"$.metric"}},'
    '    { "column" : "Value", "Properties":{"Path":"$.metric_value"}},'
    '    { "column" : "EventHubEnqueuedTime", "Properties":{"Path":"$.x-opt-enqueued-time"}},'
    '    { "column" : "EventHubOffset", "Properties":{"Path":"$.x-opt-offset"}}'
    ']'
```
