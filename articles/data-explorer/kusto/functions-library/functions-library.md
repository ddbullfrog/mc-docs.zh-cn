---
title: 函数库 - Azure 数据资源管理器
description: 本文介绍用于扩展 Azure 数据资源管理器功能的用户定义函数。
author: orspod
ms.author: v-tawe
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: reference
origin.date: 09/08/2020
ms.date: 09/24/2020
ms.openlocfilehash: a965d38962636c9bc123a6128a041945933c278f
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146868"
---
# <a name="functions-library"></a>函数库

下面的文章包含用户定义函数的分类列表。

## <a name="machine-learning-functions"></a>机器学习函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[predict_fl()](predict-fl.md)|使用经过训练的现有机器学习模型进行预测。 |
|[predict_onnx_fl()](predict-onnx-fl.md)| 使用 ONNX 格式的经过训练的现有机器学习模型进行预测。 |

## <a name="series-processing-functions"></a>序列处理函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[quantize_fl()](quantize-fl.md)|量化指标列。 |
|[series_fit_poly_fl()](series-fit-poly-fl.md)|使用回归分析将多项式拟合到序列。 |
|[series_moving_avg_fl()](series-moving-avg-fl.md)|对序列应用移动平均滤波器。 |
|[series_rolling_fl()](series-rolling-fl.md)|对序列应用滚动聚合函数。 |
