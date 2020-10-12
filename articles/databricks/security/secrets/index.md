---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: 机密管理 - Azure Databricks
description: 了解如何使用 Azure Databricks 机密来存储凭据，以通过 JDBC 对外部数据源进行身份验证。
ms.openlocfilehash: 061c5990f8445b58a3bf0d10b692bffc44c83659
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937737"
---
# <a name="secret-management"></a><a id="secret-management"> </a><a id="secrets-user-guide"> </a>机密管理

有时，访问数据需要通过 JDBC 对外部数据源进行身份验证。
如果不直接在笔记本中输入凭据，可以使用 Azure Databricks 机密来存储凭据并在笔记本和作业中引用这些凭据。 若要管理机密，可以使用 [Databricks CLI](../../dev-tools/cli/index.md) 访问[机密 API](../../dev-tools/api/latest/secrets.md)。

若要设置机密，请执行以下操作：

1. [创建机密范围](secret-scopes.md)。 机密范围名称不区分大小写。
2. [将机密添加到范围](secrets.md)。 机密名称不区分大小写。
3. 如果你有 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)，请将[访问控制分配](../access-control/secret-acl.md)到机密范围。

本指南介绍如何执行这些设置任务以及管理机密。 有关详细信息，请参阅：

* 演示如何在工作流中使用机密的端到端[示例](example-secret-workflow.md)。
* [机密 CLI](../../dev-tools/cli/secrets-cli.md) 参考。
* [机密 API](../../dev-tools/api/latest/secrets.md) 参考。
* 如何使用[机密实用工具](../../dev-tools/databricks-utils.md#dbutils-secrets)在笔记本和作业中引用机密。

**本指南内容：**

* [机密范围](secret-scopes.md)
  * [概述](secret-scopes.md#overview)
  * [创建 Azure Key Vault 支持的机密范围](secret-scopes.md#create-an-azure-key-vault-backed-secret-scope)
  * [创建 Databricks 支持的机密范围](secret-scopes.md#create-a-databricks-backed-secret-scope)
  * [列出机密范围](secret-scopes.md#list-secret-scopes)
  * [删除机密范围](secret-scopes.md#delete-a-secret-scope)
* [机密](secrets.md)
  * [创建机密](secrets.md#create-a-secret)
  * [列出机密](secrets.md#list-secrets)
  * [读取机密](secrets.md#read-a-secret)
  * [删除机密](secrets.md#delete-a-secret)
* [机密编修](redaction.md)
* [机密工作流示例](example-secret-workflow.md)
  * [创建机密范围](example-secret-workflow.md#create-a-secret-scope)
  * [创建机密](example-secret-workflow.md#create-secrets)
  * [在笔记本中使用机密](example-secret-workflow.md#use-the-secrets-in-a-notebook)
  * [授予对另一个组的访问权限](example-secret-workflow.md#grant-access-to-another-group)