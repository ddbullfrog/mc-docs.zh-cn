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
ms.openlocfilehash: 170323bdb4ff6b2cbff6fdf6f872af0661a28ee6
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246669"
---
<!--Verified successfully-->
<!--Only characters content from verfied articles-->
此步骤在中心与 VNet 之间创建互连。 针对要连接的每个 VNet 重复这些步骤。

1. 在虚拟 WAN 的页面上，选择“虚拟网络连接”。
1. 在虚拟网络连接页上，选择“+ 添加连接”。
1. 在“添加连接”页上填写以下字段：

    * **连接名称** - 为连接命名。
    * **中心** - 选择要与此连接关联的中心。
    * **订阅** - 验证订阅。
    * **虚拟网络** - 选择要连接到此中心的虚拟网络。 此虚拟网络不能包含现有的虚拟网络网关。
1. 单击“确定”以创建连接。

<!-- Update_Description: new article about virtual wan connect vnet hub include -->
<!--NEW.date: 08/10/2020-->