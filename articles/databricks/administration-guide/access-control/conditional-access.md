---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 条件访问 - Azure Databricks
description: 了解如何为 Azure Databricks 启用 Azure Active Directory 条件访问，以便管理员进行相关控制，允许用户在何时何地登录到 Azure Databricks。
ms.openlocfilehash: 5762e1efc8470b91f634d7e2bf36b58c0f514cc6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106461"
---
# <a name="conditional-access"></a>条件性访问

Azure Databricks 支持 Azure Active Directory 条件访问，以便管理员进行相关控制，允许用户在何时何地登录到 Azure Databricks。 例如，条件访问策略可能会限制企业网络的登录，也可能会要求多重身份验证。 有关条件访问的详细信息，请参阅 [Azure Active Directory 中的条件访问是什么](/active-directory/conditional-access/overview)。

> [!NOTE]
>
> 条件访问仅在 Azure AD Premium 中可用。 有关详细信息，请参阅 [Azure AD 定价](https://azure.microsoft.com/pricing/details/active-directory/)。

本文介绍如何为 Azure Databricks 启用条件访问。

## <a name="requirements"></a>要求

你必须是 Azure Active Directory 的条件访问管理员或全局管理员。 有关详细信息，请参阅[在 Azure Active Directory 中分配管理员角色](/active-directory/users-groups-roles/directory-assign-admin-roles)。

## <a name="enable-conditional-access-for-azure-databricks"></a>为 Azure Databricks 启用条件访问

1. 在 Azure 门户中，单击“Azure Active Directory”服务。
2. 单击“安全性”部分的“条件访问”。
3. 单击“新建策略”以创建新的条件访问策略。
4. 在“云应用”中，单击“选择应用”，然后搜索应用程序 ID“2ff814a6-3304-4ab8-85cb-cd0e6f879c1d”。 选择“AzureDatabricks”。

   > [!div class="mx-imgBorder"]
   > ![条件性访问](../../_static/images/conditional-access/azure-databricks-conditional-access.png)

5. 根据首选条件访问配置输入其余设置。 有关教程和详细信息，请参阅 [Azure AD 条件访问文档](/active-directory/conditional-access/)。