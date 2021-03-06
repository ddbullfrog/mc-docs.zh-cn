---
title: include 文件
description: include 文件
services: container-registry
author: rockboyfor
ms.service: container-registry
ms.topic: include
origin.date: 06/18/2020
ms.date: 08/10/2020
ms.testscope: no
ms.testdate: 05/18/2020
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 4d59051c8ed46a31cbf6880883d433879b0f744d
ms.sourcegitcommit: ac70b12de243a9949bf86b81b2576e595e55b2a6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2020
ms.locfileid: "88753616"
---
| 资源 | 基本 | Standard | Premium |
|---|---|---|---|
| 包括的存储<sup>1</sup> (GiB) | 10 | 100 | 500 |
| 存储限制 (TiB) | 20| 20 | 20 |
| 最大映像层大小 (GiB) | 200 | 200 | 200 |
| 每分钟读取操作数<sup>2、3</sup> | 1,000 | 3,000 | 10,000 |
| 每分钟写入操作数<sup>2、4</sup> | 100 | 500 | 2,000 |
| 下载带宽 (MBps)<sup>2</sup> | 30 | 60 | 100 |
| 上传带宽 (MBps)<sup>2</sup> | 10 | 20 | 50 |
| Webhook | 2 | 10 | 500 |
| 异地复制 | 空值 | 空值 | [支持][geo-replication] |
| 内容信任 | 空值 | 空值 | [支持][content-trust] |
| 服务终结点 VNet 访问 | 空值 | 空值 | [预览][vnet] |
| 客户管理的密钥 | 空值 | 空值 | [支持][cmk] |
| 存储库范围内的权限 | 空值 | 空值 | [预览][token]|
| &bull; 令牌 | 空值 | 空值 | 20,000 |
| &bull; 范围映射 | 空值 | 空值 | 20,000 |
| &bull; 每个范围映射的存储库 | 空值 | 空值 | 500 |

<!--Not Available on Line 28+1 | Private link with private endpoints -->

<sup>1</sup> 在每日费率中包括的每个层级的存储。 对于附加存储，将按 GiB（存在存储限制）收取额外的每日费率费用。 有关费率的信息，请参阅 [Azure 容器注册表定价][pricing]。

<sup>2</sup>读取操作数、写入操作数和带宽是最小估计值。 Azure 容器注册表致力于根据使用情况来提高性能。

<sup>3</sup>[docker pull](https://docs.docker.com/registry/spec/api/#pulling-an-image) 将根据映像中的层数和清单检索行为转换为多个读取操作。

<sup>4</sup>[docker push](https://docs.docker.com/registry/spec/api/#pushing-an-image) 将根据必须推送的层数转换为多个写入操作。 `docker push` 包含 ReadOps，用于检索现有映像的清单。

<!-- LINKS - External -->

[pricing]: https://www.azure.cn/pricing/details/container-registry/

<!-- LINKS - Internal -->

[geo-replication]: ../articles/container-registry/container-registry-geo-replication.md
[content-trust]: ../articles/container-registry/container-registry-content-trust.md
[vnet]: ../articles/container-registry/container-registry-vnet.md

<!-- Not Available on [plink]: ../articles/container-registry/container-registry-private-link.md-->

[cmk]: ../articles/container-registry/container-registry-customer-managed-keys.md
[token]: ../articles/container-registry/container-registry-repository-scoped-permissions.md

<!-- Update_Description: update meta properties, wording update, update link -->