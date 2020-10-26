---
title: include 文件
description: include 文件
services: virtual-wan
ms.service: virtual-wan
ms.topic: include
origin.date: 10/06/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: 10/26/2020
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 3db85ba96dac10f71c391774c3baf37156d772d4
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472548"
---
<!--Verified Successfully-->
1. 在虚拟 WAN 下选择“中心”，然后选择“+新建中心”。

   :::image type="content" source="media/virtual-wan-p2s-hub/new-hub.jpg" alt-text="新中心":::

1. 在“创建虚拟中心”页上，请填写以下字段。

   * **区域** - 选择要在其中部署虚拟中心的区域。
   * **名称** - 输入要用于称呼虚拟中心的名称。
   * **中心专用地址空间** - 用 CIDR 表示法来表示的中心地址范围。

   :::image type="content" source="media/virtual-wan-p2s-hub/create-hub.jpg" alt-text="新中心":::

1. 在“点到站点”选项卡上填写以下字段：

   * **网关缩放单元** - 表示用户 VPN 网关的聚合容量。
   * **点到站点配置** - 已在上一步中创建。
   * **客户端地址池** - 用于远程用户。
   * **自定义 DNS 服务器 IP** 。

   :::image type="content" source="media/virtual-wan-p2s-hub/hub-with-p2s.png" alt-text="新中心":::

1. 选择“查看 + 创建”。
1. 在“验证已通过”页上，选择“创建” 。

<!-- Update_Description: new article about virtual wan p2s hub include -->
<!--NEW.date: 10/26/2020-->