---
title: 为 SMTP 横幅检查配置反向查找区域
titlesuffix: Azure Virtual Network
description: 介绍如何在 Azure 中为 SMTP 横幅检查配置反向查找区域
services: virtual-network
documentationcenter: virtual-network
manager: dcscontentpm
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
origin.date: 10/31/2018
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: yes
ms.testdate: 08/10/2020
ms.author: v-yeche
ms.openlocfilehash: 57f6c494e0703f2b0f04038786a514753137110f
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564468"
---
# <a name="configure-reverse-lookup-zones-for-an-smtp-banner-check"></a>为 SMTP 横幅检查配置反向查找区域

本文介绍如何使用 Azure DNS 中的反向区域，以及如何为 SMTP 横幅检查创建反向 DNS (PTR) 记录。

## <a name="symptom"></a>症状

如果在 Azure 中托管 SMTP 服务器，则自远程邮件服务器收发邮件时，可能收到以下错误消息：

**554: 无 PTR 记录**

## <a name="solution"></a>解决方案

对于 Azure 中的虚拟 IP 地址，将在 Azure 拥有的域区域（而不是自定义域区域）中创建反向记录。

若要在 Azure 拥有区域配置 PTR 记录，请使用 PublicIpAddress 资源的 -ReverseFqdn 属性。 有关详细信息，请参阅[为 Azure 中托管的服务配置反向 DNS](../dns/dns-reverse-dns-for-azure-services.md)。

配置 PTR 记录时，请确保 IP 地址和反向 FQDN 为订阅所有。 如果尝试设置不属于订阅的反向 FQDN，将收到以下错误消息：

```output
Set-AzPublicIpAddress : ReverseFqdn mail.contoso.com that PublicIPAddress ip01 is trying to use does not belong to subscription <Subscription ID>. One of the following conditions need to be met to establish ownership:

1) ReverseFqdn matches fqdn of any public ip resource under the subscription;
2) ReverseFqdn resolves to the fqdn (through CName records chain) of any public ip resource under the subscription;
3) It resolves to the ip address (through CName and A records chain) of a static public ip resource under the subscription.
```

如果将 SMTP 横幅手动更改为与默认反向 FQDN 相匹配，远程邮件服务器仍可能失败，因为它可能期望 SMTP 横幅主机与域的 MX 记录相匹配。

<!-- Update_Description: update meta properties, wording update, update link -->
