---
title: mv-expand 运算符 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 mv-expand 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/24/2019
ms.date: 09/24/2020
ms.openlocfilehash: ecaf7bf1e525fce63a16bed22a4c6bf9f765bd4c
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146777"
---
# <a name="mv-expand-operator"></a>mv-expand 运算符

展开多值数组或属性包。

`mv-expand` 应用于 [dynamic](./scalar-data-types/dynamic.md) 类型的数组或属性包，以便集合中的每个值都获得一个单独的行。 将复制扩展行中的所有其他列。 

## <a name="syntax"></a>语法

*T* `| mv-expand ` [`bagexpansion=`(`bag` | `array`)] [`with_itemindex=`*IndexColumnName*] *ColumnName* [`,` *ColumnName* ...] [`limit` *Rowlimit*]

*T* `| mv-expand ` [`bagexpansion=`(`bag` | `array`)] [*Name* `=`] *ArrayExpression* [`to typeof(`*Typename*`)`] [, [*Name* `=`] *ArrayExpression* [`to typeof(`*Typename*`)`] ...] [`limit` *Rowlimit*]

## <a name="arguments"></a>参数

* *ColumnName*：在结果中，已命名列中的数组将扩展为多行。 
* *ArrayExpression*：生成数组的表达式。 如果使用此形式，则会添加新列并保留现有列。
* *Name*：新列的名称。
* Typename：指示数组元素的基础类型，该类型将成为 `mv-apply` 运算符生成的列的类型。 应用类型的操作仅限强制转换，不包括分析或类型转换。 不符合声明类型的数组元素将为 `null` 值。
* *RowLimit*：从每个原始行生成的最大行数。 默认值为 2147483647。 

  > [!Note]
  > 运算符 `mvexpand` 的旧格式和过时形式的默认行限制为 128。

* IndexColumnName：如果指定了 `with_itemindex`，则输出将包含一个名为 IndexColumnName 的附加列，该列包含原始展开集合中的项的索引（从 0 开始）。 

## <a name="returns"></a>返回

用于已命名列或数组表达式中任何数组的每个值的多个行。
如果指定了多个列或表达式，则它们会并行展开。 对于每个输入行，将存在与最长展开表达式中的元素数相同的输出行数（较短的列表用 null 填充）。 如果行中的值为空数组，则该行在展开后无内容（不会显示在结果集中）。 但是，如果行中的值不是数组，则该行将按原样保留在结果集中。 

扩展列始终具有动态类型。 如需计算或聚合值，请使用转换，如 `todatetime()` 或 `tolong()`。

支持两种模式的属性包扩展：
* `bagexpansion=bag`：将属性包扩展为单个条目属性包。 此模式是默认扩展。
* `bagexpansion=array`：将属性包扩展为双元素 `[`*key*`,`*value*`]` 数组结构，可允许统一访问键和值（以及对属性名称运行非重复计数聚合）。 

## <a name="examples"></a>示例

### <a name="single-column"></a>单个列

单个列的简单展开：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
 ```kusto
datatable (a:int, b:dynamic)[1,dynamic({"prop1":"a", "prop2":"b"})]
| mv-expand b 
```

|a|b|
|---|---|
|1|{"prop1":"a"}|
|1|{"prop2":"b"}|

### <a name="zipped-two-columns"></a>压缩的两个列

展开两个列时，将首先“压缩”适用的列，然后将其展开：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
datatable (a:int, b:dynamic, c:dynamic)[1,dynamic({"prop1":"a", "prop2":"b"}), dynamic([5, 4, 3])]
| mv-expand b, c
```

|a|b|c|
|---|---|---|
|1|{"prop1":"a"}|5|
|1|{"prop2":"b"}|4|
|1||3|

### <a name="cartesian-product-of-two-columns"></a>两个列的笛卡尔乘积

如果要在展开两列时获取笛卡尔乘积，请将其逐个展开：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
datatable (a:int, b:dynamic, c:dynamic)
  [
  1,
  dynamic({"prop1":"a", "prop2":"b"}),
  dynamic([5, 6])
  ]
| mv-expand b
| mv-expand c
```

|a|b|c|
|---|---|---|
|1|{  "prop1": "a"}|5|
|1|{  "prop1": "a"}|6|
|1|{  "prop2": "b"}|5|
|1|{  "prop2": "b"}|6|

### <a name="convert-output"></a>转换输出

如果希望将 mv-expand 的输出强制转换为某个类型（默认为 dynamic），请使用 `to typeof`：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
datatable (a:string, b:dynamic, c:dynamic)["Constant", dynamic([1,2,3,4]), dynamic([6,7,8,9])]
| mv-expand b, c to typeof(int)
| getschema 
```

ColumnName|ColumnOrdinal|DateType|ColumnType
-|-|-|-
a|0|System.String|string
b|1|System.Object|动态
c|2|System.Int32|int

请注意，列 `b` 是 `dynamic` 类型的，而 `c` 是 `int` 类型的。

### <a name="using-with_itemindex"></a>使用 with_itemindex

通过 `with_itemindex` 展开数组：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
range x from 1 to 4 step 1
| summarize x = make_list(x)
| mv-expand with_itemindex=Index x
```

|x|索引|
|---|---|
|1|0|
|2|1|
|3|2|
|4|3|
 
## <a name="see-also"></a>另请参阅

* 如需更多示例，请参阅[为一段时间内的实时活动计数绘制图表](./samples.md#chart-concurrent-sessions-over-time)。
* [mv-apply](./mv-applyoperator.md) 运算符。
* [summarize make_list()](makelist-aggfunction.md)，这是 mv-expand 的反向函数。
* [bag_unpack()](bag-unpackplugin.md) 插件，用于将动态 JSON 对象扩展为使用属性包键的列。
