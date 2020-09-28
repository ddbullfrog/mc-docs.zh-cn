---
title: bag_unpack 插件 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 bag_unpack 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 06/15/2020
ms.date: 09/24/2020
ms.openlocfilehash: bb291b8bb3dcd563f65a1db194b06acde3f25dbf
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146257"
---
# <a name="bag_unpack-plugin"></a>bag_unpack 插件

`bag_unpack` 插件通过将每个属性包顶级槽视为列来解压缩 `dynamic` 类型的单个列。

```kusto
T | evaluate bag_unpack(col1)
```

## <a name="syntax"></a>语法

*T* `|` `evaluate` `bag_unpack(` *Column* [`,` *OutputColumnPrefix* ] [`,` *columnsConflict* ] [`,` *ignoredProperties* ] `)`

## <a name="arguments"></a>参数

* *T*：要解压缩其列 Column 的表格输入。
* *Column*：要解压缩的列 T。 必须是类型 `dynamic`。
* *OutputColumnPrefix*：添加到插件生成的所有列的常见前缀。 该参数可选。
* *columnsConflict*：有关列冲突解决的指导。 该参数可选。 如果提供了参数，该参数应为与以下值之一匹配的字符串文本：
    - `error` - 查询生成错误（默认值）
    - `replace_source` -源列被替换
    - `keep_source` - 源列被保留
* *ignoredProperties*：要忽略包属性的可选集。 如果提供了参数，该参数应为具有一个或多个字符串文本的 `dynamic` 数组的常量。

## <a name="returns"></a>返回

`bag_unpack` 插件返回一个表，其中包含的记录与其表格输入 (T) 的数量相同。 该表格的架构与表格输入的架构相同，但有以下修改：

* 删除了指定的输入列 (Column)。
* 该架构扩展多个列，与 T 的顶层属性包值中非重复槽的数量相同。每个列的名称对应每个槽的名称，可选择将 OutputColumnPrefix 作为前缀。 如果同一槽的所有值具有相同的类型，则其类型为槽的类型；如果类型中的值不同，则为 `dynamic`。

**说明**

插件的输出架构依赖于数据值，这使得它和数据本身一样“不可预测”。 使用不同的数据输入多次执行插件可能会产生不同的输出架构。

插件的输入数据必须确保输出架构遵循表格架构的所有规则。 具体而言：

* 如果不是要解压缩的列 (Column)，则输出列名称不能与表格输入 T 中的现有列相同，因为这会产生两个具有相同名称的列 。

* 以 OutputColumnPrefix 为前缀时，所有槽名称必须是有效的实体名称，并遵循[标识符命名规则](./schema-entities/entity-names.md#identifier-naming-rules)。

## <a name="examples"></a>示例

### <a name="expand-a-bag"></a>展开包


<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(d:dynamic)
[
    dynamic({"Name": "John", "Age":20}),
    dynamic({"Name": "Dave", "Age":40}),
    dynamic({"Name": "Jasmine", "Age":30}),
]
| evaluate bag_unpack(d)
```

|名称  |Age|
|------|---|
|John  |20 个 |
|Dave  |40 |
|Jasmine|30 |


### <a name="expand-a-bag-with-outputcolumnprefix"></a>使用 OutputColumnPrefix 展开包

展开包，并使用 `OutputColumnPrefix` 选项生成以前缀“Property_”开头的列名称。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(d:dynamic)
[
    dynamic({"Name": "John", "Age":20}),
    dynamic({"Name": "Dave", "Age":40}),
    dynamic({"Name": "Jasmine", "Age":30}),
]
| evaluate bag_unpack(d, 'Property_')
```

|Property_Name|Property_Age|
|---|---|
|John|20 个|
|Dave|40|
|Jasmine|30|

### <a name="expand-a-bag-with-columnsconflict"></a>使用 columnsConflict 展开包

展开包，并使用 `columnsConflict` 选项来解决现有的列和 `bag_unpack()` 运算符生成的列之间的冲突。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(Name:string, d:dynamic)
[
    'Old_name', dynamic({"Name": "John", "Age":20}),
    'Old_name', dynamic({"Name": "Dave", "Age":40}),
    'Old_name', dynamic({"Name": "Jasmine", "Age":30}),
]
| evaluate bag_unpack(d, columnsConflict='replace_source') // Use new name
```

|名称|Age|
|---|---|
|John|20 个|
|Dave|40|
|Jasmine|30|

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(Name:string, d:dynamic)
[
    'Old_name', dynamic({"Name": "John", "Age":20}),
    'Old_name', dynamic({"Name": "Dave", "Age":40}),
    'Old_name', dynamic({"Name": "Jasmine", "Age":30}),
]
| evaluate bag_unpack(d, columnsConflict='keep_source') // Keep old name
```

|名称|Age|
|---|---|
|Old_name|20 个|
|Old_name|40|
|Old_name|30|

### <a name="expand-a-bag-with-ignoredproperties"></a>使用 ignoredProperties 展开包

展开包，并使用 `ignoredProperties` 选项以忽略属性包中的某些属性。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(d:dynamic)
[
    dynamic({"Name": "John", "Age":20, "Address": "Address-1" }),
    dynamic({"Name": "Dave", "Age":40, "Address": "Address-2"}),
    dynamic({"Name": "Jasmine", "Age":30, "Address": "Address-3"}),
]
// Ignore 'Age' and 'Address' properties
| evaluate bag_unpack(d, ignoredProperties=dynamic(['Address', 'Age']))
```

|名称|
|---|
|John|
|Dave|
|Jasmine|
