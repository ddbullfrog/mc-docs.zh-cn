---
title: Azure 密钥保管库的安全控制
description: 用于评估 Azure 密钥保管库的安全控制的清单
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.topic: conceptual
origin.date: 04/16/2019
ms.date: 10/30/2020
ms.author: v-tawe
ms.openlocfilehash: 757c1a3bc89f4309233120269b1a583eb832b705
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106140"
---
# <a name="security-controls-for-azure-key-vault"></a>Azure 密钥保管库的安全控制

本文介绍 Azure 密钥保管库中内置的安全控制。 

[!INCLUDE [Security controls Header](../../../includes/security-controls-header.md)]

## <a name="network"></a>网络

| 安全控制 | Yes/No | 注释 |
|---|---|--|
| 服务终结点支持| 是 | 使用虚拟网络 (VNet) 服务终结点。 |
| VNet 注入支持| 否 |  |
| 网络隔离和防火墙支持| 是 | 使用 VNet 防火墙规则。 |
| 强制隧道支持| 否 |  |

## <a name="monitoring--logging"></a>监视和日志记录

| 安全控制 | Yes/No | 说明|
|---|---|--|
| Azure 监视支持（Log Analytics、App Insights 等）| 是 | 使用 Log Analytics。 |
| 控制/管理平面日志记录和审核| 是 | 使用 Log Analytics。 |
| 数据平面日志记录和审核| 是 | 使用 Log Analytics。 |

## <a name="identity"></a>标识

| 安全控制 | Yes/No | 说明|
|---|---|--|
| 身份验证| 是 | 身份验证通过 Azure Active Directory 来进行。 |
| 授权| 是 | 使用密钥保管库访问策略。 |

## <a name="data-protection"></a>数据保护

<!--| Server-side encryption at rest: customer-managed keys (BYOK) | Yes | The customer controls all keys in their Key Vault. When hardware security module (HSM) backed keys are specified, a FIPS Level 2 HSM protects the key, certificate, or secret. | -->

| 安全控制 | Yes/No | 说明 |
|---|---|--|
| 服务器端静态加密：Microsoft 管理的密钥 | 是 | 加密所有对象。 |
| 列级加密（Azure 数据服务）| 空值 |  |
| 传输中加密（例如 ExpressRoute 加密、VNet 中加密，以及 VNet-VNet 加密）| 是 | 所有通信都通过加密的 API 调用进行 |
| 加密的 API 调用| 是 | 使用 HTTPS。 |

## <a name="access-controls"></a>访问控制

| 安全控制 | Yes/No | 注释|
|---|---|--|
| 控制/管理平面访问控制 | 是 | Azure Resource Manager 基于角色的访问控制 (RBAC) |
| 数据平面访问控制（在每个服务级别） | 是 | 密钥保管库访问策略 |

<!-- ## Next steps -->

<!-- - Learn more about the [built-in security controls across Azure services](../../security/fundamentals/security-controls.md). -->