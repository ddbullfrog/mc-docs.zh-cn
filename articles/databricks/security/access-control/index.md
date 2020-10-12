---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 访问控制 - Azure Databricks
description: 了解如何管理对 Azure Databricks 数据表、群集、池、作业和工作区对象（如笔记本、试验和文件夹）的访问。
ms.openlocfilehash: 830bcac858025abb2c7a0b66c19dbe2cd911dc70
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937748"
---
# <a name="access-control"></a>访问控制

在 Azure Databricks 中，可以使用访问控制列表 (ACL) 来配置访问工作区对象（文件夹、笔记本、试验和模型）、群集、池、作业和数据表的权限。 所有管理员用户都可以管理访问控制列表，被授予访问控制列表委托管理权限的用户也可以进行此类管理。

管理员用户在 Azure Databricks 工作区级别启用和禁用访问控制。 请参阅[启用访问控制](../../administration-guide/access-control/index.md)。

> [!NOTE]
>
> 只有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)中提供了工作区对象、群集、池、作业和表访问控制。

本部分的内容：

* [工作区对象访问控制](workspace-acl.md)
* [群集访问控制](cluster-acl.md)
* [池访问控制](pool-acl.md)
* [作业访问控制](jobs-acl.md)
* [表访问控制](table-acls/index.md)
* [Secret access control（机密访问控制）](secret-acl.md)