---
author: rockboyfor
ms.service: container-instances
ms.topic: include
origin.date: 01/31/2020
ms.date: 09/25/2020
ms.testscope: no
ms.testdate: 03/02/2020
ms.author: v-yeche
ms.openlocfilehash: 6133d0a6818a672e370bbe225f9927e2627240ad
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246689"
---
### <a name="maximum-resources-per-sku"></a>每个 SKU 的最大资源数

| OS | GPU SKU | GPU 计数 | 最大 CPU | 最大内存 (GB) | 存储器 (GB) |
| --- | --- | --- | --- | --- | --- |
| Linux | V100 | 1 | 6 | 112 | 50 |
| Linux | V100 | 2 | 12 | 224 | 50 |
| Linux | V100 | 4 | 24 | 448 | 50 |

<!--Not Avaialble on K80, P100-->
<!--VM NCv3 match to Linux V100-->
<!-- Update_Description: update meta properties, wording update, update link -->