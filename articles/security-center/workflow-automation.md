---
title: Azure 安全中心的工作流自动化 | Microsoft Docs
description: 了解如何在 Azure 安全中心创建工作流并将其自动化
services: security-center
author: Johnnytechn
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: v-johya
ms.openlocfilehash: 3ade892e60325d615ae628d6b4b3247485ddf65a
ms.sourcegitcommit: cdb7228e404809c930b7709bcff44b89d63304ec
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2020
ms.locfileid: "91403839"
---
# <a name="workflow-automation"></a>工作流自动化

每个安全计划都包含事件响应的多个工作流。 这些流程可能包含通知相关利益干系人、启动更改管理进程，以及应用特定的修正步骤。 安全专家建议你尽可能多地将这些流程自动化。 自动化可减少开销， 还可确保根据你预定义的要求快速、一致地执行处理步骤，从而增强安全性。

本文介绍 Azure 安全中心的工作流自动化功能。 此功能可根据安全警报和建议触发逻辑应用。 例如，你可能希望安全中心在出现警报时向特定用户发送电子邮件。 你还将了解如何使用 [Azure 逻辑应用](/logic-apps/logic-apps-overview)创建逻辑应用。

> [!NOTE]
> 如果你之前使用过边栏上的 Playbook（预览）视图，则将在新的工作流自动化页面中找到相同的功能以及扩展功能。



## <a name="availability"></a>可用性

|方面|详细信息|
|----|:----|
|发布状态：|正式版|
|定价：|免费层|
|所需角色和权限：|资源组上的安全管理员角色或所有者角色 <br>还必须具有对目标资源的写入权限<br><br>若要使用 Azure 逻辑应用工作流，还必须具有以下逻辑应用角色/权限：<br> 逻辑应用读取/触发访问需要- [逻辑应用操作员](/role-based-access-control/built-in-roles#logic-app-operator)权限（此角色无法创建或编辑逻辑应用，仅可运行现有应用）<br> 创建和修改逻辑应用需要- [逻辑应用参与者](/role-based-access-control/built-in-roles#logic-app-contributor)权限<br>如果要使用逻辑应用连接器，可能需要使用额外的凭据登录到各自的服务（例如 Outlook/Teams/Slack 实例）|
|云：|![是](./media/icons/yes-icon.png) 中国云|
|||



## <a name="create-a-logic-app-and-define-when-it-should-automatically-run"></a>创建一个逻辑应用，并定义它应自动运行的时间 
<!--Customized in MC-->

1. 从安全中心的边栏选择“工作流自动化”。

    [![工作流自动化列表](./media/workflow-automation/list-of-workflow-automations.png)](./media/workflow-automation/list-of-workflow-automations.png#lightbox)

    在此页上，你可创建新的自动化规则，还可启用、禁用或删除现有规则。

1. 若要定义新工作流，请单击“添加工作流自动化”。 

    此时会出现一个窗格，其中包含用于新的自动化的选项。 可在此处输入：
    1. 自动化的名称和说明。
    1. 将启动此自动工作流的触发器。 例如，你可能希望在生成包含“SQL”的安全警报时运行逻辑应用。
    1. 满足触发条件时将运行的逻辑应用。 

        [![工作流自动化列表](./media/workflow-automation/add-workflow.png)](./media/workflow-automation/add-workflow.png#lightbox)

1. 在“操作”部分中单击“访问逻辑应用页面”，创建新的逻辑应用。

    你将转到 Azure 逻辑应用。

    [![创建新的逻辑应用](./media/workflow-automation/logic-apps-create-new.png)](./media/workflow-automation/logic-apps-create-new.png#lightbox)

1. 输入名称、资源组和位置，然后单击“创建”。

1. 在新的逻辑应用中，可从安全类别中选择内置的预定义模板。 也可定义在触发此进程时要发生的自定义事件流。

    在逻辑应用设计器中，支持以下来自安全中心连接器的触发器：

    * **创建或触发 Azure 安全中心建议时**
    * **创建或触发 Azure 安全中心警报时** 
    
    > [!TIP]
    > 你可自定义触发器，使其仅与你关注的严重性级别的警报关联。
    
    > [!NOTE]
    > 如果使用名为“触发 Azure 安全中心警报的响应时”的旧触发器，逻辑应用不会通过工作流自动化功能启动。 请改用上述的任一触发器。 

    [![示例逻辑应用](./media/workflow-automation/sample-logic-app.png)](./media/workflow-automation/sample-logic-app.png#lightbox)

1. 定义逻辑应用后，回到工作流自动化定义窗格（“添加工作流自动化”）。 单击“刷新”，确保新的逻辑应用可供选择。

    ![刷新](./media/workflow-automation/refresh-the-list-of-logic-apps.png)

1. 选择逻辑应用并保存自动化。 请注意，“逻辑应用”下拉列表仅显示支持上述安全中心连接器的逻辑应用。

<!--Customized in MC-->

## <a name="manually-trigger-a-logic-app"></a>手动触发逻辑应用

查看任何安全警报或建议时，还可手动运行逻辑应用。

若要手动运行逻辑应用，请打开警报或建议，然后单击“触发逻辑应用”：

[![手动触发逻辑应用](./media/workflow-automation/manually-trigger-logic-app.png)](./media/workflow-automation/manually-trigger-logic-app.png#lightbox)

## <a name="data-types-schemas"></a>数据类型架构

若要查看传递到逻辑应用实例的安全警报或建议事件的原始事件架构，请访问[工作流自动化数据类型架构](https://aka.ms/ASCAutomationSchemas)。 如果你没有使用上述安全中心的内置逻辑应用连接器，而是使用逻辑应用的通用 HTTP 连接器，这将非常有用，而且你可根据需要使用事件 JSON 架构手动分析它。

## <a name="next-steps"></a>后续步骤

在本文中，你学习了如何创建逻辑应用、如何在安全中心自动执行这些应用以及如何手动运行它们。 

有关其他相关资料，请参阅： 

- [关于如何使用工作流自动化来自动执行安全响应的 Microsoft Learn 模块](https://docs.microsoft.com/learn/modules/resolve-threats-with-azure-security-center/)
- [Azure 安全中心的安全建议](security-center-recommendations.md)
- [Azure 安全中心中的安全警报](security-center-alerts-overview.md)
- [关于 Azure 逻辑应用](/logic-apps/logic-apps-overview)
- [逻辑应用连接器](https://docs.microsoft.com/connectors/)
- [工作流自动化数据类型架构](https://aka.ms/ASCAutomationSchemas)

