---
title: 智能检测通知更改 - Azure Application Insights
description: 通过智能检测更改为默认通知收件人。 使用 Azure Application Insights 通过智能检测监视应用程序跟踪，以了解跟踪遥测中的异常模式。
ms.topic: conceptual
author: Johnnytechn
ms.author: v-johya
ms.date: 10/29/2020
ms.reviewer: mbullwin
ms.openlocfilehash: 86a6dcd4d3d78444ac7fbc7c21955c0e0afc10d5
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106205"
---
# <a name="smart-detection-e-mail-notification-change"></a>智能检测电子邮件通知更改

根据客户反馈，我们将于 2019 年 4 月 1 日更改从智能检测接收电子邮件通知的默认角色。

## <a name="what-is-changing"></a>有什么变化？

目前，智能检测电子邮件通知默认发送给 _订阅所有者_ 、 _订阅参与者_ 和 _订阅读者_ 角色。 这些角色通常包括未积极参与监视的用户，这将导致其中许多用户不必要地接收通知。 为了改进此体验，我们会进行更改，以便电子邮件通知仅默认发送给[监视读者](../../role-based-access-control/built-in-roles.md#monitoring-reader)和[监视参与者](../../role-based-access-control/built-in-roles.md#monitoring-contributor)角色。

## <a name="scope-of-this-change"></a>此更改的范围

此更改会影响所有智能检测规则，以下规则除外：

* 标记为预览版的智能检测规则。 这些智能检测规则目前不支持电子邮件通知。

* 故障异常规则。 如果此规则从经典警报迁移到统一警报平台，它会开始针对新的默认角色。

## <a name="how-to-prepare-for-this-change"></a>如何为此更改做准备？

若要确保将来自智能检测的电子邮件通知发送给相关用户，必须将这些用户分配到订阅的[监视读者](../../role-based-access-control/built-in-roles.md#monitoring-reader)或[监视参与者](../../role-based-access-control/built-in-roles.md#monitoring-contributor)角色。

若要通过 Azure 门户将用户分配到“监视读者”或“监视参与者”角色，请按照[添加角色分配](../../role-based-access-control/role-assignments-portal.md#add-a-role-assignment)一文中所述的步骤执行操作。 确保选择 _监视读者_ 或 _监视参与者_ 作为用户分配到的角色。

> [!NOTE]
> 使用规则设置中的“其他电子邮件收件人”选项配置的智能检测通知特定收件人将不会受此更改影响  。 这些收件人将继续接收电子邮件通知。

如果对此更改有任何问题或疑问，欢迎[联系我们](https://support.azure.cn/support/contact/)。

## <a name="next-steps"></a>后续步骤

详细了解智能检测：

- [失败异常](./proactive-failure-diagnostics.md)
- [内存泄漏](./proactive-potential-memory-leak.md)
- [性能异常](./proactive-performance-diagnostics.md)


