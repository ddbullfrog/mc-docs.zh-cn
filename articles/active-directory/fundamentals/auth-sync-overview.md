---
title: Azure Active Directory 身份验证和同步协议概述
description: 实现此身份验证模式的体系结构指南
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/23/2020
ms.author: v-junlch
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: f7cad38b7eebeedbd8c6302ace39d3980381af9c
ms.sourcegitcommit: ca5e5792f3c60aab406b7ddbd6f6fccc4280c57e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92754802"
---
# <a name="azure-active-directory-integrations-with-legacy-authentication-and-synchronization-protocols"></a>Azure Active Directory 与旧式身份验证和同步协议的集成

Azure Active Directory (Azure AD) 支持与许多身份验证和同步协议集成。 通过身份验证集成，只需对使用旧式身份验证方法的应用程序进行少量更改（或无需更改），即可使用 Azure AD 及其安全和管理功能。 利用同步集成，可以将用户和组数据同步到 Azure AD，然后使用用户 Azure AD 管理功能。 某些同步模式还支持自动预配。

## <a name="authentication-patterns"></a>身份验证模式

下表显示了身份验证模式及其功能。 选择要查看的身份验证模式的名称

* 详细说明

* 何时使用

* 体系结构图

* 系统组件的说明

* 有关如何实现集成的链接

 

| 身份验证模式| 身份验证| 授权| 多重身份验证| 条件性访问 |
| - |- | - | - | - |
| [LDAP 身份验证](auth-ldap.md)| ![选中标记](./media/authentication-patterns/check.png)| | |  |
| [OAuth 2.0 身份验证](auth-oauth2.md)| ![选中标记](./media/authentication-patterns/check.png)| ![选中标记](./media/authentication-patterns/check.png)| ![选中标记](./media/authentication-patterns/check.png)| ![选中标记](./media/authentication-patterns/check.png) |
| [OIDC 身份验证](auth-oidc.md)| ![选中标记](./media/authentication-patterns/check.png)| ![选中标记](./media/authentication-patterns/check.png)| ![选中标记](./media/authentication-patterns/check.png)| ![选中标记](./media/authentication-patterns/check.png) |
 
## <a name="synchronization-patterns"></a>同步模式

下表显示了同步模式及其功能。 选择要查看的模式的名称

* 详细说明

* 何时使用

* 体系结构图

* 系统组件的说明

* 有关如何实现集成的链接



| 同步模式| 目录同步| 用户预配 |
| - | - | - |
| [目录同步](sync-directory.md)| ![选中标记](./media/authentication-patterns/check.png)|  |
| [LDAP 同步](sync-ldap.md)| ![选中标记](./media/authentication-patterns/check.png)|  |

