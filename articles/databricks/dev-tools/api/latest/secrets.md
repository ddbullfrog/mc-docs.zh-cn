---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/07/2020
title: 机密 API - Azure Databricks
description: 了解 Databricks 机密 API。
ms.openlocfilehash: 081469405b9e283d42aa250017394a8be4d87d5c
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937822"
---
# <a name="secrets-api"></a>机密 API

可使用机密 API 来管理机密、机密范围和访问权限。 若要管理机密，必须：

1. [创建机密范围](#secretsecretservicecreatescope)。
2. [将机密添加到范围中](#secretsecretserviceputsecret)。
3. 如果你具有 [Azure Databricks 高级计划](https://databricks.com/product/azure-pricing)，请[将访问控制分配到机密范围中](#secretsecretserviceputacl)。

若要详细了解如何创建和管理机密，请参阅[机密管理](../../../security/secrets/index.md#secrets-user-guide)。 使用[机密实用工具](../../databricks-utils.md#dbutils-secrets)在笔记本和作业中访问和引用机密。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。 若要将机密 API 与 Azure Key Vault 机密一起使用，必须使用 Azure Active Directory 令牌进行身份验证。

## <a name="create-secret-scope"></a><a id="create-secret-scope"> </a><a id="secretsecretservicecreatescope"> </a>创建机密范围

| 端点                                  | HTTP 方法     |
|-------------------------------------------|-----------------|
| `2.0/secrets/scopes/create`               | `POST`          |

可以：

* 创建 Azure Key Vault 支持的范围，其中机密存储在 Azure 管理的存储中，并使用基于云的特定加密密钥进行加密。
* 创建 Databricks 支持的机密范围，其中机密存储在 Databricks 管理的存储中，并使用基于云的特定加密密钥进行加密。

### <a name="create-an-azure-key-vault-backed-scope"></a>创建 Azure Key Vault 支持的范围

范围名称：

* 在工作区中必须唯一。
* 必须包含字母数字字符、短划线、下划线和句点，并且不得超过 128 个字符。

这些名称被视为是非敏感信息，工作区中的所有用户都可读取它们。 一个工作区最多只能有 100 个机密范围。

示例请求：

```json
{
  "scope": "my-simple-azure-keyvault-scope",
  "scope_backend_type": "AZURE_KEYVAULT",
  "backend_azure_keyvault":
  {
    "resource_id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azure-rg/providers/Microsoft.KeyVault/vaults/my-azure-kv",
    "dns_name": "https://my-azure-kv.vault.azure.net/"
  },
  "initial_manage_principal": "users"
}
```

如果已指定 `initial_manage_principal`，则应用于该范围的初始 ACL 将应用到所提供的具有 `MANAGE` 权限的主体（用户或组）。 此选项唯一支持的主体是 `users` 组，其中包含工作区中的所有用户。 如果未指定 `initial_manage_principal`，则会将应用于该范围的具有 `MANAGE` 权限的初始 ACL 分配给 API 请求颁发者的用户标识。

如果具有指定名称的范围已存在，会引发 `RESOURCE_ALREADY_EXISTS`。
如果超出工作区中的最大范围数，会引发 `RESOURCE_LIMIT_EXCEEDED`。
如果范围名称无效，则会引发 `INVALID_PARAMETER_VALUE`。

### <a name="create-a-databricks-backed-secret-scope"></a>创建 Databricks 支持的机密范围

范围名称：

* 在工作区中必须唯一。
* 必须包含字母数字字符、短划线、下划线和句点，并且不得超过 128 个字符。

这些名称被视为是非敏感信息，工作区中的所有用户都可读取它们。 一个工作区最多只能有 100 个机密范围。

示例请求：

```json
{
  "scope": "my-simple-databricks-scope",
  "initial_manage_principal": "users"
}
```

如果已指定 `initial_manage_principal`，则应用于该范围的初始 ACL 将应用到所提供的具有 `MANAGE` 权限的主体（用户或组）。
此选项唯一支持的主体是 `users` 组，其中包含工作区中的所有用户。 如果未指定 `initial_manage_principal`，则会将应用于该范围的具有 `MANAGE` 权限的初始 ACL 分配给 API 请求颁发者的用户标识。

如果具有指定名称的范围已存在，会引发 `RESOURCE_ALREADY_EXISTS`。
如果超出工作区中的最大范围数，会引发 `RESOURCE_LIMIT_EXCEEDED`。
如果范围名称无效，则会引发 `INVALID_PARAMETER_VALUE`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretcreatescope"> </a>请求结构

| 字段名称                   | 类型           | 描述                                                                             |
|------------------------------|----------------|-----------------------------------------------------------------------------------------|
| scope                        | `STRING`       | 用户请求的范围名称。 范围名称是唯一的。 此字段为必需字段。       |
| initial_manage_principal     | `STRING`       | 最初就所创建的范围获得 `MANAGE` 权限的主体。       |

## <a name="delete-secret-scope"></a><a id="delete-secret-scope"> </a><a id="secretsecretservicedeletescope"> </a>删除机密范围

| 端点                                  | HTTP 方法     |
|-------------------------------------------|-----------------|
| `2.0/secrets/scopes/delete`               | `POST`          |

删除机密范围。

示例请求：

```json
{
  "scope": "my-secret-scope"
}
```

如果范围不存在，会引发 `RESOURCE_DOES_NOT_EXIST`。
如果用户没有进行此 API 调用的权限，则会引发 `PERMISSION_DENIED`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretdeletescope"> </a>请求结构

| 字段名称     | 类型           | 描述                                          |
|----------------|----------------|------------------------------------------------------|
| scope          | `STRING`       | 要删除的范围的名称。 此字段为必需字段。 |

## <a name="list-secret-scopes"></a><a id="list-secret-scopes"> </a><a id="secretsecretservicelistscopes"> </a>列出机密范围

| 端点                                | HTTP 方法     |
|-----------------------------------------|-----------------|
| `2.0/secrets/scopes/list`               | `GET`           |

列出工作区中所有可用的机密范围。

示例响应:

```json
{
  "scopes": [{
      "name": "my-databricks-scope",
      "backend_type": "DATABRICKS"
  },{
      "name": "mount-points",
      "backend_type": "DATABRICKS"
  }]
}
```

如果你没有进行此 API 调用的权限，会引发 `PERMISSION_DENIED`。

### <a name="response-structure"></a><a id="response-structure"> </a><a id="secretlistscopesresponse"> </a>响应结构

| 字段名称     | 类型                                          | 描述                      |
|----------------|-----------------------------------------------|----------------------------------|
| 范围         | [SecretScope](#secretsecretscope) 的数组 | 可用的机密范围。     |

## <a name="put-secret"></a><a id="put-secret"> </a><a id="secretsecretserviceputsecret"> </a>放置机密

机密的创建/修改方法由[范围后端](#secretscopebackendtype)的类型决定。 若要在 Azure Key Vault 支持的范围中创建或修改机密，请使用 Azure [SetSecret](https://docs.microsoft.com/rest/api/keyvault/setsecret) REST API。 若要创建或修改 Databricks 支持的范围中的机密，请使用以下终结点：

| 端点                        | HTTP 方法     |
|---------------------------------|-----------------|
| `2.0/secrets/put`               | `POST`          |

在提供的具有指定名称的范围下插入机密。 如果已存在同名的机密，此命令将覆盖现有机密的值。
服务器在存储机密之前，会使用机密范围的加密设置对它进行加密。 你必须对机密范围具有 `WRITE` 或 `MANAGE` 权限。

密钥必须包含字母数字字符、短划线、下划线和句点，且不得超过 128 个字符。 允许的最大机密值大小为 128 KB。 指定范围内的最大机密数为
1000.

只能从群集上的命令中读取机密值（例如通过笔记本）；没有用于读取群集外部的机密值的 API。 应用的权限取决于调用命令的用户，而且你必须至少具有 `READ` 权限。

示例请求：

```json
{
  "scope": "my-databricks-scope",
  "key": "my-string-key",
  "string_value": "foobar"
}
```

“string_value”或“bytes_value”输入字段会指定机密的类型，此类型将决定请求机密值时返回的值。 必须恰好指定一个类型。

如果没有这种机密范围，会引发 `RESOURCE_DOES_NOT_EXIST`。
如果超出范围内的最大机密数，会引发 `RESOURCE_LIMIT_EXCEEDED`。
如果密钥名称或值的长度无效，会引发 `INVALID_PARAMETER_VALUE`。
如果用户没有进行此 API 调用的权限，则会引发 `PERMISSION_DENIED`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretputsecret"> </a>请求结构

| 字段名称                              | 类型                        | 描述                                                                                                                                      |
|-----------------------------------------|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| string_value 或 bytes_value             | `STRING` 或 `BYTES`         | 使用指定为 string_value，则值将以 UTF-8 (MB4) 格式存储。<br><br>如果指定为 bytes_value，则值将存储为字节。 |
| scope                                   | `STRING`                    | 将与机密关联的范围的名称。 此字段为必需字段。                                                       |
| key                                     | `STRING`                    | 用于标识机密的唯一名称。 此字段为必需字段。                                                                                    |

## <a name="delete-secret"></a><a id="delete-secret"> </a><a id="secretsecretservicedeletesecret"> </a>删除机密

机密的删除方法由[范围后端](#secretscopebackendtype)的类型决定。 若要从 Azure Key Vault 支持的范围中删除机密，请使用 Azure [SetSecret](https://docs.microsoft.com/rest/api/keyvault/setsecret) REST API。 若要删除 Databricks 支持的范围中的机密，请使用以下终结点：

| 端点                           | HTTP 方法     |
|------------------------------------|-----------------|
| `2.0/secrets/delete`               | `POST`          |

删除存储在此机密范围中的机密。 你必须对机密范围具有 `WRITE` 或 `MANAGE` 权限。

示例请求：

```json
{
  "scope": "my-secret-scope",
  "key": "my-secret-key"
}
```

如果没有这种机密范围或机密，会引发 `RESOURCE_DOES_NOT_EXIST`。
如果你没有进行此 API 调用的权限，会引发 `PERMISSION_DENIED`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretdeletesecret"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                       |
|----------------|----------------|-----------------------------------------------------------------------------------|
| scope          | `STRING`       | 包含要删除的机密的范围的名称。 此字段为必需字段。 |
| key            | `STRING`       | 要删除的机密的名称。 此字段为必需字段。                             |

## <a name="list-secrets"></a><a id="list-secrets"> </a><a id="secretsecretservicelistsecrets"> </a>列出机密

| 端点                         | HTTP 方法     |
|----------------------------------|-----------------|
| `2.0/secrets/list`               | `GET`           |

列出在此范围中存储的密钥。 这是一项仅限元数据的操作；无法使用此 API 检索机密数据。 必须具有 `READ` 权限才能进行此调用。

示例响应:

```json
{
  "secrets": [
    {
      "key": "my-string-key",
      "last_updated_timestamp": 1520467595000
    },
    {
      "key": "my-byte-key",
      "last_updated_timestamp": 1520467595000
    }
  ]
}
```

从 epoch 开始，返回的 last_updated_timestamp 以毫秒为单位。

如果没有这种机密范围，会引发 `RESOURCE_DOES_NOT_EXIST`。
如果你没有进行此 API 调用的权限，会引发 `PERMISSION_DENIED`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretlistsecrets"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                   |
|----------------|----------------|-------------------------------------------------------------------------------|
| scope          | `STRING`       | 要列出其机密的范围的名称。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="response-structure"> </a><a id="secretlistsecretsresponse"> </a>响应结构

| 字段名称     | 类型                                                | 描述                                                               |
|----------------|-----------------------------------------------------|---------------------------------------------------------------------------|
| secrets        | [SecretMetadata](#secretsecretmetadata) 的数组 | 指定范围内包含的所有机密的元数据信息。     |

## <a name="put-secret-acl"></a><a id="put-secret-acl"> </a><a id="secretsecretserviceputacl"> </a>放置机密 ACL

| 端点                             | HTTP 方法     |
|--------------------------------------|-----------------|
| `2.0/secrets/acls/put`               | `POST`          |

在指定范围点上创建或覆盖与给定主体（用户或组）相关联的 ACL。 通常，用户或组将使用其可用的最强大的权限，这些权限的排序如下：

* `MANAGE` - 允许更改 ACL，并在此机密范围进行读取和写入。
* `WRITE` - 允许在此机密范围进行读取和写入。
* `READ` - 允许读取此机密范围并列出哪些机密可用。

必须具有 `MANAGE` 权限才能调用此 API。

示例请求：

```json
{
  "scope": "my-secret-scope",
  "principal": "data-scientists",
  "permission": "READ"
}
```

主体是与要授予或撤销访问权限的现有 Azure Databricks 主体相对应的用户名或组名。

如果没有这种机密范围，会引发 `RESOURCE_DOES_NOT_EXIST`。
如果主体的权限已存在，会引发 `RESOURCE_ALREADY_EXISTS`。
如果权限无效，会引发 `INVALID_PARAMETER_VALUE`。
如果你没有进行此 API 调用的权限，会引发 `PERMISSION_DENIED`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretputacl"> </a>请求结构

| 字段名称     | 类型                                  | 描述                                                               |
|----------------|---------------------------------------|---------------------------------------------------------------------------|
| scope          | `STRING`                              | 要向其应用权限的范围的名称。 此字段为必需字段。    |
| 主体      | `STRING`                              | 要应用权限的主体。 此字段为必需字段。 |
| 权限 (permission)     | [AclPermission](#secretaclpermission) | 应用于主体的权限级别。 此字段为必需字段。    |

## <a name="delete-secret-acl"></a><a id="delete-secret-acl"> </a><a id="secretsecretservicedeleteacl"> </a>删除机密 ACL

| 端点                                | HTTP 方法     |
|-----------------------------------------|-----------------|
| `2.0/secrets/acls/delete`               | `POST`          |

删除给定范围上的给定 ACL。

必须具有 `MANAGE` 权限才能调用此 API。

示例请求：

```json
{
  "scope": "my-secret-scope",
  "principal": "data-scientists"
}
```

如果没有这种机密范围、主体或 ACL，会引发 `RESOURCE_DOES_NOT_EXIST`。
如果你没有进行此 API 调用的权限，会引发 `PERMISSION_DENIED`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretdeleteacl"> </a>请求结构

| 字段名称     | 类型           | 描述                                                               |
|----------------|----------------|---------------------------------------------------------------------------|
| scope          | `STRING`       | 要从中删除权限的范围的名称。 此字段为必需字段。 |
| 主体      | `STRING`       | 要从中删除现有 ACL 的主体。 此字段为必需字段。     |

## <a name="get-secret-acl"></a><a id="get-secret-acl"> </a><a id="secretsecretservicegetacl"> </a>获取机密 ACL

| 端点                             | HTTP 方法     |
|--------------------------------------|-----------------|
| `2.0/secrets/acls/get`               | `GET`           |

介绍有关给定 ACL 的详细信息，例如组和权限。

必须具有 `MANAGE` 权限才能调用此 API。

示例响应:

```json
{
  "principal": "data-scientists",
  "permission": "READ"
}
```

如果没有这种机密范围，会引发 `RESOURCE_DOES_NOT_EXIST`。
如果你没有进行此 API 调用的权限，会引发 `PERMISSION_DENIED`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretgetacl"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                  |
|----------------|----------------|------------------------------------------------------------------------------|
| scope          | `STRING`       | 要从中提取 ACL 信息的范围的名称。 此字段为必需字段。 |
| 主体      | `STRING`       | 要为其提取 ACL 信息的主体。 此字段为必需字段。          |

### <a name="response-structure"></a><a id="response-structure"> </a><a id="secretgetaclresponse"> </a>响应结构

| 字段名称     | 类型                                  | 描述                                                               |
|----------------|---------------------------------------|---------------------------------------------------------------------------|
| 主体      | `STRING`                              | 要应用权限的主体。 此字段为必需字段。 |
| 权限 (permission)     | [AclPermission](#secretaclpermission) | 应用于主体的权限级别。 此字段为必需字段。    |

## <a name="list-secret-acls"></a><a id="list-secret-acls"> </a><a id="secretsecretservicelistacls"> </a>列出机密 ACL

| 端点                              | HTTP 方法     |
|---------------------------------------|-----------------|
| `2.0/secrets/acls/list`               | `GET`           |

列出在给定范围上设置的 ACL。

必须具有 `MANAGE` 权限才能调用此 API。

示例响应:

```json
{
  "items": [
    {
        "principal": "admins",
        "permission": "MANAGE"
    },{
        "principal": "data-scientists",
        "permission": "READ"
    }
  ]
}
```

如果没有这种机密范围，会引发 `RESOURCE_DOES_NOT_EXIST`。
如果你没有进行此 API 调用的权限，会引发 `PERMISSION_DENIED`。

### <a name="request-structure"></a><a id="request-structure"> </a><a id="secretlistacls"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                  |
|----------------|----------------|------------------------------------------------------------------------------|
| scope          | `STRING`       | 要从中提取 ACL 信息的范围的名称。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="response-structure"> </a><a id="secretlistaclsresponse"> </a>响应结构

| 字段名称     | 类型                                  | 描述                                                            |
|----------------|---------------------------------------|------------------------------------------------------------------------|
| items          | [AclItem](#secretaclitem) 的数组 | 应用于给定范围内的主体的关联 ACL 规则。     |

## <a name="data-structures"></a>数据结构

### <a name="in-this-section"></a>本节内容：

* [AclItem](#aclitem)
* [SecretMetadata](#secretmetadata)
* [SecretScope](#secretscope)
* [AclPermission](#aclpermission)
* [ScopeBackendType](#scopebackendtype)

### <a name="aclitem"></a><a id="aclitem"> </a><a id="secretaclitem"> </a>AclItem

一个表示在关联范围点上应用于给定主体（用户或组）的 ACL 规则的项。

| 字段名称     | 类型                                  | 描述                                                               |
|----------------|---------------------------------------|---------------------------------------------------------------------------|
| 主体      | `STRING`                              | 要应用权限的主体。 此字段为必需字段。 |
| 权限 (permission)     | [AclPermission](#secretaclpermission) | 应用于主体的权限级别。 此字段为必需字段。    |

### <a name="secretmetadata"></a><a id="secretmetadata"> </a><a id="secretsecretmetadata"> </a>SecretMetadata

有关机密的元数据。 列出机密时返回。 不包含实际的机密值。

| 字段名称                 | 类型           | 描述                                                      |
|----------------------------|----------------|------------------------------------------------------------------|
| key                        | `STRING`       | 用于标识机密的唯一名称。                            |
| last_updated_timestamp     | `INT64`        | 机密的上次更新时间戳（以毫秒为单位）。     |

### <a name="secretscope"></a><a id="secretscope"> </a><a id="secretsecretscope"> </a>SecretScope

用于存储机密的组织资源。 机密范围可以是不同的类型，而且可将 ACL 应用于范围内所有机密的控制权限。

| 字段名称       | 类型                                        | 说明                                     |
|------------------|---------------------------------------------|-------------------------------------------------|
| name             | `STRING`                                    | 用于标识机密范围的唯一名称。     |
| backend_type     | [ScopeBackendType](#secretscopebackendtype) | 机密范围后端的类型。               |

### <a name="aclpermission"></a><a id="aclpermission"> </a><a id="secretaclpermission"> </a>AclPermission

应用于机密范围的机密 ACL 的 ACL 权限级别。

| 权限       | 说明                                                                    |
|------------------|--------------------------------------------------------------------------------|
| READ             | 允许对此范围中的机密执行读取操作（获取和列出）。       |
| WRITE            | 允许在此机密范围对机密进行读取和写入。                        |
| 管理           | 允许读取/写入 ACL 以及在此机密范围对机密进行读取/写入。       |

### <a name="scopebackendtype"></a><a id="scopebackendtype"> </a><a id="secretscopebackendtype"> </a>ScopeBackendType

机密范围后端的类型。

| 类型               | 描述                                                                                                                        |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------|
| AZURE_KEYVAULT     | 将机密存储在 Azure Key Vault 中的机密范围。                                                                  |
| DATABRICKS         | 将机密存储在 Databricks 管理的存储中，并使用基于云的特定加密密钥进行加密的机密范围。 |