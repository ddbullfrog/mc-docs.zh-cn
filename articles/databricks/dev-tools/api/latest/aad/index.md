---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/15/2020
title: 使用 Azure Active Directory 令牌进行身份验证 - Azure Databricks
description: 了解如何使用 Azure Active Directory (Azure AD) 令牌向 Databricks REST API 进行身份验证并访问 Databricks REST API。
ms.openlocfilehash: 2a868932f9b855ba675a392487bcd86b7da8e9e6
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937801"
---
# <a name="authentication-using-azure-active-directory-tokens"></a>使用 Azure Active Directory 令牌进行身份验证

若要向 Databricks REST API 进行身份验证，可以使用 Azure Databricks 个人访问令牌或 Azure Active Directory 令牌。

此部分介绍了如何获取、使用和刷新 Azure AD 令牌。 有关 Azure Databricks 个人访问令牌，请参阅[使用 Azure Databricks 个人访问令牌进行身份验证](../authentication.md)。

此部分介绍了两种用于获取和使用 Azure AD 访问令牌的方法：

* 使用 Azure Active Directory 身份验证库 (ADAL) 以编程方式为用户获取 Azure AD 访问令牌。
* 在 Azure Active Directory 中定义服务主体，并为该服务主体（而不是用户）获取 AAD 访问令牌。 将服务主体配置为，可以在 Azure Databricks 中强制执行身份验证和授权策略。 Azure Databricks 工作区中的服务主体可以有不同于常规用户（用户主体）的精细访问控制。

本部分内容：

* [使用 Azure Active Directory 身份验证库获取 Azure Active Directory 令牌](app-aad-token.md)
* [使用服务主体获取 Azure Active Directory 令牌](service-prin-aad-token.md)
* [Azure Active Directory 访问令牌疑难解答](troubleshoot-aad-token.md)