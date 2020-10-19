---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/29/2020
title: SCIM API (ServicePrincipals) - Azure Databricks
description: 了解如何使用 Databricks SCIM API 在 Azure Databricks 中创建、读取、更新和删除服务主体。
ms.openlocfilehash: f8cd0263003b79e7ef79931245d2ae4bd42006a4
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937654"
---
# <a name="scim-api-serviceprincipals"></a>SCIM API (ServicePrincipals)

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../../release-notes/release-types.md)提供。

可以通过 SCIM (ServicePrincipals) 在 Azure Databricks 中管理 Azure Active Directory [服务主体](/active-directory/develop/app-objects-and-service-principals)。

## <a name="get-service-principals"></a>获取服务主体

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/ServicePrincipals`      | `GET`           |

在 Azure Databricks 工作区中检索所有服务主体的列表。

### <a name="example-request"></a>示例请求

```http
GET /api/2.0/preview/scim/v2/ServicePrincipals  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

可以使用[筛选器](index.md#filter-results)指定服务主体的子集。 例如，可以将 `eq`（“等于”）筛选器参数应用到 `applicationId` 以检索特定服务主体：

```http
GET /api/2.0/preview/scim/v2/ServicePrincipals?filter=applicationId+eq+b4647a57-063a-43e3-a6b4-c9a4e9f9f0b7  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

## <a name="get-service-principal-by-id"></a>按 ID 获取服务主体

| 端点                                      | HTTP 方法     |
|-----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/ServicePrincipals/{id}`  | `GET`           |

在给定 Azure Databricks ID 的情况下，从 Azure Databricks 工作区中检索单个服务主体资源。

### <a name="example-request"></a>示例请求

```http
GET /api/2.0/preview/scim/v2/ServicePrincipals/7535194597985784  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

## <a name="add-service-principal"></a>添加服务主体

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/ServicePrincipals`      | `POST`          |

在 Azure Databricks 工作区中添加服务主体。

请求参数遵循标准 SCIM 2.0 协议。

请求应包含以下属性：

* `schemas`：设置为 `urn:ietf:params:scim:schemas:core:2.0:ServicePrincipal`
* `applicationId`：（服务主体的 Azure AD 应用程序 ID）
* `displayName`：（可选）

### <a name="example-request"></a>示例请求

```http
POST /api/2.0/preview/scim/v2/ServicePrincipals HTTP/1.1
Host: <databricks-instance>
Authorization: Bearer dapi48…a6138b
Content-Type: application/scim+json
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:schemas:core:2.0:ServicePrincipal"
  ],
  "applicationId":"b4647a57-063a-43e3-a6b4-c9a4e9f9f0b7",
  "displayName":"test-service-principal",
  "groups":[
    {
       "value":"123456"
    }
  ],
  "entitlements":[
    {
       "value":"allow-cluster-create"
    }
  ]
}
```

## <a name="update-service-principal-by-id-patch"></a>按 ID 更新服务主体 (PATCH)

| 端点                                      | HTTP 方法     |
|-----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/ServicePrincipals/{id}`  | `PATCH`         |

使用对特定属性（不可变属性除外）的操作来更新服务主体资源。 建议使用 `PATCH` 方法（而不是 `PUT` 方法）来设置或更新用户权利。

请求参数遵循标准 SCIM 2.0 协议，并依赖于 `schemas` 属性的值。

### <a name="add-entitlements"></a>添加权利

#### <a name="example-request"></a>示例请求

```http
PATCH /api/2.0/preview/scim/v2/ServicePrincipals/654321  HTTP/1.1
Host: <databricks-instance>
Content-Type: application/scim+json
Authorization: Bearer dapi48…a6138b
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations":[
    {
      "op":"add",
      "path":"entitlements",
      "value":[
        {
           "value":"allow-cluster-create"
        }
      ]
    }
  ]
}
```

### <a name="remove-entitlements"></a>删除权利

#### <a name="example-request"></a>示例请求

```http
PATCH /api/2.0/preview/scim/v2/ServicePrincipals/654321  HTTP/1.1
Host: <databricks-instance>
Content-Type: application/scim+json
Authorization: Bearer dapi48…a6138b
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations":[
    {
      "op":"remove",
      "path":"entitlements",
      "value":[
        {
           "value":"allow-cluster-create"
        }
      ]
    }
  ]
}
```

### <a name="add-to-a-group"></a>添加到组

#### <a name="example-request"></a>示例请求

```http
PATCH /api/2.0/preview/scim/v2/ServicePrincipals/654321  HTTP/1.1
Host: <databricks-instance>
Content-Type: application/scim+json
Authorization: Bearer dapi48…a6138b
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations":[
    {
      "op":"add",
      "path":"groups",
      "value":[
        {
           "value":"123456"
        }
      ]
    }
  ]
}
```

### <a name="remove-from-a-group"></a>从组中删除

#### <a name="example-request"></a>示例请求

```http
PATCH /api/2.0/preview/scim/v2/Groups/<group_id>  HTTP/1.1
Host: <databricks-instance>
Content-Type: application/scim+json
Authorization: Bearer dapi48…a6138b
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations":[
    {
      "op":"remove",
      "path":"members[value eq \"<service_principal_id>\"]"
    }
  ]
}
```

## <a name="update-service-principal-by-id-put"></a>按 ID 更新服务主体 (PUT)

| 端点                                      | HTTP 方法     |
|-----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/ServicePrincipals/{id}`  | `PUT`           |

跨多个属性（不可变属性除外）覆盖服务主体资源。

请求必须包含设置为 `urn:ietf:params:scim:schemas:core:2.0:ServicePrincipal` 的 `schemas` 属性。

> [!NOTE]
>
> 建议使用 `PATCH` 方法（而不是 `PUT` 方法）来设置或更新服务主体属性。

### <a name="example-request"></a>示例请求

```http
PUT /api/2.0/preview/scim/v2/ServicePrincipals/654321 HTTP/1.1
Host: <databricks-instance>
Authorization: Bearer dapi48…a6138b
Content-Type: application/scim+json
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:schemas:core:2.0:ServicePrincipal"
  ],
  "applicationId":"b4647a57-063a-43e3-a6b4-c9a4e9f9f0b7",
  "displayName":"test-service-principal",
  "groups":[
    {
       "value":"123456"
    }
  ],
  "entitlements":[
    {
       "value":"allow-cluster-create"
    }
  ]
}
```

## <a name="delete-service-principal-by-id"></a>按 ID 删除服务主体

| 端点                                      | HTTP 方法     |
|-----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/ServicePrincipals/{id}`  | `DELETE`        |

停用服务主体资源。 不拥有或不属于 Azure Databricks 工作区的服务主体将在 30 天后被自动清除。

```http
DELETE /api/2.0/preview/scim/v2/ServicePrincipals/654321  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```