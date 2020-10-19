---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/07/2020
title: 机密范围 - Azure Databricks
description: 了解如何创建和管理 Azure Databricks 的两种机密范围类型（Azure Key Vault 支持的范围和 Databricks 支持的范围），并对机密范围使用最佳做法。
ms.openlocfilehash: 6b05a28a5caed31a66db3e82514781e43f4754ab
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937741"
---
# <a name="secret-scopes"></a>机密范围

管理机密时首先需要创建机密范围。 机密范围是由名称标识的机密的集合。  一个工作区最多只能有 100 个机密范围。

## <a name="overview"></a>概述

有两种类型的机密范围：Azure Key Vault 支持和 Databricks 支持。

### <a name="azure-key-vault-backed-scopes"></a>Azure Key Vault 支持的范围

若要引用 [Azure Key Vault](/key-vault/key-vault-overview) 中存储的机密，可以创建 Azure Key Vault 支持的机密范围。 然后，你可以利用该机密范围中相应 Key Vault 实例中的所有机密。 由于 Azure Key Vault 支持的机密范围是 Key Vault 的只读接口，因此不允许进行 `PutSecret` 和 `DeleteSecret` [机密 API](../../dev-tools/api/latest/secrets.md) 操作。 若要在 Azure Key Vault 中管理机密，必须使用 Azure [SetSecret](https://docs.microsoft.com/rest/api/keyvault/setsecret) REST API 或 Azure 门户 UI。

### <a name="databricks-backed-scopes"></a>Databricks 支持的范围

Databricks 支持的机密范围存储在 Azure Databricks 拥有并管理的加密数据库中并由该数据库支持。 机密范围名称：

* 在工作区中必须唯一。
* 必须包含字母数字字符、短划线、下划线和句点，并且不得超过 128 个字符。

这些名称被视为是非敏感信息，工作区中的所有用户都可读取它们。

使用 [Databricks CLI](../../dev-tools/cli/index.md)（版本 0.7.1 及更高版本）创建 Databricks 支持的机密范围。 也可以使用[机密 API](../../dev-tools/api/latest/secrets.md)。

### <a name="scope-permissions"></a>范围权限

使用 [ACL](../access-control/secret-acl.md) 控制的权限创建范围。 默认情况下，使用创建范围的用户（创建者）的 `MANAGE` 权限创建范围，这使创建者可以读取范围中的机密、将机密写入范围以及更改范围的 ACL。 如果你的帐户具有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，则可以在创建范围后随时分配细粒度权限。 有关详细信息，请参阅[机密访问控制](../access-control/secret-acl.md)。

你还可以替代默认值，并在创建范围时向所有用户显式授予 `MANAGE` 权限。 事实上，如果你的帐户不具有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，则必须执行此操作。

### <a name="best-practices"></a><a id="best-practices"> </a><a id="databricks-backed"> </a>最佳做法

作为团队主管，你可能想要为 Azure Synapse Analytics 和 Azure Blob 存储凭据创建不同的范围，然后在团队中提供不同的子组来访问这些范围。 你应考虑如何使用不同的范围类型来实现此目的：

* 如果你使用 Databricks 支持的范围并在这两个范围中添加机密，它们将是不同的机密（Azure Synapse Analytics 在范围 1 中，Azure Blob 存储在范围 2 中）。
* 如果你使用 Azure Key Vault 支持的范围，每个范围都引用不同的 Azure Key Vault，并将机密添加到这两个 Azure Key Vault，则它们将是不同的机密集（Azure Synapse Analytics 在范围 1 中，Azure Blob 存储在范围 2 中）。 这些范围的工作方式类似于 Databricks 支持的范围。
* 如果你使用两个 Azure Key Vault 支持的范围，且两个范围都引用同一个 Azure Key Vault，并将机密添加到该 Azure Key Vault，则所有 Azure Synapse Analytics 和 Azure Blob 存储机密都将可用。 由于 ACL 处于范围级别，因此这两个子组中的所有成员都可看到所有机密。 这种安排并不满足你限制每个组访问一组机密的用例。

## <a name="create-an-azure-key-vault-backed-secret-scope"></a><a id="akv-ss"> </a><a id="create-an-azure-key-vault-backed-secret-scope"> </a>创建 Azure Key Vault 支持的机密范围

可以使用 UI 或 Databricks CLI 创建 Azure Key Vault 支持的机密范围。

### <a name="create-an-azure-key-vault-backed-secret-scope-using-the-ui"></a>使用 UI 创建 Azure Key Vault 支持的机密范围

1. 验证你是否对要用于支持机密范围的 Azure Key Vault 实例具有“参与者”权限。

   如果没有 Key Vault 实例，请按照[快速入门：使用 Azure 门户创建 Key Vault](/key-vault/quick-create-portal) 中的说明进行操作。

2. 转到 `https://<databricks-instance>#secrets/createScope`。 此 URL 区分大小写；`createScope` 中的范围必须大写。

   > [!div class="mx-imgBorder"]
   > ![创建范围](../../_static/images/secrets/azure-kv-scope.png)

3. 输入机密范围的名称。 机密范围名称不区分大小写。
4. 使用“管理主体”下拉列表指定是所有用户都对此机密范围具有 `MANAGE` 权限，还是仅机密范围的创建者具有该权限 。

   `MANAGE` 权限允许用户在此机密范围内进行读取和写入，如果是 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)中的帐户，还允许更改范围的权限。

   你的帐户必须具有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，你才能选择“创建者”。 建议的做法是：在创建机密范围时向“创建者”授予 `MANAGE` 权限，然后在测试范围后分配更细粒度的访问权限。 有关示例工作流的信息，请参阅[机密工作流示例](example-secret-workflow.md)。

   如果你的帐户具有标准计划，则必须将 `MANAGE` 权限设置为“所有用户”组。  如果在此处选择“创建者”，则在尝试保存该范围时，将看到一条错误消息。

   有关 `MANAGE` 权限的详细信息，请参阅[机密访问控制](../access-control/secret-acl.md)。

