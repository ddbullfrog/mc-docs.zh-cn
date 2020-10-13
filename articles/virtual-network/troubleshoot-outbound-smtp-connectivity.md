---
title: 在 Azure 中排查出站 SMTP 连接问题 | Azure
description: 了解发送电子邮件的建议方法，以及如何对 Azure 中的出站 SMTP 连接问题进行故障排除。
services: virtual-network
manager: dcscontentpm
editor: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 11/20/2018
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 149cfa9d40ce951e8f1d0e78225e9f3fe9159381
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564164"
---
# <a name="troubleshoot-outbound-smtp-connectivity-issues-in-azure"></a>在 Azure 中排查出站 SMTP 连接问题

从 2017 年 11 月 15 日开始，从虚拟机 (VM) 直接发送到外部域（例如 outlook.com）的出站电子邮件将仅适用于 Azure 中的某些订阅类型。 将阻止使用 TCP 端口 25 的出站 SMTP 连接。 （端口 25 主要用于未经身份验证的电子邮件传递。）

<!--Not Available on gmail.com-->

此行为更改仅适用于 2017 年 11 月 15 日及以后的新订阅和新部署。

## <a name="recommended-method-of-sending-email"></a>推荐的电子邮件发送方法
建议使用经过身份验证的 SMTP 中继服务（通常通过 TCP 端口 587 或 443 连接，但也支持其他端口），从 Azure VM 或 Azure 应用服务发送电子邮件。 这些服务用于维护 IP 或域信誉，以尽量降低第三方电子邮件提供商拒绝邮件的可能性。 此类 SMTP 中继服务包括但不限于 [SendGrid](https://sendgrid.com/partners/azure/)。 也可以使用本地运行的安全的 SMTP 中继服务。

在 Azure 中使用这些电子邮件传递服务不受限制（无论是哪种订阅类型）。

## <a name="enterprise-agreement"></a>企业协议
企业协议 Azure 用户仍然无需使用经身份验证的中继便可发送电子邮件。 新的和现有的企业协议用户均可以尝试从 Azure VM 直接向外部电子邮件提供商发送出站电子邮件，且没有任何 Azure 平台限制。 虽然不保证电子邮件提供商会接受来自任何给定用户的传入电子邮件，但对于企业协议订阅中的 VM，Azure 平台不会阻止发送尝试。 必须直接与电子邮件提供商接洽，以修复涉及特定提供商的所有邮件发送或垃圾邮件筛选问题。

## <a name="standard-pay-in-advance-offer"></a>标准预付费套餐
如果已在 2017 年 11 月 15 日前注册标准预付费套餐或 Azure 合作伙伴网络订阅产品/服务，则依然能够尝试发送出站电子邮件。 将能够继续尝试从这些订阅中的 Azure VM 直接向外部电子邮件提供商发送出站电子邮件，且没有任何 Azure 平台限制。 同样，不保证电子邮件提供商会接受来自任何给定用户的传入电子邮件，用户需要直接与电子邮件提供商协作，以处理涉及特定提供商的所有邮件发送或垃圾邮件筛选问题。

在 2017 年 11 月 15 日之后创建的标准预付费套餐或 Azure 合作伙伴网络订阅中存在一个技术限制，它会阻止这些订阅中的 VM 直接发送电子邮件。 如需将电子邮件从 Azure VM 直接发送到外部电子邮件提供商（不使用经身份验证的 SMTP 中继），可以请求去除该限制。 世纪互联会根据自己的判断审批请求，并且只会在执行额外的防欺诈检查后才授予权限。 若要提交请求，请使用以下支持类型开启一个支持案例：**订阅管理**服务类型：**维护通知相关问题**。 请确保添加详细信息，解释为什么要直接向电子邮件提供商发送邮件而不是使用经身份验证的中继。

<!--MOONCAKE CORRECT ON support type: **Subscription Management** service type: **Maintenance notification related issue**-->

在豁免标准预付费套餐或 Azure 合作伙伴网络订阅并在 Azure 门户中先“停止”后“启动” VM 以后，该订阅中的所有 VM 在此后会被豁免。 豁免仅适用于请求的订阅，并且仅适用于直接路由到 Internet 的虚拟机流量。 不支持通过 Azure PaaS 服务（如 [Azure 防火墙](https://www.azure.cn/home/features/azure-firewall/)）路由端口 25 流量。

> [!NOTE]
> 如果确定发生违反服务条款的情况，Azure 保留撤销此豁免的权利。

## <a name="msdn-azure-pass-azure-in-open-education-bizspark-and-trial"></a>MSDN、Azure Pass、Azure 开放许可、教育、BizSpark 和试用版
2017 年 11 月 15 日后创建的 MSDN、Azure Pass、Azure 开放许可、Azure 教育、BizSpark、Azure 赞助、Azure Student、试用版或任何 Visual Studio 订阅均存在技术限制，会阻止从这些订阅中的 VM 直接向电子邮件提供商发送电子邮件。 此限制是为了防止滥用邮件。 不接受去除此限制的请求。

如果使用这些订阅类型，建议使用 SMTP 中继服务，如本文前面部分所述，或者更改订阅类型。

## <a name="cloud-service-provider-csp"></a>云服务提供商 (CSP)

如果通过 CSP 使用 Azure 资源，并且无法使用安全的 SMTP 中继，则可请求 CSP 代表你通过 Azure 创建“取消阻止”豁免请求。

## <a name="need-help-contact-support"></a>需要帮助？ 联系支持人员

如果仍需要帮助，可[联系支持人员](https://support.azure.cn/support/support-azure/)，通过以下支持类型快速解决问题：**订阅管理**服务类型：**维护通知相关问题**。

<!--MOONCAKE CORRECT ON support type: **Subscription Management** service type: **Maintenance notification related issue**-->

<!-- Update_Description: update meta properties, wording update, update link -->