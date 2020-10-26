---
title: 渗透测试 | Azure Docs
description: 本文概述了渗透测试过程，以及对运行在 Azure 基础结构中的应用进行渗透测试的方式。
services: security
documentationcenter: na
author: Johnnytechn
manager: barbkess
editor: TomSh
ms.assetid: 695d918c-a9ac-4eba-8692-af4526734ccc
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
origin.date: 08/13/2018
ms.date: 10/12/2020
ms.author: v-johya
ms.openlocfilehash: fe0814d02c67f71340c7704dce6dc4e22525ec3d
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128045"
---
# <a name="penetration-testing"></a>渗透测试

使用 Azure 进行应用程序测试和部署的一个优点是可快速创建环境。 不必为请求、获取以及“搭架和堆叠”本地硬件担心。

快速创建环境很好，但仍需确保进行常规安全审慎调查。 你可能想要做的事情之一就是对部署在 Azure 中的应用程序进行渗透测试。

用户可能已经知道，Microsoft 将执行[对 Azure 环境的渗透测试](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)。 此测试有助于推动 Azure 改进。

我们不会为你进行应用程序渗透测试，但我们理解你希望对自己的应用程序进行渗透测试，并且需要这样做。 这是好事，因为改进自己的应用程序的安全性可以加强整个 Azure 生态系统的安全性。
<!--Following content was modified to resolve the conflict -->

由于此类测试与实际攻击难以区分，因此客户应仅在事先获得客户支持批准后才进行渗透测试，这一点至关重要。 渗透测试必须按照我们的条款和条件进行。 渗透测试申请应至少提前 7 天提交。
要详细了解或启动渗透测试，请下载[渗透测试审批表](https://www.trustcenter.cn/file/Penetration_Test_Questionnaire_202006-en.docx)，然后联系[客户支持](https://www.21vbluecloud.com/Dynamics365/)。
>[!IMPORTANT]
>渗透测试申请应至少提前 7 天提交，客户也必须遵守 [Microsoft 云统一渗透测试参与规则](https://technet.microsoft.com/mt784683)。

<!--Above content was modified to resolve the conflict -->
可以执行的标准测试包括：

* 对终结点进行测试，以发现[开放 Web 应用程序安全项目 (OWASP) 的前 10 个漏洞](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)
* 终结点的[模糊测试](https://cloudblogs.microsoft.com/microsoftsecure/2007/09/20/fuzz-testing-at-microsoft-and-the-triage-process/)
* 终结点的[端口扫描](https://en.wikipedia.org/wiki/Port_scanner)

你不能执行的一类渗透测试是任何类型的拒绝服务 (DoS) 攻击。 此测试包括：自行发起 DoS 攻击，或者执行相关的测试，以便确定、演示或模拟任何类型的 DoS 攻击。

<!--Not available in MC: ../../virtual-network/manage-ddos-protection.md#validate-ddos-detection-->
## <a name="next-steps"></a>后续步骤

* 详细了解[渗透测试参与规则](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=2)。

