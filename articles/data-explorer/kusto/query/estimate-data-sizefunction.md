---
title: estimate_data_size() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 estimate_data_size()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: 0d5256dc612df4dd901cba1c3f113a03ce160500
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105521"
---
# <a name="estimate_data_size"></a>estimate_data_size()

返回表格表达式的选定列的估计数据大小（以字节为单位）。

```kusto
estimate_data_size(*)
estimate_data_size(Col1, Col2, Col3)
```

## <a name="syntax"></a>语法

`estimate_data_size(*)`

`estimate_data_size(`*col1*`, `*col2*`, `...`)`

## <a name="arguments"></a>参数

* col1、col2：选择用于数据大小估算的源表格表达式中的列引用。 若要包括所有列，请使用 `*`（星号）语法。

## <a name="returns"></a>返回

* 记录的估计数据大小（以字节为单位）。 根据数据类型和值的长度进行估算。

## <a name="examples"></a>示例

使用 `estimated_data_size()` 计算总数据大小：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
range x from 1 to 10 step 1                    // x (long) is 8 
| extend Text = '1234567890'                   // Text length is 10  
| summarize Total=sum(estimate_data_size(*))   // (8+10)x10 = 180
```

|总计|
|---|
|180|
