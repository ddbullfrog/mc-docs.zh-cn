---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/24/2020
title: 机密 - Azure Databricks
description: 了解如何创建和管理机密，这些机密是存储机密材料的键值对。
ms.openlocfilehash: d84399e1606a97fcee5272e5f1f4c86b4c4c11f1
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937772"
---
# <a name="secrets"></a>机密

机密是一种键值对，用于存储机密材料，它具有一个密钥名称，且在[机密范围](secret-scopes.md)中是唯一的。 每个范围限制为 1000 个机密。 允许的最大机密值大小为 128 KB。

## <a name="create-a-secret"></a>创建机密

机密名称不区分大小写。

创建机密的方法取决于使用的是 Azure Key Vault 支持的范围还是 Databricks 支持的范围。

### <a name="create-a-secret-in-an-azure-key-vault-backed-scope"></a>在 Azure Key Vault 支持的范围创建机密

若要在 Azure Key Vault 中创建机密，请使用 Azure [SetSecret](https://docs.microsoft.com/rest/api/keyvault/setsecret) REST API 或 Azure 门户 UI。

> [!div class="mx-imgBorder"]
> ![Azure Key Vault](../../_static/images/secrets/azure-kv-secrets.png)

### <a name="create-a-secret-in-a-databricks-backed-scope"></a>在 Databricks 支持的范围中创建机密

若要使用 [Databricks CLI](../../dev-tools/cli/index.md)（版本 0.7.1 及更高版本）在 Databricks 支持的范围中创建机密：

```bash
databricks secrets put --scope <scope-name> --key <key-name>
```

此时将打开一个编辑器，其中显示如下内容：

```
# ----------------------------------------------------------------------
# Do not edit the above line. Everything that follows it will be ignored.
# Please input your secret value above the line. Text will be stored in
# UTF-8 (MB4) form and any trailing new line will be stripped.
# Exit without saving will abort writing secret.
```

将机密值粘贴到行上方，然后保存并退出编辑器。 你的输入将剥除注释并与范围中的键关联存储。

如果你使用已存在的密钥发出写入请求，则新值将覆盖现有值。

还可以从文件或命令行提供机密。 有关写入机密的详细信息，请参阅[机密 CLI](../../dev-tools/cli/secrets-cli.md)。

## <a name="list-secrets"></a>列出机密

列出给定范围内的机密：

```bash
databricks secrets list --scope <scope-name>
```

响应会显示有关机密的元数据信息，如机密密钥名称和最后一次更新时间戳（自 epoch 起算，以毫秒为单位）。 你可以使用笔记本或作业中的[机密实用程序](../../dev-tools/databricks-utils.md#dbutils-secrets)来读取机密。 例如：

```bash
databricks secrets list --scope jdbc
```

```
Key name    Last updated
----------  --------------
password    1531968449039
username    1531968408097
```

## <a name="read-a-secret"></a>读取机密

你可以使用 REST API 或 CLI 创建机密，但必须使用笔记本或作业中的[机密实用程序](../../dev-tools/databricks-utils.md#dbutils-secrets)来读取机密。

### <a name="secret-paths-in-spark-configuration-properties-and-environment-variables"></a><a id="secret-paths-in-spark-configuration-properties-and-environment-variables"> </a><a id="spark-conf-env-var"> </a>Spark 配置属性和环境变量中的机密路径

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../release-notes/release-types.md)提供。

> [!NOTE]
>
> 在 Databricks Runtime 6.1 及更高版本中可用。

可以在 Spark 配置属性或环境变量中存储机密的路径。 检索到的机密是从笔记本的输出以及 Spark 驱动程序和执行器的日志中修订的。

> [!IMPORTANT]
>
> 机密不是从 stdout 和 stderr 中修正的。 一种解决方法是将 Spark 配置属性 `spark.databricks.acl.needAdminPermissionToViewLogs true` 设置为仅允许拥有管理权限的用户查看 stdout 页面。

#### <a name="requirements-and-limitations"></a>要求和限制

* 群集所有者需要具有机密范围内的读取权限。
* 只有群集所有者才能将路径添加到 Spark 配置或环境变量中的机密，并编辑现有的范围和名称。 所有者使用[放置机密](../../dev-tools/api/latest/secrets.md#secretsecretserviceputsecret) API 更改机密。 必须重新启动群集才能再次提取机密。
* 对群集具有“可管理”权限的用户可以删除机密属性和环境变量。

#### <a name="path-value"></a>路径值

Spark 属性或环境变量路径值的语法必须是 `{{secrets/<scope-name>/<secret-name>}}`。

该值必须以 `{{secrets/` 开头，以 `}}` 结尾。 属性或环境变量的变量部分为：

* `<secret-prop-name>`：Spark 配置中的机密属性的名称。
* `<scope-name>`：机密关联的范围的名称。
* `<secret-name>`：范围中的机密的唯一名称。

> [!NOTE]
>
> * 大括号中不应有空格。 如果有空格，它们将被视为范围或机密名称的一部分。
> * 如果值格式不正确（例如，只有一个左大括号或右大括号），则该值将被视为 Spark 配置属性或环境变量值。

#### <a name="store-the-path-to-a-secret-in-a-spark-configuration-property"></a>在 Spark 配置属性中存储机密的路径

按以下格式指定 [Spark 配置](../../clusters/configure.md#spark-config)中的机密路径：

```ini
spark.<secret-prop-name> <path-value>
```

`spark.<secret-prop-name>` 是一个映射到机密路径的 Spark 配置属性名称。 只要机密属性名称是唯一的，你就可以将多个机密添加到 Spark 配置。

**示例**

```ini
spark.password {{secrets/testScope/testKey1}}
```

若要获取笔记本中的机密并使用它，请运行 `spark.conf.get("spark.<secret-name>")`：

```py
spark.conf.get("spark.password")
```

### <a name="store-the-path-to-a-secret-in-an-environment-variable"></a>在环境变量中存储机密的路径

指定[环境变量](../../clusters/configure.md#environment-variables)中的机密路径，并在[群集范围的初始化脚本](../../clusters/init-scripts.md#environment-variables)中使用它。 无法从在 Spark 中运行的程序访问这些环境变量。

```ini
SPARKPASSWORD=<path-value>
```

若要在初始化脚本中提取机密，请访问 `$SPARKPASSWORD`：

```bash
if [[ $SPARKPASSWORD ]]; then
  use $SPARKPASSWORD
fi
```

## <a name="delete-a-secret"></a>删除机密

若要从 Azure Key Vault 支持的范围中删除机密，请使用 Azure [SetSecret](https://docs.microsoft.com/rest/api/keyvault/setsecret) REST API 或 Azure 门户 UI。