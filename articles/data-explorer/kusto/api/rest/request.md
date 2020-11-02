---
title: 查询/管理 HTTP 请求 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的查询/管理 HTTP 请求。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 01/27/2020
ms.date: 10/29/2020
ms.openlocfilehash: dc4cb14f7db664530138fceae3422626049ec069
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103896"
---
# <a name="querymanagement-http-request"></a>查询/管理 HTTP 请求

## <a name="request-verb-and-resource"></a>请求谓词和资源

|操作    |HTTP 谓词|HTTP 资源   |
|----------|---------|----------------|
|查询     |GET      |`/v1/rest/query`|
|查询     |POST     |`/v1/rest/query`|
|查询 v2  |GET      |`/v2/rest/query`|
|查询 v2  |POST     |`/v2/rest/query`|
|管理|POST     |`/v1/rest/mgmt` |

例如，若要将控制命令（“management”）发送到服务终结点，请使用以下请求行：

```
POST https://help.kusto.chinacloudapi.cn/v1/rest/mgmt HTTP/1.1
```

有关要包含的请求标头和正文，请参见下文。

## <a name="request-headers"></a>请求标头

下表包含用于查询和管理操作的常用标头。

|标准标头  |说明                                                                                 |必需/可选 |
|-----------------|--------------------------------------------------------------------------------------------|------------------|
|`Accept`         |设置为 `application/json`                                                                   |必须          |
|`Accept-Encoding`|支持的编码为 `gzip` 和 `deflate`                                                |可选          |
|`Authorization`  |请参阅[身份验证](./authentication.md)                                                   |必须          |
|`Connection`     |建议启用 `Keep-Alive`                                                   |可选          |
|`Content-Length` |建议在已知的情况下指定请求正文长度                            |可选          |
|`Content-Type`   |使用 `charset=utf-8` 将此项设置为 `application/json`                                              |必须          |
|`Expect`         |可设置为 `100-Continue`                                                                |可选          |
|`Host`           |设置为请求所发送到的限定域名（例如，`help.kusto.chinacloudapi.cn`） |必须|

下表包含用于查询和管理操作的常用自定义标头。 除非另有说明，否则这些标头仅用于遥测目的，而不会影响功能。

所有标头都是可选项。 建议指定 `x-ms-client-request-id` 自定义标头。 在某些情况下（例如取消正在运行的查询），此标头是必需的，因为它用于标识请求。

|自定义标头           |说明                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-app`              |发出请求的应用程序的（易记）名称                                                 |
|`x-ms-user`             |发出请求的用户的（易记）名称                                                        |
|`x-ms-user-id`          |与 `x-ms-user` 相同                                                                                       |
|`x-ms-client-request-id`|请求的唯一标识符                                                                       |
|`x-ms-client-version`   |发出请求的客户端的（易记）版本标识符                                       |
|`x-ms-readonly`         |如果已指定此项，则强制请求以只读模式运行，以防止其进行持久更改 |

## <a name="request-parameters"></a>请求参数

以下参数可以在请求中传递。 它们在请求中被编码为查询参数，或编码为正文的一部分，具体取决于是使用 GET 还是 POST。

|参数   |说明                                                                                 |必需/可选 |
|------------|--------------------------------------------------------------------------------------------|------------------|
|`csl`       |要执行的查询或控制命令的文本                                             |必须          |
|`db`        |作用域中作为查询或控制命令目标的数据库的名称            |对于某些控制命令是可选的。 <br>对于其他命令和所有查询是必需的。 </br>                                                                   |
|`properties`|提供客户端请求属性，这些属性修改请求的处理方式及其结果。 有关详细信息，请参阅[客户端请求属性](../netfx/request-properties.md)                                               | 可选         |

## <a name="get-query-parameters"></a>GET 查询参数

使用 GET 时，请求的查询参数会指定请求参数。

## <a name="body"></a>正文

使用 POST 时，请求正文是以 UTF-8 编码的单个 JSON 文档，其中包含请求参数的值。

## <a name="examples"></a>示例

此示例显示查询的 HTTP POST 请求。

```txt
POST https://help.kusto.chinacloudapi.cn/v2/rest/query HTTP/1.1
```

请求标头

```txt
Accept: application/json
Authorization: Bearer ...AzureActiveDirectoryAccessToken...
Accept-Encoding: deflate
Content-Type: application/json; charset=utf-8
Host: help.kusto.chinacloudapi.cn
x-ms-client-request-id: MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1
x-ms-user-id: EARTH\davidbg
x-ms-app: MyApp
```

请求正文

```json
{
  "db":"Samples",
  "csl":"print Test=\"Hello, World!\"",
  "properties":"{\"Options\":{\"queryconsistency\":\"strongconsistency\"},\"Parameters\":{},\"ClientRequestId\":\"MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1\"}"
}
```

此示例演示如何使用 [curl](https://curl.haxx.se/) 创建用于发送上述查询的请求。

1. 获取用于身份验证的令牌。

    在设置了 [AAD 应用程序身份验证](../../management/access-control/how-to-provision-aad-app.md)之后，将 `AAD_TENANT_NAME_OR_ID`、`AAD_APPLICATION_ID` 和 `AAD_APPLICATION_KEY` 替换为相关值

    ```
    curl "https://login.partner.microsoftonline.cn/AAD_TENANT_NAME_OR_ID/oauth2/token" \
      -F "grant_type=client_credentials" \
      -F "resource=https://help.kusto.chinacloudapi.cn" \
      -F "client_id=AAD_APPLICATION_ID" \
      -F "client_secret=AAD_APPLICATION_KEY"
    ```

    此代码片段将为你提供持有者令牌。

    ```
    {
      "token_type": "Bearer",
      "expires_in": "3599",
      "ext_expires_in":"3599", 
      "expires_on":"1578439805",
      "not_before":"1578435905",
      "resource":"https://help.kusto.chinacloudapi.cn",
      "access_token":"eyJ0...uXOQ"
    }
    ```

1. 在向查询终结点发送的请求中使用持有者令牌。

    ```
    curl -d '{"db":"Samples","csl":"print Test=\"Hello, World!\"","properties":"{\"Options\":{\"queryconsistency\":\"strongconsistency\"}}"}"' \
    -H "Accept: application/json" \
    -H "Authorization: Bearer eyJ0...uXOQ" \
    -H "Content-Type: application/json; charset=utf-8" \
    -H "Host: help.kusto.chinacloudapi.cn" \
    -H "x-ms-client-request-id: MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1" \
    -H "x-ms-user-id: EARTH\davidbg" \
    -H "x-ms-app: MyApp" \
    -X POST https://help.kusto.chinacloudapi.cn/v2/rest/query
    ```

1. 根据[此规范](response.md)，读取响应。
