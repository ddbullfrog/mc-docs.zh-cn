---
title: extract_all() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 extract_all()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 07223483af513002247514ee1434d55f0d7bac9e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105156"
---
# <a name="extract_all"></a>extract_all()

从文本字符串中获取[正则表达式](./re2.md)的所有匹配项。
还可以检索匹配组的子集。

```kusto
print extract_all(@"(\d+)", "a set of numbers: 123, 567 and 789") // results with the dynamic array ["123", "567", "789"]
```

## <a name="syntax"></a>语法

`extract_all(`*regex*`,` [ *captureGroups*`,`] *text*`)`

## <a name="arguments"></a>参数

|参数        |描述                                  |必需或可选  |
|----------------|---------------------------------------------|----------------------|
|regex           | 一个[正则表达式](./re2.md)。 该表达式必须至少有一个捕获组，并且其捕获组数量必须小于或等于 16 个                                                         |必须              |
|captureGroups   |一个动态数组常量，指示要提取的捕获组。 有效值为 1 到正则表达式中的捕获组数量。 还允许命名的捕获组（请参阅[示例](#examples)）|可选         |
|text            |要搜索的 `string`                         |必须              |

## <a name="returns"></a>返回

* 如果 regex 在 text 中找到匹配项，则：返回一个动态数组，其中包括针对所指示的捕获组 captureGroups 或 regex 中的所有捕获组的所有匹配项。
* 如果 captureGroups 的数目为 1，则：返回的数组有一维的匹配值。
* 如果 captureGroups 的数目大于 1，则：返回的数组是一个二维集合，其中包含针对 captureGroups 选择或 regex 中出现的所有捕获组（如果省略 captureGroups）的多值匹配项。
* 如果没有匹配项，则返回：`null`。

## <a name="examples"></a>示例

### <a name="extract-a-single-capture-group"></a>提取单个捕获组

返回 GUID 的十六进制字节表示形式（两个十六进制数字）。

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"([\da-f]{2})", Id) 
```

|ID|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|["82","b8","be","2d","df","a7","4b","d1","8f","63","24","ad","26","d3","14","49"]|

### <a name="extract-several-capture-groups"></a>提取多个捕获组 

使用带有三个捕获组的正则表达式，将每个 GUID 部分拆分为第一个字母、最后一个字母以及中间部分。

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(\w)(\w+)(\w)", Id)
```

|ID|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8","2b8be2","d"],["d","fa","7"],["4","bd","1"],["8","f6","3"],["2","4ad26d3144","9"]]|

### <a name="extract-a-subset-of-capture-groups"></a>提取捕获组的子集

说明如何选择捕获组的子集。 正则表达式匹配第一个字母、最后一个字母以及所有剩余部分。 captureGroups 参数仅用于选择第一个和最后一个部分。

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(\w)(\w+)(\w)", dynamic([1,3]), Id) 
```

|ID|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8","d"],["d","7"],["4","1"],["8","3"],["2","9"]]|

### <a name="using-named-capture-groups"></a>使用命名捕获组

可以在 extract_all() 中使用 RE2 的命名捕获组。
captureGroups 使用捕获组索引和命名捕获组引用来提取匹配值。

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(?P<first>\w)(?P<middle>\w+)(?P<last>\w)", dynamic(['first',2,'last']), Id) 
```

|ID|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8","2b8be2","d"],["d","fa","7"],["4","bd","1"],["8","f6","3"],["2","4ad26d3144","9"]]|
