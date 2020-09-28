---
title: 教程 - 添加示例模型 - Azure Analysis Services | Azure
description: 本教程介绍如何在 Azure Analysis Services 中添加示例模型。
ms.service: azure-analysis-services
ms.topic: tutorial
origin.date: 08/31/2020
author: rockboyfor
ms.date: 09/21/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.reviewer: minewiskan
ms.openlocfilehash: 6bd4af0439957194e8b63ad597b863c7b708e860
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146236"
---
# <a name="tutorial-add-a-sample-model-from-the-portal"></a>教程：从门户添加示例模型

在本教程中，将向服务器添加示例 Adventure Works 表格模型数据库。 此示例模型是 Adventure Works Internet Sales (1200) 示例数据模型的完整版本。 示例模型可用于测试模型管理、与工具和客户端应用程序进行连接，以及查询模型数据。 本教程使用 [Azure 门户](https://portal.azure.cn)和 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) 执行以下操作： 

> [!div class="checklist"]
> * 将已完成的示例表格数据模型添加到服务器 
> * 使用 SSMS 连接到模型

如果没有 Azure 订阅，请在开始前[创建一个试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

## <a name="prerequisites"></a>先决条件

要完成本教程，需要：

- Azure Analysis Services 服务器。 若要了解更多信息，请参阅[创建服务器 - 门户](analysis-services-create-server.md)。
- 服务器管理员权限
- [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录[门户](https://portal.azure.cn/)。

## <a name="add-a-sample-model"></a>添加示例模型

1. 在服务器“概述”中，单击“新建模型”。

    :::image type="content" source="./media/analysis-services-create-sample-model/aas-create-sample-new-model.png" alt-text="创建示例模型":::

2. 在“新建模型” > “选择数据源”中，确认已选中“示例数据”，然后单击“添加”。   

    :::image type="content" source="./media/analysis-services-create-sample-model/aas-create-sample-data.png" alt-text="选择“新建模型”":::

3. 在“概述”中，确认是否添加了 `adventureworks` 示例模型。

    :::image type="content" source="./media/analysis-services-create-sample-model/aas-create-sample-verify.png" alt-text="选择“示例数据”":::

## <a name="clean-up-resources"></a>清理资源

示例模型使用了缓存内存资源。 如果不使用示例模型进行测试，则应将其从服务器中删除。

下面的步骤介绍了如何使用 SSMS 从服务器中删除模型。

<!-- Not Available on You can also delete a model by using the preview Web designer feature.-->
  
1. 在 SSMS >“对象资源管理器”中，单击“连接” > “Analysis Services”。

2. 在“连接到服务器”中，粘贴服务器名称，然后在“身份验证”中选择“Active Directory - 支持 MFA 的通用身份验证”，输入你的用户名，然后单击“连接”。

    :::image type="content" source="./media/analysis-services-create-sample-model/aas-create-sample-cleanup-signin.png" alt-text="登录":::

3. 在“对象资源管理器”中，右键单击 `adventureworks` 示例数据库，然后单击“删除”。 

    :::image type="content" source="./media/analysis-services-create-sample-model/aas-create-sample-cleanup-delete.png" alt-text="删除示例数据库":::

## <a name="next-steps"></a>后续步骤 

本教程介绍了如何将基本示例模型添加到服务器。 现在，已有模型数据库，可以从 SQL Server Management Studio 连接到它，并添加用户角色。 若要了解详细信息，请继续阅读下一个教程。

> [!div class="nextstepaction"]
> [教程：配置服务器管理员和用户角色](tutorials/analysis-services-tutorial-roles.md)

<!-- Update_Description: update meta properties, wording update, update link -->