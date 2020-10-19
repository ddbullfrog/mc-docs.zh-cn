---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/02/2020
title: SCIM API（我）- Azure Databricks
description: 了解如何使用 Databricks SCIM API 获取自己的相关信息。
ms.openlocfilehash: bdf134c2d077098ed3f54d67b3e5762cd8683842
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937660"
---
# <a name="scim-api-me"></a>SCIM API（我）

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../../release-notes/release-types.md)提供。

## <a name="get-me"></a><a id="get-me"> </a><a id="scim-get-me"> </a>获取我

| 端点                           | HTTP 方法     |
|------------------------------------|-----------------|
| `2.0/preview/scim/v2/Me`           | `GET`           |

检索自己的相关信息（与[按 ID 获取用户](scim-users.md#get-user-by-id)返回的信息相同）。

### <a name="example-request"></a>示例请求

```http
GET /api/2.0/preview/scim/v2/Me  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```