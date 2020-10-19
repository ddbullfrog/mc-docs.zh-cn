---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/07/2020
title: Databricks CLI - Azure Databricks
description: 了解如何安装和配置用于运行 Databricks 命令行界面的环境。
ms.openlocfilehash: 8efe14b11d2f85f068231eb8caf370325dddb03c
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937639"
---
# <a name="databricks-cli"></a>Databricks CLI

Databricks 命令行界面 (CLI) 提供了针对 Azure Databricks 平台的易用界面。 此开放源代码项目承载在 [GitHub](https://github.com/databricks/databricks-cli) 上。 此 CLI 在 Databricks [REST API 2.0](../api/latest/index.md#rest-api-v2) 的基础上构建，根据[工作区 API](../api/latest/workspace.md)[群集 API](../api/latest/clusters.md)、[实例池 API](../api/latest/instance-pools.md)、[DBFS API](../api/latest/dbfs.md)、[组 API](../api/latest/groups.md)、[作业 API](../api/latest/jobs.md)、[库 API](../api/latest/libraries.md) 和 [机密 API](../api/latest/secrets.md) 整理到命令组中：`workspace`、`clusters`、`instance-pools`、`fs`、`groups`、`jobs`、`runs`、`libraries` 和 `secrets`。

> [!IMPORTANT]
>
> 我们正积极开发此 CLI，将以[试验](../../release-notes/release-types.md)客户端的形式发布它。 这意味着，相关界面仍可能会变化。

## <a name="set-up-the-cli"></a>设置 CLI

此部分列出了 CLI 的要求，还介绍了如何安装和配置用于运行 CLI 的环境。

### <a name="requirements"></a>要求

* **Python 3** - 3.6 及更高版本
* **Python 2** - 2.7.9 及更高版本

  > [!IMPORTANT]
  >
  > 在 MacOS 上，默认的 Python 2 安装未实现 TLSv1_2 协议。将 CLI 与此 Python 安装一起运行会导致以下错误：`AttributeError: 'module' object has no attribute 'PROTOCOL_TLSv1_2'`。 使用 [Homebrew](https://brew.sh/) 来安装具有 `ssl.PROTOCOL_TLSv1_2` 的 Python 版本。

### <a name="limitations"></a>限制

不支持将 Databricks CLI 用于启用了防火墙的存储容器。 Databricks 建议使用 [Databricks Connect](../databricks-connect.md) 或 [az storage](https://docs.microsoft.com/cli/azure/storage?view=azure-cli-latest)。

### <a name="install-the-cli"></a>安装 CLI

请使用与 Python 安装相对应的 `pip` 版本运行 `pip install databricks-cli`。

### <a name="set-up-authentication"></a><a id="cli-auth"> </a><a id="set-up-authentication"> </a>设置身份验证

在运行 CLI 命令之前，必须设置身份验证。 若要向 CLI 进行身份验证，可使用 [Databricks 个人访问令牌](../api/latest/authentication.md)或 [Azure Active Directory (Azure AD) 令牌](../api/latest/aad/index.md)。

#### <a name="set-up-authentication-using-an-azure-ad-token"></a>使用 Azure AD 令牌设置身份验证

若要使用 Azure AD 令牌配置 CLI，请[生成 Azure AD 令牌](../api/latest/aad/index.md)并将它存储在环境变量 `DATABRICKS_AAD_TOKEN` 中。

```bash
export DATABRICKS_AAD_TOKEN=<azure-ad-token>
```

运行 `databricks configure --aad-token`。 此命令发出提示：

```console
Databricks Host (should begin with https://):
```

输入每工作区 URL（格式为 `adb-<workspace-id>.<random-number>.databricks.azure.cn`）。若要获取每工作区 URL，请参阅[每工作区 URL](../../workspace/workspace-details.md#per-workspace-url)。

按提示操作后，访问凭据会存储在 `~/.databrickscfg` 文件中。 此文件应包含如下所示条目：

```console
host = https://<databricks-instance>
token =  <azure-ad-token>
```

#### <a name="set-up-authentication-using-a-databricks-personal-access-token"></a>使用 Databricks 个人访问令牌设置身份验证

若要将 CLI 配置为使用个人访问令牌，请运行 `databricks configure --token`。 此命令发出提示：

```console
Databricks Host (should begin with https://):
Token:
```

完成提示后，访问凭据会存储在 `~/.databrickscfg` 文件中。 此文件应包含如下所示条目：

```console
host = https://<databricks-instance>
token =  <personal-access-token>
```

对于 CLI 0.8.1 及更高版本，可以通过设置环境变量 `DATABRICKS_CONFIG_FILE` 来更改此文件的路径。

> [!IMPORTANT]
>
> 由于 CLI 在 REST API 基础上构建，因此 [.netrc 文件](../api/latest/authentication.md#netrc)中的身份验证配置优先于 `.databrickscfg` 中的配置。

CLI 0.8.0 及更高版本支持以下环境变量：

* `DATABRICKS_HOST`
* `DATABRICKS_TOKEN`

环境变量设置优先于配置文件中的设置。

### <a name="connection-profiles"></a><a id="cli-profile"> </a><a id="connection-profiles"> </a>连接配置文件

Databricks CLI 配置支持多个连接配置文件。 同一 Databricks CLI 安装可以用来在多个 Azure Databricks 工作区进行 API 调用。

若要添加连接配置文件，请执行以下命令：

```bash
databricks configure [--profile <profile>]
```

若要使用连接配置文件，请执行以下命令：

```bash
databricks workspace ls --profile <profile>
```

### <a name="alias-command-groups"></a><a id="alias-command-groups"> </a><a id="alias_databricks_cli"> </a>Alias 命令组

有时候，使用命令组的名称作为每个 CLI 调用的前缀并不方便，例如 `databricks workspace ls`。 若要使 CLI 更易于使用，可以通过 alias 命令组来使用较短的命令。
例如，若要在 Bourne again shell 中将 `databricks workspace ls` 缩写为 `dw ls`，可以将 `alias dw="databricks workspace"` 添加到相应的 bash 配置文件。 通常，该文件位于 `~/.bash_profile`。

> [!TIP]
>
> Azure Databricks 已将 `databricks fs` 的别名设置为 `dbfs`；`databricks fs ls` 和 `dbfs ls` 等效。

## <a name="use-the-cli"></a>使用 CLI

此部分介绍如何获取 CLI 帮助、如何分析 CLI 输出，以及如何调用每个命令组中的命令。

### <a name="display-cli-command-group-help"></a>显示 CLI 命令组帮助

可以通过运行 `databricks <group> -h` 列出任意命令组的子命令。 例如，可以通过运行 `databricks fs -h` 列出 DBFS CLI 子命令。

### <a name="use-jq-to-parse-cli-output"></a><a id="jq"> </a><a id="use-jq-to-parse-cli-output"> </a>使用 `jq` 分析 CLI 输出

某些 Databricks CLI 命令从 API 终结点输出 JSON 响应。 有时候，可以分析将要通过管道传输到其他命令中的 JSON 部件。 例如，若要复制作业定义，必须获取 `/api/2.0/jobs/get` 的 `settings` 字段并将其用作 `databricks jobs create` 命令的参数。

在这些情况下，建议使用实用程序 `jq`。 可以将 Homebrew 与 `brew install jq` 配合使用，以便在 MacOS 上安装 `jq`。

有关 `jq` 的详细信息，请参阅 [jq 手册](https://stedolan.github.io/jq/manual/)。

### <a name="json-string-parameters"></a>JSON 字符串参数

字符串参数的处理方式各异，具体取决于你的操作系统：

* **Unix**：必须将 JSON 字符串参数用单引号引起来。 例如：

  ```bash
  databricks jobs run-now --job-id 9 --jar-params '["20180505", "alantest"]'
  ```

* Windows：必须将 JSON 字符串参数用双引号引起来，字符串内的引号字符必须在 `\` 之后。 例如：

  ```bash
  databricks jobs run-now --job-id 9 --jar-params "[\"20180505\", \"alantest\"]"
  ```

## <a name="cli-commands"></a>CLI 命令

* [工作区 CLI](workspace-cli.md)
* [群集 CLI](clusters-cli.md)
* [实例池 CLI](instance-pools-cli.md)
* [DBFS CLI](dbfs-cli.md)
* [组 CLI](groups-cli.md)
* [作业 CLI](jobs-cli.md)
* [库 CLI](libraries-cli.md)
* [机密 CLI](secrets-cli.md)
* [堆栈 CLI](stack-cli.md)