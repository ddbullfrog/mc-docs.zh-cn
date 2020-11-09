---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 管理工作区安全性标头 - Azure Databricks
description: 了解如何启用和禁用 Azure Databricks 工作区的安全性标头。
ms.openlocfilehash: f8f2ab260a1d9c86cf16401794842905238e709e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106768"
---
# <a name="manage-workspace-security-headers"></a>管理工作区安全性标头

作为管理员用户，你可以管理发送的安全性标头，以防止对工作区的攻击，如下所示：

1. 转到[管理控制台](../admin-console.md)。
2. 单击“高级”  选项卡。

## <a name="manage-third-party-iframing-prevention"></a>管理第三方 iFraming 防护

若要防止第三方域访问 iFraming Azure Databricks，可以通过单击“第三方 iFraming 防护”右侧的“启用”和“禁用”按钮来启用和禁用发送 `X-Frame-Options: sameorigin` [响应头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)  。 默认情况下启用此标志。

## <a name="manage-mime-type-sniffing-prevention"></a>管理 MIME 类型探查防护

若要指示浏览器不执行 MIME 类型的探查，可以通过单击“MIME 类型探查防护”右侧的“启用”和“禁用”按钮来启用和禁用发送 `X-Content-Type-Options: nosniff` [响应头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)  。 默认情况下启用此标志。

## <a name="manage-xss-attack-page-rendering-prevention"></a>管理 XSS 攻击页面呈现防护

若要指示浏览器在检测到攻击时阻止页面呈现，可以通过单击“XSS 攻击页面呈现防护”右侧的“启用”和“禁用”按钮来启用和禁用发送 `X-XSS-Protection: 1; mode=block` [响应头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)  。 默认情况下启用此标志。