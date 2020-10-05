---
title: 链接的 Log Analytics 工作区支持的区域
description: 本文介绍支持的自动化帐户与 Log Analytics 工作区之间的区域映射。
services: automation
ms.service: automation
ms.subservice: process-automation
author: WenJason
ms.author: v-jay
origin.date: 09/03/2020
ms.date: 09/28/2020
ms.topic: conceptual
manager: digimobile
ms.custom: references_regions
ms.openlocfilehash: 63f58cb52015a438f44af62b5a864adc4eb414db
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246727"
---
# <a name="supported-regions-for-linked-log-analytics-workspace"></a>链接的 Log Analytics 工作区支持的区域

在 Azure 自动化中，可以为服务器和虚拟机启用更新管理。 此功能依赖于 Log Analytics 工作区，因此需要将工作区链接到自动化帐户。 但是，只有某些区域才支持将它们链接在一起。 通常，如果你计划将自动化帐户链接到不会启用这些功能的工作区，则该映射不适用。

本文提供了支持的映射，以便在自动化帐户中成功启用和使用这些功能。

有关更多信息，请参阅 [Log Analytics 工作区和自动化帐户](../../azure-monitor/insights/solutions.md#log-analytics-workspace-and-automation-account)。

## <a name="supported-mappings"></a>支持的映射

下表显示了受支持的映射：

|**Log Analytics 工作区域**|**Azure 自动化区域**|
|---|---|
|ChinaEast2|ChinaEast2|

## <a name="unlink-a-workspace"></a>取消链接工作区

如果决定不再想要将自动化帐户与 Log Analytics 工作区集成，可以直接从 Azure 门户取消链接帐户。 在继续操作之前，首先需要删除正在使用的更新管理。 如果不删除，则无法完成取消链接操作。 

删除这些功能后，可以按照以下步骤取消链接自动化帐户。

> [!NOTE]
> 某些功能（包括早期版本的 Azure SQL 监视解决方案）可能已创建需要在取消链接工作区之前删除的自动化资产。

1. 在 Azure 门户中，打开自动化帐户。 在“自动化帐户”页上，选择“相关资源”下的“链接的工作区” 。

2. 在“取消链接工作区”页上，选择“取消链接工作区”。 系统会提示用户确认是否要继续。

3. 当 Azure 自动化从 Log Analytics 工作区中取消链接帐户时，可以在菜单中的“通知”下跟踪进度。

4. 建议删除以下不再需要的项：

    * 更新计划：每个计划都具有与所创建的更新部署匹配的名称。
    * 为功能创建的混合辅助角色组：每个名称都具有类似于 `machine1.contoso.com_9ceb8108-26c9-4051-b6b3-227600d715c8` 的名称。

或者，也可在工作区内取消工作区与自动化帐户的链接。

1. 在工作区中，选择“相关资源”下的“自动化帐户” 。
2. 在“自动化帐户”页上，选择“取消链接帐户”。

## <a name="next-steps"></a>后续步骤

* 在[更新管理概述](../update-management/update-mgmt-overview.md)中详细了解更新管理。
