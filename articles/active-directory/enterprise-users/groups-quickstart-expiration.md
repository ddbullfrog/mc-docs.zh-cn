---
title: 组过期策略快速入门  - Azure AD | Microsoft Docs
description: Microsoft 365 组的过期 - Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: quickstart
ms.date: 11/05/2020
ms.author: v-junlch
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9003b1352ac2f0c3fc517da95cd575c7b787c6fb
ms.sourcegitcommit: 33f2835ec41ca391eb9940edfcbab52888cf8a01
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327103"
---
# <a name="quickstart-set-microsoft-365-groups-to-expire-in-azure-active-directory"></a>将 Azure Active Directory 中的 Microsoft 365 组设置为过期

本快速入门介绍如何设置 Microsoft 365 组的过期策略。 当用户可以设置自己的组时，未使用的组可能会成倍增加。 若要管理未使用的组，一种方式是将这些组设置为过期，以便减少手动删除组的维护工作量。

过期策略很简单：

- 具有用户活动的组在快要过期时自动续订
- 系统通知组所有者续订要过期的组
- 未续订的组会被删除
- 已删除的 Microsoft 365 组可以由组所有者或 Azure AD 管理员在 30 天内还原

> [!NOTE]
> 现在，组使用 Azure AD 智能，根据是否在最近使用而自动续订。 此续订决策基于 Microsoft 365 服务（如 Outlook、SharePoint、Teams、Yammer 等）中组的用户活动。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

## <a name="prerequisite"></a>先决条件

 设置组过期所需的最低特权角色是组织的用户管理员。

## <a name="turn-on-user-creation-for-groups"></a>为组启用创建用户的功能

1. 使用用户管理员帐户登录到 [Azure 门户](https://portal.azure.cn)。

2. 依次选择“组”、“常规”。
  
   ![自助组设置页](./media/groups-quickstart-expiration/self-service-settings.png)

3. 将“用户可创建 Microsoft 365 组”设置为“是”。

4. 设置完成后，选择“保存”以保存组设置。

## <a name="set-group-expiration"></a>设置组过期

1. 登录到 [Azure 门户](https://portal.azure.cn)，选择“Azure Active Directory” > “组” > “过期”打开过期设置。
  
   ![组过期设置页](./media/groups-quickstart-expiration/expiration-settings.png)

2. 设置过期时间间隔。 选择一个预设值，或者输入一个超过 31 天的自定义值。 

3. 提供当组没有所有者时用于发送过期通知的电子邮件地址。

4. 在本快速入门中，请将“为这些 Microsoft 365 组启用到期功能”设置为“所有”。

5. 设置完成后，选择“保存”来保存过期设置。

就这么简单！ 在本快速入门中，你已成功设置所选 Microsoft 365 组的过期策略。

## <a name="clean-up-resources"></a>清理资源

### <a name="to-remove-the-expiration-policy"></a>删除过期策略

1. 确保已使用 Azure AD 组织的全局管理员帐户登录到 [Azure 门户](https://portal.azure.cn)。
2. 选择“Azure Active Directory” > “组” > “过期”。
3. 将“为这些 Microsoft 365 组启用到期功能”设置为“无”。

### <a name="to-turn-off-user-creation-for-groups"></a>为组禁用创建用户的功能

1. 选择“Azure Active Directory” > “组” > “常规”。 
2. 将“用户可以在 Azure 门户中创建 Microsoft 365 组”设置为“否”。

