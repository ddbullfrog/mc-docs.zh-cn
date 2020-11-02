---
title: 由于静态 IP 而无法通过远程桌面连接到 Azure 虚拟机 | Azure
description: 了解如何对由 Azure 中的静态 IP 导致的 RDP 问题进行故障排除 | Azure
services: virtual-machines-windows
manager: dcscontentpm
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
origin.date: 11/08/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 647af6cb7f29fb72b782101617f2a9f42045acd5
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105243"
---
# <a name="cannot-remote-desktop-to-azure-virtual-machines-because-of-static-ip"></a>由于静态 IP 而无法通过远程桌面连接到 Azure 虚拟机

本文介绍了在 VM 中配置静态 IP 后无法通过远程桌面连接到 Azure Windows 虚拟机 (VM) 的问题。

## <a name="symptoms"></a>症状

与 Azure 中的 VM 建立 RDP 连接时，收到以下错误消息：

**由于以下原因之一，远程桌面无法连接到远程计算机：**

1. **未启用服务器的远程访问**

2. **远程计算机已关闭**

3. **远程计算机在网络中不可用**

**请确保远程计算机已打开并已连接到网络，并且已启用远程访问。**

在 Azure 门户中的[“启动诊断”](../troubleshooting/boot-diagnostics.md)中检查屏幕截图时，你看到 VM 正常启动并且在登录屏幕中等待凭据。

## <a name="cause"></a>原因

VM 具有一个在 Windows 中的网络接口上定义的静态 IP 地址。 此 IP 地址不同于在 Azure 门户中定义的地址。

## <a name="solution"></a>解决方案

在执行这些步骤之前，请创建受影响 VM 的 OS 磁盘的快照作为备份。 有关详细信息，请参阅[拍摄磁盘快照](../windows/snapshot-copy-managed-disk.md)。

若要解决此问题，请为 VM [重置网络接口](reset-network-interface.md)。

<!-- Not Available on use Serial control to enable DHCP or-->
<!-- Not Available on ### Use Serial control-->

之后，如果希望为 VM 配置静态 IP，请参阅[为 VM 配置静态 IP 地址](../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md)。

<!-- Update_Description: update meta properties, wording update -->
