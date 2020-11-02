---
title: top-hitters 运算符 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 top-hitters 运算符。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 35a1f4edfe8e495ada46f7d2fcec8ec1253c5263
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104178"
---
# <a name="top-hitters-operator"></a>top-hitters 运算符

返回前 N 个结果的近似值（假定输入分布呈扭曲状态）。

```kusto
T | top-hitters 25 of Page by Views 
```

> [!NOTE]
> `top-hitters` 是近似算法，应在使用大型数据进行运行时使用。 top-hitters 的近似值基于 Count-Min-Sketch 算法。  

## <a name="syntax"></a>语法

*T* `| top-hitters` *NumberOfRows* `of` *sort_key* `[` `by` *expression* `]`

## <a name="arguments"></a>参数

* NumberOfRows：要返回的 T 的行数。 可以指定任何数值表达式。
* sort_key：对行进行排序所依据的列的名称。
* expression：（可选）一个将用于 top-hitters 估算的表达式。 
    * expression：top-hitters 将返回 NumberOfRows 行，这些行包含 sum(expression) 的近似最大值。 Expression 可以是计算结果为数字的列或任何其他表达式。 
    *  如果未提及 expression，top-hitters 算法将计算 sort-key 出现的次数。  

## <a name="examples"></a>示例

### <a name="get-most-frequent-items"></a>获取最频繁的项 

下一个示例演示如何在维基百科中查找（在 2016 年 4 月之后访问的）页面最多的前 5 种语言。 

```kusto
PageViews
| where Timestamp > datetime(2016-04-01) and Timestamp < datetime(2016-05-01) 
| top-hitters 5 of Language 
```

|语言|approximate_count_Language|
|---|---|
|en|1539954127|
|zh|339827659|
|de|262197491|
|ru|227003107|
|fr|207943448|

### <a name="get-top-hitters-based-on-column-value"></a>获取排名最靠前的项（基于列值）

下一个示例演示如何找到 2016 年维基百科浏览量最多的英文页面。 该查询使用“Views”（整数）来计算页面受欢迎程度（查看次数）。 

```kusto
PageViews
| where Timestamp > datetime(2016-01-01)
| where Language == "en"
| where Page !has 'Special'
| top-hitters 10 of Page by Views
```

|页面|approximate_sum_Views|
|---|---|
|Main_Page|1325856754|
|Web_scraping|43979153|
|Java_(programming_language)|16489491|
|United_States|13928841|
|Wikipedia|13584915|
|Donald_Trump|12376448|
|YouTube|11917252|
|The_Revenant_(2015_film)|10714263|
|Star_Wars:_The_Force_Awakens|9770653|
|Portal:Current_events|9578000|