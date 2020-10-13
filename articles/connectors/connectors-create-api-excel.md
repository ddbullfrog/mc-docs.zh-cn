---
title: 管理 Excel Online 中的数据、工作表和表
description: 使用 Azure 逻辑应用管理 Excel Online for Business 或 Excel Online for OneDrive 的工作表和表中的数据
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
origin.date: 08/23/2018
author: rockboyfor
ms.date: 10/05/2020
ms.author: v-yeche
tags: connectors
ms.openlocfilehash: 7de60df206ab9b3f6174acbd95dac9914e642e22
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564535"
---
# <a name="manage-excel-online-data-with-azure-logic-apps"></a>使用 Azure 逻辑应用管理 Excel Online 数据

<!--Not Available on [Excel Online for Business](https://docs.microsoft.com/connectors/excelonlinebusiness/)-->

使用 [Azure 逻辑应用](../logic-apps/logic-apps-overview.md)和 [Excel Online for OneDrive](https://docs.microsoft.com/connectors/excelonline/) 连接器，可以基于 Excel Online for Business 或 Excel Online for OneDrive 中的数据创建自动化任务和工作流。 此连接器提供了可以帮助你处理数据以及管理电子表格的操作，例如：

* 创建新的工作表和表。
* 获取和管理工作表、表和行。
* 添加单个行和键列。

然后，你可以将来自这些操作的输出用于其他服务的操作。 例如，如果你使用一个每周创建工作表的操作，则可以使用另一个操作通过 Office 365 Outlook 连接器发送确认电子邮件。

如果你不熟悉逻辑应用，请查看[什么是 Azure 逻辑应用？](../logic-apps/logic-apps-overview.md)

<!--MOONCAKE: EXCELONLINE(BUSINESS) NOT EXISTS IN MOONCAKE-->

> [!NOTE]
> [Excel Online for OneDrive](https://docs.microsoft.com/connectors/excelonline/) 连接器可用于 Azure 逻辑应用，不同于[适用于 PowerApps 的 Excel 连接器](https://docs.microsoft.com/connectors/excel/)。

<!--Not Available on [Excel Online for Business](https://docs.microsoft.com/connectors/excelonlinebusiness/)-->


## <a name="prerequisites"></a>先决条件

* Azure 订阅。 如果没有 Azure 订阅，请[注册一个 Azure 试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

* 用于你的工作帐户或个人 Microsoft 帐户的一个[工作或学校帐户](https://www.office.com/)

    你的 Excel 数据可以作为以逗号分隔的值 (CSV) 文件存在于存储文件夹中，例如在 OneDrive 中。 
    还可以将同一 CSV 文件与[平面文件连接器](../logic-apps/logic-apps-enterprise-integration-flatfile.md)结合使用。

* 有关[如何创建逻辑应用](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知识

* 要在其中访问 Excel Online 数据的逻辑应用。 
    此连接器仅提供操作，因此，若要启动逻辑应用，请选择单独的触发器（例如“重复”触发器）。 

## <a name="add-excel-action"></a>添加 Excel 操作

1. 在 [Azure 门户](https://portal.azure.cn)中，在逻辑应用设计器中打开你的逻辑应用（如果尚未打开）。

1. 在触发器下，选择“新建步骤”。 

1. 在搜索框中，输入“excel”作为筛选器。 在操作列表下，选择所需的操作。

    > [!NOTE]
    > 逻辑应用设计器无法加载包含 100 列或更多列的表。 如果可能，请减少所选表中的列数，以便设计器可以加载表。

1. 如果收到提示，则登录工作或学校帐户。

    你的凭据授权逻辑应用创建与 Excel Online 的连接并访问你的数据。

1. 继续为所选操作提供必要的详细信息并构建逻辑应用的工作流。

## <a name="connector-reference"></a>连接器参考

如需技术详细信息（例如触发器、操作和限制，如连接器的 OpenAPI（以前为 Swagger）文件所述），请参阅以下连接器参考页：

<!--Not Available on * [Excel Online for Business](https://docs.microsoft.com/connectors/excelonlinebusiness/)-->

* [Excel Online for OneDrive](https://docs.microsoft.com/connectors/excelonline/)

## <a name="next-steps"></a>后续步骤

* 了解其他[逻辑应用连接器](../connectors/apis-list.md)

<!-- Update_Description: update meta properties, wording update, update link -->