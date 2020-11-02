---
title: 生成和导出用于用户 VPN 连接的证书 | Azure 虚拟 WAN
description: 在 Windows 10 或 Windows Server 2016 上使用 PowerShell 创建自签名根证书、导出公钥和生成用于用户 VPN 连接的客户端证书。
services: virtual-wan
ms.service: virtual-wan
ms.topic: how-to
origin.date: 09/22/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 11/02/2020
ms.author: v-yeche
ms.openlocfilehash: 3420e36b52dc1672feab8d9342d332bf2b0ac9d7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103786"
---
<!--Verified successfully-->
# <a name="generate-and-export-certificates-for-user-vpn-connections"></a>生成和导出用于用户 VPN 连接的证书

用户 VPN（点到站点）连接使用证书进行身份验证。 本文介绍如何在 Windows 10 或 Windows Server 2016 上使用 PowerShell 创建自签名根证书并生成客户端证书。

必须在运行 Windows 10 或 Windows Server 2016 的计算机上执行本文中的步骤。 用于生成证书的 PowerShell cmdlet 是操作系统的一部分，在其他版本的 Windows 上不正常工作。 只需 Windows 10 或 Windows Server 2016 计算机即可生成证书。 生成证书后，可上传证书，或在任何支持的客户端操作系统上安装该证书。

[!INCLUDE [Export public key](../../includes/vpn-gateway-generate-export-certificates-include.md)]

## <a name="next-steps"></a>后续步骤

继续阅读[用户 VPN 连接的虚拟 WAN 步骤](virtual-wan-about.md)

<!-- Update_Description: update meta properties, wording update, update link -->