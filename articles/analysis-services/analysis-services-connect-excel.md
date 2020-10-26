---
title: 使用 Excel 连接到 Azure Analysis Services | Azure
description: 了解如何使用 Excel 连接到 Azure Analysis Services 服务器。 连接后，用户可以创建数据透视表以浏览数据。
ms.service: azure-analysis-services
ms.topic: conceptual
origin.date: 03/30/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.reviewer: minewiskan
ms.openlocfilehash: 1ef74d57c38848c8a8337c26bd485ebb84eab73b
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470157"
---
# <a name="connect-with-excel"></a>使用 Excel 进行连接

你创建服务器并向其部署表格模型后，客户端便可以连接并开始浏览数据。 

## <a name="before-you-begin"></a>开始之前

用于登录的帐户必须属于具有至少读取权限的模型数据库角色。 若要了解详细信息，请参阅[身份验证和用户权限](analysis-services-manage-users.md)。 

## <a name="connect-in-excel"></a>在 Excel 中连接

可通过使用 Excel 2016 及更高版本中的“获取数据”在 Excel 中连接到服务器。 不支持使用 Power Pivot 中的“导入表向导”进行连接。 

1. 在 Excel 的“数据”功能区上，单击“获取外部数据” > “从其他源获取” > “从 Analysis Services 获取”   。

2. 在“数据连接向导”的“服务器名称”中，输入服务器名称，包括协议和 URI。  例如，asazure://chinanorth.asazure.chinacloudapi.cn/advworks. 然后，在“登录凭据”  中，选择“使用以下用户名和密码”  ，并键入组织的用户名（例如 nancy@adventureworks.com）和密码。

    > [!IMPORTANT]
    > 如果使用 Microsoft 帐户、Live ID、Yahoo 等登录或被要求使用多重身份验证登录，请将密码字段留空。 单击“下一步”后，系统会提示输入密码。
    
    <!--Not Available on Gmail -->
    
    :::image type="content" source="./media/analysis-services-connect-excel/aas-connect-excel-logon.png" alt-text="“从 Excel 中连接”登录":::

3. 在“选择数据库和表”  中，选择数据库和模型或视角，并单击“完成”  。

    :::image type="content" source="./media/analysis-services-connect-excel/aas-connect-excel-select.png" alt-text="“从 Excel 中连接”登录":::

## <a name="see-also"></a>另请参阅

[客户端库](https://docs.microsoft.com/analysis-services/client-libraries?view=azure-analysis-services-current)   
[管理服务器](analysis-services-manage.md)

<!-- Update_Description: update meta properties, wording update, update link -->