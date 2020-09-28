---
title: include 文件
description: include 文件
services: virtual-wan
author: rockboyfor
ms.service: virtual-wan
ms.topic: include
origin.date: 07/09/2020
ms.date: 09/25/2020
ms.testscope: yes|no
ms.testdate: 08/10/2020null
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 58333822e9b69c8504e4baaf45655813f10a53bc
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246672"
---
<!--Verified successfully-->
<!--Only characters content from verfied articles-->
从浏览器导航到 Azure 门户并使用 Azure 帐户登录。

1. 导航到“虚拟 WAN”页。 在门户中，单击“+创建资源”。 在搜索框中键入“虚拟 WAN”，然后选择 Enter。
1. 从结果中选择“虚拟 WAN”。 在“虚拟 WAN”页上，单击“创建”以打开“创建 WAN”页。
1. 在“创建 WAN”页的“基本信息”选项卡上，填写以下字段 ：

    :::image type="content" source="./media/virtual-wan-create-vwan-include/basics.png" alt-text="基础知识":::

    * **订阅** - 选择要使用的订阅。
    * **资源组** - 新建资源组或使用现有的资源组。
    * **资源组位置** - 从下拉列表中选择资源位置。 WAN 是一个全局资源，不会驻留在某个特定区域。 但是，必须选择一个区域才能更轻松地管理和查找所创建的 WAN 资源。
    * **名称** - 键入要用于称呼 WAN 的名称。
    * **类型** - 免费、基本或标准。 如果创建基本 WAN，则只能创建基本中心。 基本中心仅支持 VPN 站点到站点连接。
1. 填写完字段后，单击“审阅 + 创建”。
1. 验证通过后，选择“创建”以创建虚拟 WAN。

<!-- Update_Description: new article about virtual wan create vwan include -->
<!--NEW.date: 08/10/2020-->