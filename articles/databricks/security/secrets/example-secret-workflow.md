---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 机密工作流示例 - Azure Databricks
description: 了解如何通过创建机密范围、创建机密以及在笔记本中使用它们，使用机密设置用于连接到 Azure Data Lake Store 的 JDBC 凭据。
ms.openlocfilehash: bd94c6bd7526fda30e60689809922bfa519c194a
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937739"
---
# <a name="secret-workflow-example"></a>机密工作流示例

在此工作流示例中，我们使用机密来设置用于连接到 Azure Data Lake Store 的 JDBC 凭据。

## <a name="create-a-secret-scope"></a>创建机密范围

创建名为 `jdbc` 的机密范围。

若要创建 Databricks 支持的机密范围：

```bash
databricks secrets create-scope --scope jdbc
```

若要创建 Azure Key Vault 支持的机密范围，请按照[创建 Azure Key Vault-支持的机密范围](secret-scopes.md#akv-ss)中的说明操作。

> [!NOTE]
>
> 如果你的帐户没有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，则必须创建范围，并将 `MANAGE` 权限授予所有用户（以下称为“用户”）。 例如：
>
> ```bash
> databricks secrets create-scope --scope jdbc --initial-manage-principal users
> ```

## <a name="create-secrets"></a>创建机密

创建机密的方法取决于你使用的是 Azure Key Vault 支持的范围还是 Databricks 支持的范围。

### <a name="create-the-secrets-in-an-azure-key-vault-backed-scope"></a>在 Azure Key Vault 支持的范围创建机密

使用 Azure [SetSecret](https://docs.microsoft.com/rest/api/keyvault/setsecret) REST API 或 Azure 门户 UI 添加机密 `username` 和 `password`：

> [!div class="mx-imgBorder"]
> ![将机密添加到 Azure Key Vault](../../_static/images/secrets/azure-kv-secrets.png)

### <a name="create-the-secrets-in-a-databricks-backed-scope"></a>在 Databricks 支持的范围内创建机密

添加机密 `username` 和 `password`。 运行以下命令，然后在打开的编辑器中输入机密值。

```bash
databricks secrets put --scope jdbc --key username
databricks secrets put --scope jdbc --key password
```

## <a name="use-the-secrets-in-a-notebook"></a><a id="secret-example-notebook"> </a><a id="use-the-secrets-in-a-notebook"> </a>在笔记本中使用机密

在笔记本中，读取存储在机密范围 `jdbc` 中的机密以配置 JDBC 连接器：

```scala
val driverClass = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
val connectionProperties = new java.util.Properties()
connectionProperties.setProperty("Driver", driverClass)

val jdbcUsername = dbutils.secrets.get(scope = "jdbc", key = "username")
val jdbcPassword = dbutils.secrets.get(scope = "jdbc", key = "password")
connectionProperties.put("user", s"${jdbcUsername}")
connectionProperties.put("password", s"${jdbcPassword}")
```

现在，可以将这些 `ConnectionProperties` 与 JDBC 连接器结合使用，以便与你的数据源通信。
从该范围中获取的值永远不会显示在笔记本中（请参阅[机密编辑](redaction.md)）。

## <a name="grant-access-to-another-group"></a>授予对另一个组的访问权限

> [!NOTE]
>
> 此步骤要求你的帐户拥有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)。

验证凭据配置正确后，请与 `datascience` 组共享这些凭据，以用于分析。

通过发出以下请求，向 `datascience` 组授予对这些凭据的只读权限：

```bash
databricks secrets put-acl --scope jdbc --principal datascience --permission READ
```