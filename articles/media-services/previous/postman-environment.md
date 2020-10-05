---
title: 为 Azure 媒体服务 REST 调用导入 Postman 环境
description: 本文提供了用于 Azure 媒体服务 REST 调用的 Postman 环境的定义。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 02/08/2019
ms.date: 09/28/2020
ms.author: v-jay
ms.openlocfilehash: 73eb573a4567c1e0cb86b93e60c241bc98a5d626
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245420"
---
# <a name="import-the-postman-environment"></a>导入 Postman 环境

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

本文包含 [Postman 集合](postman-collection.md)使用的 **Postman** 环境变量的定义，该集合包含调用媒体服务 REST API 的分组 HTTP 请求。 [配置 Postman 以便进行媒体服务 REST API 调用](media-rest-apis-with-postman.md)教程将使用这些环境和集合文件。

> [!NOTE]
> `AzureADSTSEndpoint ` 的值 =  `https://login.partner.microsoftonline.cn/{{TenantId}}/oauth2/token`。 若要获取你的租户 ID，可以将鼠标悬停在门户中的用户名（在右上角）上，该 ID 将位于“目录:Microsoft ( {{TENANTID}} )”中。

```
{
  "id": "2dbce3ce-74c2-2ceb-0461-c4c2323f5b09",
  "name": "AzureMedia",
  "values": [
    {
      "enabled": true,
      "key": "TenantID",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "AzureADSTSEndpoint",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "RESTAPIEndpoint",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "ClientID",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "ClientSecret",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "AccessToken",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "LastAssetId",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "LastAccessPolicyId",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "UploadURL",
      "value": "",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "MediaFileName",
      "value": "BigBuckBunny.mp4",
      "type": "text"
    },
    {
      "enabled": true,
      "key": "LastChannelId",
      "value": "",
      "type": "text"
    }
  ],
  "_postman_variable_scope": "environment",
  "_postman_exported_using": "Postman/5.5.0"
}
```
