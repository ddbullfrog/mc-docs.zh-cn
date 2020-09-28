---
title: Azure 数据资源管理器 REST API - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 Azure 数据资源管理器 REST API。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 05/29/2019
ms.date: 09/24/2020
ms.openlocfilehash: 6f32a88ad932a3f464766164e4a8a84b209236b8
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146349"
---
# <a name="azure-data-explorer-rest-api"></a>Azure 数据资源管理器 REST API

本文介绍如何通过 HTTPS 与 Azure 数据资源管理器交互。

## <a name="supported-actions"></a>支持的操作

终结点支持的这一组操作各不相同，具体取决于终结点是引擎终结点还是数据管理终结点。

|操作         |HTTP 谓词   |URI 模板           |引擎|数据管理|身份验证 |
|---------------|------------|-----------------------|------|---------------|---------------|
|查询          |GET 或 POST |/v1/rest/query         |是   |否             |是            |
|查询          |GET 或 POST |/v2/rest/query         |是   |否             |是            |
|管理     |POST        |/v1/rest/mgmt          |是   |是            |是            |
|UI             |GET         |/                      |是   |否             |否             |
|UI             |GET         |/{dbname}              |是   |否             |否             |

其中的“操作”表示一组相关的活动

* “查询”操作向服务发送一个查询，然后取回查询结果。
* “管理”操作向服务发送一个控制命令，然后取回该控制命令的结果。
* UI 操作可用于启动桌面客户端或 Web 客户端。 操作通过 HTTP 重定向响应来执行，以便与服务交互。

## <a name="next-steps"></a>后续步骤

有关查询和管理操作的 HTTP 请求和响应的详细信息，请参阅：
 * [查询/管理 HTTP 请求](./request.md)
 * [查询/管理 HTTP 响应](./response.md)
 * [查询 v2 HTTP 响应](./response2.md)

有关 UI 操作的详细信息，请参阅：
 * [UI 深层链接](./deeplink.md)
