---
title: 为 Azure Databricks 路由表和防火墙更新新的 MySQL IP
description: 了解如何为 Azure Databricks 路由表和防火墙更新新的 MySQL IP 地址。
services: azure-databricks
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: azure-databricks
ms.workload: big-data
ms.topic: conceptual
ms.date: 06/02/2020
ms.openlocfilehash: fff1989a510ab8b400af209ca29dfd53fb7fb51b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106684"
---
# <a name="update-your-azure-databricks-route-tables-and-firewalls-with-new-mysql-ips"></a>为 Azure Databricks 路由表和防火墙更新新的 MySQL IP

用作 Azure Databricks 元存储的 Azure Database for MySQL 的所有 IP 地址都将更改。 请在 2020 年 6 月 30 日之前为 Azure Databricks 路由表或防火墙更新新的 MySQL IP，以避免中断。

部署在你自己的虚拟网络中的 Azure Databricks 工作区附加了一个[路由表](/databricks/administration-guide/cloud-configurations/azure/udr#--metastore-artifact-blob-storage-log-blob-storage-and-event-hub-endpoint-ip-addresses)。 路由表可直接包含 Azure Databricks 元存储 IP 地址或路由到可能将该地址列入允许列表的防火墙或代理设备。

## <a name="recommended-actions"></a>建议的操作

若要避免服务中断，请在 2020 年 6 月 30 日之前查看并应用这些操作。

* 检查你的路由表是否受 Azure MySQL IP 地址更新中的更改影响。

* 使用下一部分中的表查找新的 IP 地址。 对于每个区域，可以将多个 IP 地址用作 Azure Database for MySQL 网关的主 IP 和辅助 IP。 主 IP 地址是网关的当前 IP 地址，辅助 IP 地址是主 IP 地址发生故障时使用的故障转移 IP 地址。 若要确保服务运行良好，你应该允许出站到所有 IP 地址。 如果使用[外部元存储](/databricks/data/metastores/external-hive-metastore)，请确保已将符合 Azure MySQL 通知的有效 IP 路由或列入允许列表。

* 将路由表、防火墙或代理设备更新为新的 IP。

## <a name="updated-mysql-ip-addresses"></a>更新后的 MySQL IP 地址

下表包括必须列入允许列表的所有 IP 地址。 粗体的 IP 地址为新的 IP 地址。 

| 区域名称          | 网关 IP 地址                                                                                       |
| -------------------- | ---------------------------------------------------------------------------------------------------------- |
| 澳大利亚中部    | 20.36.105.0                                                                                                |
| 澳大利亚中部 2  | 20.36.113.0                                                                                                |
| 澳大利亚东部       | 13.75.149.87<br><br>40.79.161.1                                                                            |
| 澳大利亚东南部 | 191.239.192.109<br><br>13.73.109.251                                                                       |
| Brazil South         | 104.41.11.5 <br><br> 191.233.201.8 <br><br> **191.233.200.16**                                             |
| 加拿大中部       | 40.85.224.249                                                                                              |
| 加拿大东部          | 40.86.226.166                                                                                              |
| 美国中部           | 23.99.160.139<br><br>13.67.215.62<br><br>**52.182.136.37**<br><br>**52.182.136.38**                        |
| 中国东部           | 139.219.130.35                                                                                             |
| 中国东部 2         | 40.73.82.1                                                                                                 |
| 中国北部          | 139.219.15.17                                                                                              |
| 中国北部 2        | 40.73.50.0                                                                                                 |
| 东亚            | 191.234.2.139<br><br>52.175.33.150<br><br>**13.75.33.20**<br><br>**13.75.33.21**                           |
| 美国东部              | 40.121.158.30<br>191.238.6.43                                                                              |
| 美国东部 2            | 40.79.84.180<br><br>191.239.224.107<br><br>52.177.185.181<br><br>**40.70.144.38**<br><br>**52.167.105.38** |
| 法国中部       | 40.79.137.0<br><br>40.79.129.1                                                                             |
| 德国中部      | 51.4.144.100                                                                                               |
| 德国东北部   | 51.5.144.179                                                                                               |
| 印度中部        | 104.211.96.159                                                                                             |
| 印度南部          | 104.211.224.146                                                                                            |
| 印度西部           | 104.211.160.80                                                                                             |
| Japan East           | 13.78.61.196<br><br>191.237.240.43                                                                         |
| 日本西部           | 104.214.148.156<br><br>191.238.68.11<br><br>**40.74.96.6**<br><br>**40.74.96.7**                           |
| 韩国中部        | 52.231.32.42                                                                                               |
| 韩国南部          | 52.231.200.86                                                                                              |
| 美国中北部     | 23.96.178.199<br><br>23.98.55.75<br><br>**52.162.104.35**<br><br>**52.162.104.36**                         |
| 北欧         | 40.113.93.91<br><br>191.235.193.75<br><br>**52.138.224.6**<br><br>**52.138.224.7**                         |
| 南非北部   | 102.133.152.0                                                                                              |
| 南非西部    | 102.133.24.0                                                                                               |
| 美国中南部     | 13.66.62.124<br><br>23.98.162.75<br><br>**104.214.16.39**<br><br>**20.45.120.0**                           |
| 东南亚      | 104.43.15.0<br><br>23.100.117.95<br><br>**40.78.233.2**<br><br>**23.98.80.12**                             |
| 阿联酋中部          | 20.37.72.64                                                                                                |
| 阿拉伯联合酋长国北部            | 65.52.248.0                                                                                                |
| 英国南部             | 51.140.184.11                                                                                              |
| 英国西部              | 51.141.8.11                                                                                                |
| 美国中西部      | 13.78.145.25                                                                                               |
| 西欧          | 40.68.37.158<br><br>191.237.232.75<br><br>**13.69.105.208**                                                |
| 美国西部              | 104.42.238.205<br><br>23.99.34.75                                                                          |
| 美国西部 2            | 13.66.226.202                                                                                              |

## <a name="next-steps"></a>后续步骤

* [在 Azure 虚拟网络中部署 Azure Databricks（VNet 注入）](/databricks/administration-guide/cloud-configurations/azure/vnet-inject)
* [外部 Apache Hive 元存储](/databricks/data/metastores/external-hive-metastore)
