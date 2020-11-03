---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 启用访问控制 - Azure Databricks
description: 了解如何管理对 Azure Databricks 数据表、群集、池、作业和工作区对象（如笔记本、试验和文件夹）的访问。
ms.openlocfilehash: 13926d39f5512e9c9e7f5527a6f52494a44c77dd
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106460"
---
# <a name="enable-access-control"></a>启用访问控制

在 Azure Databricks 中，可以使用访问控制列表 (ACL) 来配置访问数据表、群集、池、作业和工作区对象（如笔记本、试验和文件夹）的权限。

所有管理员用户都可以管理访问控制列表，被授予访问控制列表委托管理权限的用户也可以进行此类管理。

本部分介绍管理员用户为启用和禁用访问控制而执行的任务。

> [!NOTE]
>
> 只能在 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)中进行表、群集、池、作业和工作区访问控制。

管理员还可以通过允许或禁止用户生成访问令牌来管理对 Azure Databricks REST API 的访问。

具有适当权限的 Azure 管理员可以配置 _Azure Active Directory 条件访问_ ，以控制允许用户登录 Azure Databricks 的位置和时间，并启用 Azure Data Lake Storage 凭据直通验证（这使用户可以使用其用于登录 Azure Data Lake Storage 的相同 Azure Databricks 标识从 Azure Active Directory 群集向 Azure Data Lake Storage 进行身份验证）。

本部分的内容：

* [启用工作区对象访问控制](workspace-acl.md)
* [为工作区启用群集访问控制](cluster-acl.md)
* [为工作区启用池访问控制](pool-acl.md)
* [为工作区启用作业访问控制](jobs-acl.md)
* [为工作区启用表访问控制](table-acl.md)
* [管理个人访问令牌](tokens.md)
* [条件性访问](conditional-access.md)