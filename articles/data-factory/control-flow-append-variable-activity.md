---
title: Azure 数据工厂中的追加变量活动
description: 了解如何设置“追加变量”活动以将值添加到数据工厂管道中定义的现有数组变量
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: WenJason
ms.author: v-jay
manager: digimobile
ms.reviewer: maghan
origin.date: 10/09/2018
ms.date: 05/11/2020
ms.openlocfilehash: 4ccaff7edaf8c255f56225eccf0caddc8ceebaea
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121694"
---
# <a name="append-variable-activity-in-azure-data-factory"></a>Azure 数据工厂中的追加变量活动
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]
使用“追加变量”活动将值添加到数据工厂管道中定义的现有数组变量。

## <a name="type-properties"></a>Type 属性

属性 | 说明 | 必须
-------- | ----------- | --------
name | 管道中活动的名称 | 是
description | 描述活动用途的文本 | 否
type | 活动类型为 AppendVariable | 是
value | 用于追加到指定变量的字符串文本或表达式对象值 | 是
variableName | 活动将修改的变量的名称，该变量必须是“Array”类型 | 是

## <a name="next-steps"></a>后续步骤
了解数据工厂支持的相关控制流活动： 

- [设置变量活动](control-flow-set-variable-activity.md)
