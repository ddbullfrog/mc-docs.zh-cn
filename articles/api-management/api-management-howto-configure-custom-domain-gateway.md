---
title: 为自承载 Azure API 管理网关配置自定义域名 | Microsoft Docs
description: 本主题介绍为自承载 Azure API 管理网关配置自定义域名的步骤。
services: api-management
documentationcenter: ''
author: Johnnytechn
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
origin.date: 10/31/2019
ms.date: 09/29/2020
ms.author: v-johya
ms.openlocfilehash: 374c7f4b6c8b6a8866b2eb4fa066d310a8df98a6
ms.sourcegitcommit: 80567f1c67f6bdbd8a20adeebf6e2569d7741923
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871412"
---
# <a name="configure-a-custom-domain-name"></a>配置自定义域名

预配[自承载 Azure API 管理网关](self-hosted-gateway-overview.md)时，系统不会为其分配主机名，必须按其 IP 地址引用它。 本文介绍如何将现有的自定义 DNS 名称（也称为主机名）映射到自承载网关。

## <a name="prerequisites"></a>先决条件

若要执行本文中所述的步骤，必须具有：

-   一个有效的 Azure 订阅。

    如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

-   API 管理实例。 有关详细信息，请参阅[创建 Azure API 管理实例](get-started-create-service-instance.md)。
- 一个自承载网关。 有关详细信息，请参阅[如何预配自承载网关](api-management-howto-provision-self-hosted-gateway.md)
-   由你或你的组织拥有的自定义域名。 本主题不会提供有关如何购买自定义域名的说明。
-   DNS 服务器上承载的一条 DNS 记录，用于将自定义域名映射到自承载网关的 IP 地址。 本主题不会提供有关如何承载 DNS 记录的说明。
-   必须具有有效的带有公钥和私钥 (.PFX) 的证书。 使用者或使用者可选名称 (SAN) 必须与域名匹配（这使得 API 管理实例可以通过 TLS 安全地公开 URL）。

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="add-custom-domain-certificate-to-your-api-management-service"></a>将自定义域证书添加到 API 管理服务

1. 在“安全性”下选择“证书”。 
2. 选择“+ 添加”。
3. 在“ID”字段中输入证书的资源名称。
4. 选择“证书”字段或其旁边的文件夹图标，以选择包含证书的文件 (.PFX)。
5. 在“密码”字段中输入证书的密码。
6. 选择“创建”将证书添加到 API 管理服务。

## <a name="use-the-azure-portal-to-set-a-custom-domain-name-for-your-self-hosted-gateway"></a>使用 Azure 门户为自承载网关设置自定义域名

1. 选择“部署和基础结构”下的“网关” 。
2. 选择要为其配置域名的自承载网关。
3. 在“设置”下选择“主机名”。 
4. 选择“+ 添加”。
5. 在“名称”字段中输入主机名的资源名称。
6. 在“主机名”字段中输入域名。
7. 从“证书”下拉列表中选择证书。
8. 如果通过此网关公开的任何 API 使用客户端证书身份验证，请选中“协商客户端证书”复选框。
    > [!WARNING]
    > 此设置由配置给网关的所有域名共享。
9. 选择“添加”，将自定义域名分配到所选的自承载网关。

## <a name="next-steps"></a>后续步骤

[升级和缩放你的服务](upgrade-and-scale.md)

