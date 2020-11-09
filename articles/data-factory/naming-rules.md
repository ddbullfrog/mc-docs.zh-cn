---
title: Azure 数据工厂实体的命名规则
description: 描述数据工厂实体的命名规则。
services: data-factory
documentationcenter: ''
author: WenJason
ms.author: v-jay
manager: digimobile
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
origin.date: 10/15/2018
ms.date: 11/02/2020
ms.openlocfilehash: 0d4db72b10a6f144985565c8d30a9a869f468b7d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106030"
---
# <a name="azure-data-factory---naming-rules"></a>Azure 数据工厂 - 命名规则

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

下表提供了数据工厂项目的命名规则。

| 名称 | 名称唯一性 | 验证检查 |
|:--- |:--- |:--- |
| 数据工厂 |在 Azure 内唯一。 名称不区分大小写，即 `MyDF` 和 `mydf` 指的是同一个数据工厂。 |<ul><li>一个数据工厂绑定到一个 Azure 订阅。</li><li>对象名称必须以字母或数字开头，并且只能包含字母、数字和短划线 (-) 字符。</li><li>每个短划线 (-) 字符的前后必须紧接字母或数字。 容器名称中不允许使用连续短划线。</li><li>名称长度为 3-63 个字符。</li></ul> |
| 链接服务/数据集/管道 |在数据工厂中唯一。 名称不区分大小写。 |<ul><li>对象名称必须以字母开头。</li><li>不允许使用以下字符：“.”、“+”、“?”、“/”、“<”、“>”、“*”、“%”、“&”、“:”、“\\”</li><li>短划线 ("-") 不可用于链接服务和数据集的名称。</li></ul>  |
| Integration Runtime |在数据工厂中唯一。 名称不区分大小写。 |<ul><li>集成运行时名称只能包含字母、数字和短划线 (-) 字符。</li><li>第一个字符和最后一个字符必须是字母或数字。 每个短划线 (-) 字符的前后必须紧接字母或数字。</li><li>集成运行时名称中不允许使用连续短划线。 </li></ul> |
| 资源组 |在 Azure 内唯一。 名称不区分大小写。 | 有关详细信息，请参阅 [Azure 命名规则和限制](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#resource-naming)。 |

## <a name="next-steps"></a>后续步骤
了解如何按照[快速入门：创建数据工厂](quickstart-create-data-factory-powershell.md)一文中的分步说明创建数据工厂。 
