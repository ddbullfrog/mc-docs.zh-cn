---
title: log10() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 log10()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 08/11/2019
ms.date: 10/29/2020
ms.openlocfilehash: ac7834a865ba715ca54dafa30cf55642bbecfb80
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105923"
---
# <a name="log10"></a>log10()

`log10()` 返回常用（以 10 为底数）对数函数。  

## <a name="syntax"></a>语法

`log10(`*x*`)`

## <a name="arguments"></a>参数

* x：大于 0 的实数。

## <a name="returns"></a>返回

* 常用对数是以 10 为底的对数：以 10 为底数的指数函数 (exp) 的倒数。
* 如果参数为负或为 NULL 或不能转换为 `real` 值，则为 `null`。 

## <a name="see-also"></a>请参阅

* 有关自然对数（以 e 为底），请参阅 [log()](log-function.md)。
* 有关以 2 为底的对数，请参阅 [log2()](log2-function.md)