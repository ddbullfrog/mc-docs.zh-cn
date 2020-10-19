---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: 使用 Azure Databricks 个人访问令牌进行身份验证 - Azure Databricks
description: 了解如何使用 Azure Databricks 个人访问令牌进行身份验证并访问 Databricks REST API。
ms.openlocfilehash: 68599798cbc820b4bc12b668c0418a5d42544adf
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937792"
---
# <a name="authentication-using-azure-databricks-personal-access-tokens"></a>使用 Azure Databricks 个人访问令牌进行身份验证

若要对 Databricks REST API 进行身份验证和访问，可使用 Azure Databricks 个人访问令牌或 Azure Active Directory (Azure AD) 令牌。

本文介绍如何使用 Azure Databricks 个人访问令牌。 有关 Azure AD 令牌，请参阅[使用 Azure Active Directory 令牌进行身份验证](aad/index.md)。

> [!IMPORTANT]
>
> 令牌取代了身份验证流中的密码；与密码一样，应始终谨慎使用令牌。
> 为了保护令牌，Databricks 建议将令牌存储到：
>
> * [机密管理](../../../security/secrets/index.md)，并使用[机密实用程序](../../databricks-utils.md#dbutils-secrets)在笔记本中检索令牌。
> * 本地密钥存储，并使用 [Python keyring](https://pypi.org/project/keyring/) 包在运行时检索令牌。

## <a name="requirements"></a>要求

2018 年 1 月之后启动的所有 Azure Databricks 帐户默认启用基于令牌的身份验证。 如果已禁用，你的管理员必须先启用它，然后你才能执行本文中所述的任务。
请参阅[管理个人访问令牌](../../../administration-guide/access-control/tokens.md)。

## <a name="generate-a-personal-access-token"></a><a id="generate-a-personal-access-token"> </a><a id="token-management"> </a>生成个人访问令牌

本部分介绍如何在 Azure Databricks UI 中生成个人访问令牌。
还可使用[令牌 API](tokens.md)生成和撤销令牌。

1. 单击“用户配置文件”图标 ![用户配置文件](../../../_static/images/account-settings/account-icon.png) 它位于 Azure Databricks 工作区的右上角。
2. 单击“用户设置”。
3. 转到“访问令牌”选项卡。

   ![List_Tokens](../../../_static/images/tokens/list-tokens.png)

4. 单击“生成新令牌”按钮。
5. 选择性地输入说明（注释）和有效期。

   ![Generate_Token](../../../_static/images/tokens/generate-token.png)

6. 单击“生成”按钮。
7. 复制生成的令牌，并将其存储在安全的位置。

## <a name="revoke-a-personal-access-token"></a>撤销个人访问令牌

本部分介绍如何使用 Azure Databricks UI 撤销个人访问令牌。
还可使用[令牌 API](tokens.md) 生成和撤销访问令牌。

1. 单击“用户配置文件”图标 ![用户配置文件](../../../_static/images/account-settings/account-icon.png) 它位于 Azure Databricks 工作区的右上角。
2. 单击“用户设置”。
3. 转到“访问令牌”选项卡。
4. 针对要撤销的令牌单击 x。
5. 在“撤销令牌”对话框中，单击“撤销令牌”按钮。

## <a name="use-a-personal-access-token-to-access-the-databricks-rest-api"></a><a id="netrc"> </a><a id="use-a-personal-access-token-to-access-the-databricks-rest-api"> </a>使用个人访问令牌访问 Databricks REST API

可在 `.netrc` 中存储个人访问令牌并在 `curl` 中使用，也可将其传递到 `Authorization: Bearer` 标头。

### <a name="store-token-in-netrc-file-and-use-in-curl"></a>在 `.netrc` 文件中存储令牌并在 `curl` 中使用

使用 `machine`、`login` 和 `password` 属性创建 [.netrc](https://ec.haxx.se/usingcurl-netrc.html) 文件：

```ini
machine <databricks-instance>
login token
password <personal-access-token>
```

其中：

* `<databricks-instance>` 是 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。
* `token` 是文本字符串 `token`
* `<personal-access-token>` 是个人访问令牌的值。

若要调用 `.netrc` 文件，请在 `curl` 命令中使用 `-n`：

```bash
curl -n -X GET https://<databricks-instance>/api/2.0/clusters/list
```

### <a name="pass-token-to-bearer-authentication"></a><a id="bearer"> </a><a id="pass-token-to-bearer-authentication"> </a>将令牌传递到 `Bearer` 身份验证

可使用 `Bearer` 身份验证将令牌包含在标头中， 也可将此方法用于 `curl` 或你构建的任何客户端。 如果是后者，请参阅[将大文件上传到 DBFS](examples.md#dbfs-large-files)。

```bash
curl -X GET -H 'Authorization: Bearer <personal-access-token>' https://<databricks-instance>/api/2.0/clusters/list
```