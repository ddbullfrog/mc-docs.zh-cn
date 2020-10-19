---
title: 连接体系结构 - Azure Database for MySQL
description: 描述 Azure Database for MySQL 服务器的连接体系结构。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.topic: conceptual
origin.date: 03/16/2020
ms.date: 10/19/2020
ms.openlocfilehash: e29bae6c014854fb6b0816751a2b0d349de17dc8
ms.sourcegitcommit: ba01e2d1882c85ebeffef344ef57afaa604b53a0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041817"
---
# <a name="connectivity-architecture-in-azure-database-for-mysql"></a>Azure Database for MySQL 中的连接体系结构
本文介绍 Azure Database for MySQL 的连接体系结构，以及如何在 Azure 内部和外部将流量从客户端定向到 Azure Database for MySQL 实例。

## <a name="connectivity-architecture"></a>连接体系结构
可以通过网关连接到 Azure Database for MySQL，该网关负责将传入连接路由到服务器在群集中的物理位置。 下图演示了流量流。

:::image type="content" source="./media/concepts-connectivity-architecture/connectivity-architecture-overview-proxy.png" alt-text="连接体系结构概述":::

客户端在连接到数据库时，会获得一个用于连接到网关的连接字符串。 该网关有一个公共 IP 地址，用于侦听端口 3306。 在数据库群集中，流量转发到相应的 Azure Database for MySQL。 因此，为了通过某种方式（例如，通过公司网络）连接到服务器，必须打开客户端防火墙，使出站流量能够访问我们的网关。 下面是一个按区域分类的可供我们的网关使用的 IP 地址的完整列表。

## <a name="azure-database-for-mysql-gateway-ip-addresses"></a>Azure Database for MySQL 网关 IP 地址
下表列出了所有数据区域的 Azure Database for MySQL 网关的主要 IP 和次要 IP。 主 IP 地址是网关的当前 IP 地址，第二个 IP 地址是主 IP 地址故障时使用的故障转移 IP 地址。 如前所述，客户应该允许到这两个 IP 地址的出站流量。 第二个 IP 地址不侦听任何服务，除非 Azure Database for MySQL 激活该地址，使之接受连接。

| **区域名称** | **网关 IP 地址** |
|:----------------|:-------------|
| 中国东部 | 139.219.130.35    |
| 中国东部 2 | 40.73.82.1  |
| 中国北部 | 139.219.15.17    |
| 中国北部 2 | 40.73.50.0     |
||||

## <a name="connection-redirection"></a>连接重定向

Azure Database for MySQL 支持一个额外的连接策略（即“重定向”  ），该策略有助于降低客户端应用程序与 MySQL 服务器之间的网络延迟。 利用此功能，在建立与 Azure Database for MySQL 服务器的初始 TCP 会话后，服务器会将承载 MySQL 服务器的节点的后端地址返回到客户端。 此后，所有后续数据包会绕过网关直接流向服务器。 由于数据包会直接流向服务器，因此延迟和吞吐量这两个指标的表现得到了改善。

引擎版本为 5.6、5.7 和 8.0 的 Azure Database for MySQL 服务器支持此功能。

对重定向的支持可通过 Microsoft 开发的 PHP [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure) 扩展获得，也可在 [PECL](https://pecl.php.net/package/mysqlnd_azure) 上获得。 有关如何在应用程序中使用重定向的详细信息，请参阅[配置重定向](./howto-redirection.md)一文。

> [!IMPORTANT]
> PHP [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure) 扩展对重定向的支持目前为预览版。

## <a name="next-steps"></a>后续步骤

* [使用 Azure 门户创建和管理 Azure Database for MySQL 防火墙规则](./howto-manage-firewall-using-portal.md)
* [使用 Azure CLI 创建和管理 Azure Database for MySQL 防火墙规则](./howto-manage-firewall-using-cli.md)
* [使用 Azure Database for MySQL 配置重定向](./howto-redirection.md)