5. 输入“DNS 名称”（例如 `https://databrickskv.vault.azure.net/`）和“资源 ID”，例如 ：

   ```
   /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/databricks-rg/providers/Microsoft.KeyVault/vaults/databricksKV
   ```

   可从 Azure 门户中 Azure Key Vault 的“属性”选项卡中使用这些属性。

   > [!div class="mx-imgBorder"]
   > ![Azure Key Vault 的“属性”选项卡](../../_static/images/secrets/azure-kv.png)

6. 单击“创建”  按钮。
7. 使用 [Databricks CLI](../../dev-tools/cli/index.md) `databricks secrets list-scopes` 命令验证是否已成功创建范围。

有关访问 Azure Blob 存储时使用机密的示例，请参阅[装载 Azure Blob 存储容器](../../data/data-sources/azure/azure-storage.md#mount-azure-blob)。

### <a name="create-an-azure-key-vault-backed-secret-scope-using-the-databricks-cli"></a>使用 Databricks CLI 创建 Azure Key Vault 支持的机密范围

1. 安装 Databricks CLI 并将其配置为使用 Azure Active Directory (Azure AD) 令牌进行身份验证。

   请参阅[安装 CLI](../../dev-tools/cli/index.md#install-the-cli)。

2. 创建 Azure Key Vault 范围：

   ```bash
   databricks secrets create-scope --scope <scope-name>    --scope-backend-type AZURE_KEYVAULT --subscription-id <azure-keyvault-subscription-id> --dns-name <azure-keyvault-dns-name>
   ```

   默认情况下，使用创建范围的用户的 `MANAGE` 权限创建范围。 如果你的帐户没有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，则必须替代此默认值，并在创建范围时向 `users`（所有用户）组显式授予 `MANAGE` 权限：

   ```bash
    databricks secrets create-scope --scope <scope-name>    --scope-backend-type AZURE_KEYVAULT --subscription-id <azure-keyvault-subscription-id> --dns-name <azure-keyvault-dns-name> --initial-manage-principal users
   ```

   如果你的帐户具有 Azure Databricks 高级计划，则可以在创建范围后随时更改权限。 有关详细信息，请参阅[机密访问控制](../access-control/secret-acl.md)。

   创建 Databricks 支持的机密范围后，可以[添加机密](secrets.md)。

有关访问 Azure Blob 存储时使用机密的示例，请参阅[装载 Azure Blob 存储容器](../../data/data-sources/azure/azure-storage.md#mount-azure-blob)。

## <a name="create-a-databricks-backed-secret-scope"></a>创建 Databricks 支持的机密范围

机密范围名称不区分大小写。

使用 Databricks CLI 创建范围：

```bash
databricks secrets create-scope --scope <scope-name>
```

默认情况下，使用创建范围的用户的 `MANAGE` 权限创建范围。 如果你的帐户没有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，则必须替代此默认值，并在创建范围时向“用户”（所有用户）组显式授予 `MANAGE` 权限：

```bash
databricks secrets create-scope --scope <scope-name> --initial-manage-principal users
```

如果你的帐户具有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，则可以在创建范围后随时更改权限。 有关详细信息，请参阅[机密访问控制](../access-control/secret-acl.md)。

创建 Databricks 支持的机密范围后，可以[添加机密](secrets.md)。

## <a name="list-secret-scopes"></a><a id="list-scopes"> </a><a id="list-secret-scopes"> </a>列出机密范围

列出工作区中的现有范围：

```bash
databricks secrets list-scopes
```

## <a name="delete-a-secret-scope"></a>删除机密范围

删除机密范围时会删除应用于该范围的所有机密和 ACL。 删除范围：

```bash
databricks secrets delete-scope --scope <scope-name>
```