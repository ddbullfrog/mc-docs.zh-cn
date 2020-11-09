---
title: 函数库 - Azure 数据资源管理器
description: 本文介绍用于扩展 Azure 数据资源管理器功能的用户定义函数。
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: reference
origin.date: 09/08/2020
ms.date: 10/29/2020
ms.openlocfilehash: 73d1b9633203c672216c8f8cafc5bc1a00fc8f4d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103567"
---
# <a name="functions-library"></a>函数库

下文包含 [UDF（用户定义的函数）](../query/functions/user-defined-functions.md)的分类列表。

文章中提供了用户定义的函数代码。  它可在查询中嵌入的 let 语句中使用，也可保留在使用 [.create 函数](../management/create-function.md)的数据库中。

## <a name="machine-learning-functions"></a>机器学习函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[kmeans_fl()](kmeans-fl.md)|使用 k-means 算法进行聚类。 |
|[predict_fl()](predict-fl.md)|使用经过训练的现有机器学习模型进行预测。 |
|[predict_onnx_fl()](predict-onnx-fl.md)| 使用 ONNX 格式的经过训练的现有机器学习模型进行预测。 |

## <a name="series-processing-functions"></a>序列处理函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[quantize_fl()](quantize-fl.md)|量化指标列。 |
|[series_dot_product_fl()](series-dot-product-fl.md)|计算两个数值向量的点积。 |
|[series_fit_poly_fl()](series-fit-poly-fl.md)|使用回归分析将多项式拟合到序列。 |
|[series_moving_avg_fl()](series-moving-avg-fl.md)|对序列应用移动平均滤波器。 |
|[series_rolling_fl()](series-rolling-fl.md)|对序列应用滚动聚合函数。 |
