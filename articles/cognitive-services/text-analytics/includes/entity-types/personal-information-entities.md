---
title: 用于个人信息的命名实体
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 10/26/2020
ms.author: v-johya
ms.openlocfilehash: 14f2fd09b178377a333d11856092b21b017cad5f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106578"
---
> [!NOTE]
> 若要检测受保护的运行状况信息 (PHI)，请使用 `domain=phi` 参数和模型版本 `2020-04-01` 或更高版本。
>
> 例如：`https://<your-custom-subdomain>.cognitiveservices.azure.cn/text/analytics/v3.1-preview.2/entities/recognition/pii?domain=phi&model-version=2020-07-01`
 
将请求发送到 `/v3.1-preview.2/entities/recognition/pii` 终结点时，会返回以下实体类别。

| 类别   | Subcategory | 说明                          | 起始模型版本 | 说明 |
|------------|-------------|--------------------------------------|------------------------|---|
| 人员     | 空值         | 人员姓名。  | `2019-10-01`  | 也随 `domain=phi` 一起返回。 |
| PersonType | 空值         | 某人的工作类型或角色。 | `2020-02-01` | |
| PhoneNumber | 空值 | 电话号码（仅限美国和欧洲电话号码）。 | `2019-10-01` | 也随 `domain=phi` 一起返回。 |
|组织  | 空值 | 公司、政治团体、乐队、体育俱乐部、政府机构和公共组织。  | `2019-10-01` | 民族和宗教不包括在此实体类型中。  |
|组织 | 医疗 | 医疗公司和团体。 | `2020-04-01` |  |
|组织 | 证券交易 | 证券交易所集团。 | `2020-04-01` |  |
| 组织 | 体育游戏 | 与体育相关的组织。 | `2020-04-01` |  |
| 地址 | 不可用 | 完整的邮寄地址。  | `2020-04-01` | 也随 `domain=phi` 一起返回。 |
| 欧盟 GPS 坐标 | 空值 | 欧盟内部位置的 GPS 坐标。  | `2019-10-01` |  |
| 电子邮件 | 空值 | 电子邮件地址。 | `2019-10-01` | 也随 `domain=phi` 一起返回。   |
| 代码 | 空值 | 指向网站的 URL。 | `2019-10-01` | 也随 `domain=phi` 一起返回。 |
| IP | 空值 | 网络 IP 地址。 | `2019-10-01` | 也随 `domain=phi` 一起返回。 |
| DateTime | 空值 | 某天的日期和时间。 | `2019-10-01` |  | 
| DateTime | Date | 日历日期。 | `2019-10-01` | 也随 `domain=phi` 一起返回。 |
| 数量 | 空值 | 数字和数量。 | `2019-10-01` |  |
| 数量 | Age | 年龄。 | `2019-10-01` | | |

## <a name="azure-information"></a>Azure 信息

此实体类别包含可识别的 Azure 信息，包括身份验证信息和连接字符串。 从模型版本 `2019-10-01` 开始提供。 不随 `domain=phi` 参数一起返回。

| Subcategory                           | 说明                                                                 |
|---------------------------------------|-----------------------------------------------------------------------------|
| Azure DocumentDB 身份验证密钥             | Azure Cosmos DB 服务器的授权密钥。                           |
| Azure IAAS 数据库连接字符串和 Azure SQL 连接字符串 | Azure 基础结构即服务 (IaaS) 数据库的连接字符串和 SQL 连接字符串。 |
| Azure SQL 连接字符串           | Azure SQL 数据库中的数据库的连接字符串。                                |
| Azure IoT 连接字符串           | Azure IoT 的连接字符串。                        |
| Azure 发布设置密码        | Azure 发布设置的密码。                                        |
| Azure Redis 缓存连接字符串   | Redis 缓存的连接字符串。                             |
| Azure SAS                             | Azure 软件即服务 (SaaS) 的连接字符串。                     |
| Azure 服务总线连接字符串   | Azure 服务总线的连接字符串。                                 |
| Azure 存储帐户密钥             | Azure 存储帐户的帐户密钥。                                   |
| Azure 存储帐户密钥（通用）   | Azure 存储帐户的通用帐户密钥。                           |
| SQL Server 连接字符串          | 运行 SQL Server 的计算机的连接字符串。                                         |

## <a name="identification"></a>标识

[!INCLUDE [supported identification entities](./identification-entities.md)]

