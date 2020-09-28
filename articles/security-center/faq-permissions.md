---
title: Azure 安全中心常见问题解答 - 有关权限的问题
description: 此常见问题解答回答有关 Azure 安全中心中的权限问题，Azure 安全中心是一个可帮助预防、检测和响应威胁的产品。
services: security-center
documentationcenter: na
author: Johnnytechn
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/14/2020
ms.author: v-johya
origin.date: 02/25/2020
ms.openlocfilehash: 58e40eb55cb0a0bb109319127b9c34bdccc5947b
ms.sourcegitcommit: cdb7228e404809c930b7709bcff44b89d63304ec
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2020
ms.locfileid: "91402574"
---
# <a name="permissions"></a>权限

## <a name="how-do-permissions-work-in-azure-security-center"></a>权限在 Azure 安全中心中的工作原理是什么？

Azure 安全中心使用 [Azure 基于角色的访问控制 (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) 提供可在 Azure 中分配给用户、组和服务的[内置角色](../role-based-access-control/built-in-roles.md)。

安全中心会评估资源的配置以识别安全问题和漏洞。 如果分配有资源所属的订阅或资源组的“所有者”、“参与者”或“读取者”角色，则仅可在安全中心看到与资源相关的信息。

若要深入了解安全中心中的角色和允许的操作，请参阅 [Azure 安全中心中的权限](security-center-permissions.md)。



## <a name="who-can-modify-a-security-policy"></a>哪些用户可以修改安全策略？

只有安全管理员或者订阅的所有者或参与者才能修改安全策略。

若要了解如何配置安全策略，请参阅[在 Azure 安全中心设置安全策略](tutorial-security-policy.md)。

