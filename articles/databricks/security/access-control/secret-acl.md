---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 机密访问控制 - Azure Databricks
description: 了解如何设置对机密和机密范围的访问控制进行管理的细化权限。
ms.openlocfilehash: a43f10e77ad2beb76b2b0bd838608c8e2e03633a
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937818"
---
# <a name="secret-access-control"></a>机密访问控制

默认情况下，所有定价计划中的所有用户都可以创建机密和机密范围。
使用 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)提供的机密访问控制，你可以配置细化的权限来管理访问控制。
本指南介绍如何设置这些控制。

> [!NOTE]
>
> * 访问控制仅在 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)中提供。 如果你的帐户有标准计划，则在[创建机密范围](../secrets/secret-scopes.md#databricks-backed)时，必须显式将 `MANAGE` 权限授予“users”（所有用户）组。
>
> * 本文介绍如何使用 [Databricks CLI](../../dev-tools/cli/index.md)（版本 0.7.1 及更高版本）管理机密访问控制。 也可以使用[机密 API](../../dev-tools/api/latest/secrets.md)。

## <a name="secret-access-control"></a>机密访问控制

机密的访问控制在机密范围级别进行管理。 访问控制列表 (ACL) 定义了 Azure Databricks 主体（用户或组）、机密范围和权限级别之间的关系。 通常，用户将使用可供其使用的最强大的权限（请参阅[权限级别](#permission-levels)）。

当使用[机密实用工具](../../dev-tools/databricks-utils.md#dbutils-secrets)通过笔记本读取机密时，将根据执行命令的人员应用用户的权限，并且该人员必须至少具有“READ”权限。

创建范围时，会将初始“MANAGE”权限级别 ACL 应用于该范围。 后续访问控制配置可以由该主体执行。

## <a name="permission-levels"></a>权限级别

机密访问权限如下所示：

* **MANAGE** - 允许更改 ACL，并在此机密范围内进行读取和写入。
* **WRITE** - 允许在此机密范围内进行读取和写入。
* **READ** - 允许读取此机密范围并列出哪些机密可用。

每个权限级别都是上一级别的权限的子集（即，对于给定范围，具有 WRITE 权限的主体可以执行所有需要 READ 权限的操作）。

> [!NOTE]
>
> Databricks 管理员在工作区中的所有机密范围内都有 MANAGE 权限。

## <a name="create-a-secret-acl"></a>创建机密 ACL

若要使用 [Databricks CLI](../../dev-tools/cli/index.md)（版本 0.7.1 及更高版本）为给定机密范围创建机密 ACL，请执行以下语句：

```bash
databricks secrets put-acl --scope <scope-name> --principal <principal> --permission <permission>
```

对已经有一个应用的权限的主体发出 put 请求会覆盖现有权限级别。

## <a name="view-secret-acls"></a>查看机密 ACL

若要查看给定机密范围的所有机密 ACL，请执行以下命令：

```bash
databricks secrets list-acls --scope <scope-name>
```

若要获取应用于给定机密范围的主体的机密 ACL，请执行以下命令：

```bash
databricks secrets get-acl --scope <scope-name> --principal <principal>
```

如果给定主体和范围不存在 ACL，此请求会失败。

## <a name="delete-a-secret-acl"></a>删除机密 ACL

若要删除应用于给定机密范围的主体的机密 ACL，请执行以下命令：

```bash
databricks secrets delete-acl --scope <scope-name> --principal <principal>
```