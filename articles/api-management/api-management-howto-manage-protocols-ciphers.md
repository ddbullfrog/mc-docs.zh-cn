---
title: 在 Azure API 管理中管理协议和加密 | Microsoft Docs
description: 了解如何在 Azure API 管理中管理协议 (TLS) 和密码 (DES)。
services: api-management
documentationcenter: ''
author: Johnnytechn
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
origin.date: 05/29/2019
ms.date: 09/29/2020
ms.author: v-johya
ms.openlocfilehash: 8814c2964c7596253dbb4600f9f9bc8101b3ef21
ms.sourcegitcommit: 80567f1c67f6bdbd8a20adeebf6e2569d7741923
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871371"
---
# <a name="manage-protocols-and-ciphers-in-azure-api-management"></a>在 Azure API 管理中管理协议和加密

Azure API 管理支持客户端和后端的多个 TLS 协议版本，同时也支持 3DES 加密。

此指南将演示如何管理 Azure API 管理实例的协议和加密配置。

![在 APIM 中管理协议和加密](./media/api-management-howto-manage-protocols-ciphers/api-management-protocols-ciphers.png)

## <a name="prerequisites"></a>必备条件

若要执行本文中的步骤，必须提供：

* API 管理实例

## <a name="how-to-manage-tls-protocols-and-3des-cipher"></a>如何管理 TLS 协议和 3DES 加密

1. 在 Azure 门户中，导航到“ Azure 管理实例”。 
2. 从菜单中选择“协议设置”。   
3. 启用或禁用所需协议或加密。
4. 单击“保存”  。 更改将在一个小时内生效。  

## <a name="next-steps"></a>后续步骤

* 详细了解 [TLS（传输层安全性）](https://docs.microsoft.com/dotnet/framework/network-programming/tls)。
