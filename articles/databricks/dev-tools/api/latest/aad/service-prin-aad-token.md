---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/10/2020
title: 使用服务主体获取 Azure Active Directory 令牌 - Azure Databricks
description: 了解如何使用 Azure Active Directory 服务主体对 Databricks REST API 进行身份验证。
ms.openlocfilehash: 820284ae79ed18a9d88513bf6c39b4dc7fcadad7
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937796"
---
# <a name="get-an-azure-active-directory-token-using-a-service-principal"></a>使用服务主体获取 Azure Active Directory 令牌

本文介绍在 Azure Active Directory (Azure AD) 中定义的服务主体如何还可以充当在 Azure Databricks 中强制执行身份验证和授权策略的主体。 Azure Databricks 工作区中的服务主体可以有不同于常规用户（用户主体）的精细访问控制。

[服务主体](/active-directory/develop/app-objects-and-service-principals)充当[客户端角色](/active-directory/develop/developer-glossary#client-application)，并使用 [OAuth 2.0 代码授权流](/active-directory/azuread-dev/v1-protocols-oauth-code)来授权 Azure Databricks 资源。

可以使用 Databricks [SCIM API (ServicePrincipals)](../scim/scim-sp.md) API 管理服务主体，也可以在 Azure 门户中使用以下过程。

还可以使用 Azure Active Directory 身份验证库 (ADAL) 以编程方式为用户获取 Azure AD 访问令牌。 请参阅[使用 Azure Active Directory 身份验证库获取 Azure Active Directory 令牌](app-aad-token.md)。

## <a name="provision-a-service-principal-in-azure-portal"></a><a id="provision-a-service-principal-in-azure-portal"> </a><a id="register-sp"> </a>在 Azure 门户中预配服务主体

1. 登录到 Azure 门户。
2. 导航到“Azure Active Directory”>“应用注册”>“新建注册”。 应会显示如下所示的屏幕：

   > [!div class="mx-imgBorder"]
   > ![注册应用](../../../../_static/images/aad/register-app.png)

3. 单击“证书和密码”，然后生成新的客户端密码。

   > [!div class="mx-imgBorder"]
   > ![注册应用](../../../../_static/images/aad/copy-secret.png)

4. 复制此密码并将其存储在安全位置，因为此密码是应用程序的密码。
5. 单击“概述”，以查看应用程序（客户端）ID 和目录（租户）ID 等详细信息。

[使用应用标识访问资源](https://docs.microsoft.com/azure-stack/operator/azure-stack-create-service-principals?view=azs-2002)介绍如何在 Azure AD 中预配应用程序（服务主体）。

## <a name="get-an-azure-active-directory-access-token"></a><a id="get-an-azure-active-directory-access-token"> </a><a id="get-token"> </a>获取 Azure Active Directory 访问令牌

如果要使用服务主体访问 Databricks REST API，需要获取服务主体的 Azure AD 访问令牌。 可以使用[客户端凭据流](/active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow)获取访问令牌（使用 AzureDatabricks 登录应用程序作为资源）。

在 `curl` 请求中，替换以下参数：

| 参数                                      | 说明                                                                                                                                |
|------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| 租户 ID                                      | Azure AD 中的租户 ID。 转到“Azure Active Directory”>“属性”>“目录 ID”。                                                       |
| 客户端 ID                                      | 在 [Azure 门户预配服务主体](#register-sp)中注册的应用程序的应用程序（服务主体）ID。 |
| Azure Databricks 资源 ID                   | `2ff814a6-3304-4ab8-85cb-cd0e6f879c1d`.                                                                                                    |
| 应用程序密码                             | 为应用程序生成的机密。                                                                                                  |

```bash
curl -X GET -H 'Content-Type: application/x-www-form-urlencoded' \
-d 'grant_type=client_credentials&client_id=<client-id>&resource=<azure_databricks_resource_id>&client_secret=<application-secret>' \
https://login.microsoftonline.com/<tenant-id>/oauth2/token
```

响应应如下所示：

```json
{
  "token_type": "Bearer",
  "expires_in": "599",
  "ext_expires_in": "599",
  "expires_on": "1575500666",
  "not_before": "1575499766",
  "resource": "2ff8...f879c1d",
  "access_token": "ABC0eXAiOiJKV1Q......un_f1mSgCHlA"
}
```

响应中的 `access_token` 是 Azure AD 访问令牌。

## <a name="use-an-azure-ad-access-token-to-access-the-databricks-rest-api"></a><a id="use-an-azure-ad-access-token-to-access-the-databricks-rest-api"> </a><a id="use-token"> </a>使用 Azure AD 访问令牌访问 Databricks REST API

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[每工作区 URL](../../../../workspace/workspace-details.md#per-workspace-url)。

### <a name="admin-user-login"></a>管理员用户登录

如果满足以下任一条件，那么必须是 Azure 中工作区资源的“参与者”或“所有者”角色，才能使用服务主体访问令牌登录：

* 服务主体不属于工作区。
* 服务主体属于工作区，但你希望以管理员用户身份进行自动添加。
* 你不知道工作区的组织 ID，但知道 Azure 中的工作区资源 ID。

必须提供：

* `X-Databricks-Azure-Workspace-Resource-Id` 标头，包含 Azure 中工作区资源的 ID。 使用 Azure 订阅 ID、资源组名称以及工作区资源名称构造 ID。
* Azure 资源管理终结点的管理访问令牌。

#### <a name="get-the-azure-management-resource-endpoint-token"></a><a id="get-management-token"> </a><a id="get-the-azure-management-resource-endpoint-token"> </a>获取 Azure 管理资源终结点令牌

在 `curl` 请求中，替换以下参数：

| 参数                     | 说明                                                                                                                                |
|-------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| 租户 ID                     | Azure AD 中的租户 ID。  转到“Azure Active Directory”>“属性”>“目录 ID”。                                                      |
| 客户端 ID                     | 在 [Azure 门户预配服务主体](#register-sp)中注册的应用程序的应用程序（服务主体）ID。 |
| 管理资源终结点  | `https://management.core.chinacloudapi.cn/`.                                                                                                    |
| 应用程序密码            | 为应用程序生成的机密。                                                                                                  |

```bash
curl -X GET -H 'Content-Type: application/x-www-form-urlencoded' \
-d 'grant_type=client_credentials&client_id=<client-id>&resource=<management-resource-endpoint>&client_secret=<application-secret>' \
https://login.microsoftonline.com/<tenantid>/oauth2/token
```

响应应如下所示：

```json
{
  "token_type": "Bearer",
  "expires_in": "599",
  "ext_expires_in": "599",
  "expires_on": "1575500666",
  "not_before": "1575499766",
  "resource": "https://management.core.chinacloudapi.cn/",
  "access_token": "LMN0eXAiOiJKV1Q......un_f1mSgCHlA"
}
```

响应中的 `access_token` 是管理终结点访问令牌。

#### <a name="use-the-management-endpoint-access-token-to-access-the-databricks-rest-api"></a>使用管理终结点访问令牌访问 Databricks REST API

| 参数                 | 描述                                                                                                             |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------|
| 访问令牌              | 在[获取 Azure Active Directory 访问令牌](#get-token)中获取的访问令牌。                                      |
| 管理访问令牌   | 在[获取 Azure 管理资源终结点令牌](#get-management-token)中获取的管理终结点访问令牌。 |
| 订阅 ID           | Azure Databricks 资源的订阅 ID。                                                                       |
| 资源组名称       | Azure Databricks 资源组的名称。                                                                            |
| 工作区名称            | Azure Databricks 工作区的名称。                                                                                 |

```bash
curl -X GET \
-H 'Authorization: Bearer <access-token>' \
-H 'X-Databricks-Azure-SP-Management-Token: <management-access-token>' \
-H 'X-Databricks-Azure-Workspace-Resource-Id: /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Databricks/workspaces/<workspace-name>' \
https://<databricks-instance>/api/2.0/clusters/list
```

示例请求如下所示：

```bash
curl -X GET \
-H 'Authorization:Bearer ABC0eXAiOiJKV1Q......un_f1mSgCHlA' \
-H 'X-Databricks-Azure-SP-Management-Token: LMN0eXAiOiJKV1Q......un_f1mSgCHlA' \
-H 'X-Databricks-Azure-Workspace-Resource-Id: /subscriptions/3f2e4d...2328b/resourceGroups/Ene...RG/providers/Microsoft.Databricks/workspaces/demo-databricks' \
https://<databricks-instance>/api/2.0/clusters/list
```

### <a name="non-admin-user-login"></a>非管理员用户登录

> [!NOTE]
>
> 在此登录之前，必须将服务主体作为[管理员用户登录](#admin-user-login)的一部分添加到工作区，或使用[添加服务主体](../scim/scim-sp.md#add-service-principal)终结点。

使用访问令牌作为 `Bearer` 令牌。

| 参数                                      | 描述                                                                                  |
|------------------------------------------------|----------------------------------------------------------------------------------------------|
| 访问令牌                                   | 从[获取 Azure Active Directory 访问令牌](#get-token)中的请求返回的令牌。 |

#### <a name="use-an-access-token-to-access-the-databricks-rest-api"></a>使用访问令牌访问 Databricks REST API

```bash
curl -X GET \
-H 'Authorization: Bearer <access-token>' \
https://<databricks-instance>/api/2.0/clusters/list
```