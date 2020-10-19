---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 令牌 API - Azure Databricks
description: 了解 Databricks 令牌 API。
ms.openlocfilehash: c3d3f7dd7bf5a88d1bb0d9158c67b9f2dd6dbd19
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937821"
---
# <a name="token-api"></a>令牌 API

通过令牌 API，可创建、列出和撤销能用于对 Azure Databricks REST API 进行身份验证和访问的令牌。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="create"></a><a id="create"> </a><a id="tokenstokensservicecreatetoken"> </a>创建

| 端点                  | HTTP 方法     |
|---------------------------|-----------------|
| `2.0/token/create`        | `POST`          |

创建并返回令牌。 如果当前未过期的令牌数量超过令牌配额，则此调用将返回错误 `QUOTA_EXCEEDED`。 用户的令牌配额为 600。

请求示例：

```json
{
  "lifetime_seconds": 100,
  "comment": "this is an example token"
}
```

响应示例：

```json
{
  "token_value":"dapideadbeefdeadbeefdeadbeefdeadbeef",
  "token_info": {
    "token_id":"5715498424f15ee0213be729257b53fc35a47d5953e3bdfd8ed22a0b93b339f4",
    "creation_time":1513120516294,
    "expiry_time":1513120616294,
    "comment":"this is an example token"
  }
}
```

### <a name="request-structure"></a><a id="request-structure"> </a><a id="tokenscreatetoken"> </a>请求结构

| 字段名称               | 类型           | 描述                                                                                               |
|--------------------------|----------------|-----------------------------------------------------------------------------------------------------------|
| lifetime_seconds         | `LONG`         | 令牌的生存期，以秒为单位。 如果未指定生存期，则令牌始终有效。 |
| comment                  | `STRING`       | 要附加到令牌的可选说明。                                                              |

### <a name="response-structure"></a>响应结构

| 字段名称            | 类型                                        | 描述                                         |
|-----------------------|---------------------------------------------|-----------------------------------------------------|
| token_value           | `STRING`                                    | 新创建的令牌的值。               |
| token_info            | [公共令牌信息](#tokenspublictokeninfo) | 新创建的令牌的公共元数据。     |

## <a name="list"></a><a id="list"> </a><a id="tokenstokensservicelisttokens"> </a>列表

| 端点                           | HTTP 方法     |
|------------------------------------|-----------------|
| `2.0/token/list`                   | `GET`           |

列出用户-工作区对的所有有效令牌。

响应示例：

```json
{
  "token_infos": [
    {
      "token_id":"5715498424f15ee0213be729257b53fc35a47d5953e3bdfd8ed22a0b93b339f4",
      "creation_time":1513120516294,
      "expiry_time":1513120616294,
      "comment":"this is an example token"
    },
    {
      "token_id":"902eb9ac42c9bef80d0097a2d1746533103c88593add482a331500187946ceb5",
      "creation_time":1512684023036,
      "expiry_time":-1,
      "comment":"this is another example token"
    }
  ]
}
```

### <a name="response-structure"></a><a id="response-structure"> </a><a id="tokenslisttokensresponse"> </a>响应结构

| 字段名称            | 类型                                                    | 描述                                               |
|-----------------------|---------------------------------------------------------|-----------------------------------------------------------|
| token_infos           | [公共令牌信息](#tokenspublictokeninfo)的数组 | 用户-工作区对的令牌信息列表。    |

## <a name="revoke"></a><a id="revoke"> </a><a id="tokenstokensservicerevoketoken"> </a>撤销

| 端点                             | HTTP 方法     |
|--------------------------------------|-----------------|
| `2.0/token/delete`                   | `POST`          |

撤销访问令牌。 如果具有指定 ID 的令牌无效，则此调用将返回错误 `RESOURCE_DOES_NOT_EXIST`。

请求示例：

```json
{"token_id":"5715498424f15ee0213be729257b53fc35a47d5953e3bdfd8ed22a0b93b339f4"}
```

### <a name="request-structure"></a>请求结构

| 字段名称               | 类型           | 描述                                    |
|--------------------------|----------------|------------------------------------------------|
| token_id                 | `STRING`       | 要撤销的令牌的 ID。             |

## <a name="data-structures"></a><a id="data-structures"> </a><a id="tokenadd"> </a>数据结构

### <a name="in-this-section"></a>本节内容：

* [公共令牌信息](#public-token-info)

### <a name="public-token-info"></a><a id="public-token-info"> </a><a id="tokenspublictokeninfo"> </a>公共令牌信息

描述访问令牌的公共元数据的数据结构。

| 字段名称               | 类型           | 描述                                                                                 |
|--------------------------|----------------|---------------------------------------------------------------------------------------------|
| token_id                 | `STRING`       | 令牌 ID。                                                                        |
| creation_time            | `LONG`         | 创建令牌时的服务器时间（从 epoch 开始的毫秒数）。                             |
| expiry_time              | `LONG`         | 令牌过期时的服务器时间（从 epoch 开始的毫秒数）；如果不适用，则为 -1。    |
| comment                  | `STRING`       | 创建令牌时的注释（如果适用）。                                          |