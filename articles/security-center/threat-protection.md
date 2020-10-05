---
title: Azure 安全中心的威胁防护
description: 本主题介绍受 Azure 安全中心威胁防护功能保护的资源
services: security-center
documentationcenter: na
author: Johnnytechn
manager: rkarlin
ms.assetid: 33c45447-3181-4b75-aa8e-c517e76cd50d
ms.service: security-center
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: v-johya
origin.date: 03/15/2020
ms.openlocfilehash: d0b1d55ccfd703e366029d062ed478f3a527f398
ms.sourcegitcommit: cdb7228e404809c930b7709bcff44b89d63304ec
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2020
ms.locfileid: "91402594"
---
# <a name="threat-protection-in-azure-security-center"></a>Azure 安全中心的威胁防护

当安全中心检测到环境中的任何区域遭到威胁时，会生成警报。 这些警报会描述受影响资源的详细信息、建议的修正步骤，在某些情况下还会提供触发逻辑应用作为响应的选项。

Azure 安全中心威胁防护为环境提供全面的防御：
<!--Customized in MC-->

* **针对 Azure 计算资源的威胁防护**：Windows 计算机、Linux 计算机

* **针对 Azure 数据资源的威胁防护**：SQL 数据库

* **针对 Azure 服务层的威胁防护**：Azure 网络层

无论警报是由安全中心生成，还是由安全中心从其他安全产品接收，你都可以导出该警报。 若要将警报导出到任何第三方 SIEM 或任何其他外部工具，请按照[将警报导出到 SIEM](continuous-export.md) 中的说明操作。 

> [!NOTE]
> 来自不同源的警报可能在不同的时间后出现。 例如，需要分析网络流量的警报的出现时间，可能比虚拟机上运行的可疑进程的相关警报要晚一些。

> [!TIP]
> 若要启用安全中心的威胁防护功能，必须将标准定价层应用到包含适用工作负荷的订阅。
>
> 可以在订阅级别或资源级别为 Azure SQL 数据库服务器启用威胁防护。

<!--Customized in MC-->


## <a name="threat-protection-for-windows-machines"></a>针对 Windows 计算机的威胁防护 <a name="windows-machines"></a>

Azure 安全中心与 Azure 服务集成，可以监视和保护基于 Windows 的计算机。 安全中心以易用的格式提供来自所有这些服务的警报和修正建议。

* **Microsoft Defender 高级威胁防护 (ATP)** <a name="windows-atp"></a> - 安全中心通过与 Microsoft Defender 高级威胁防护 (ATP) 相集成扩展了其云工作负荷保护平台。 两者共同提供全面的终结点检测和响应 (EDR) 功能。

    > [!IMPORTANT]
    > 使用安全中心的 Windows 服务器上已自动启用 Microsoft Defender ATP 传感器。

    Microsoft Defender ATP 在检测到威胁时会触发警报。 警报显示在安全中心仪表板上。 在仪表板中，可以透视 Microsoft Defender ATP 控制台，并执行详细调查来发现攻击范围。 有关 Microsoft Defender ATP 的详细信息，请参阅[将服务器加入 Microsoft Defender ATP 服务](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints)。

* **无文件攻击检测** <a name="windows-fileless"></a> - 无文件攻击将恶意有效负载注入内存，以绕过通过基于磁盘的扫描技术进行的检测。 然后，攻击者的有效负载会持久保存在遭到入侵的进程的内存中，执行各种恶意活动。

    自动内存取证技术使用无文件攻击检测来识别无文件攻击工具包、方法和行为。 此解决方案会定期在运行时扫描计算机，并直接从进程的内存中提取见解。 适用于 Linux 的特定见解包括对以下项的识别： 

    - 众所周知的工具包和加密挖掘软件 
    - Shellcode，这是一小段代码，通常用作利用软件漏洞时的有效负载。
    - 在进程内存中注入了恶意可执行文件

    无文件攻击检测会生成详细的安全警报，其中包含具有额外进程元数据的说明，例如网络活动。 这会加快警报会审、关联和下游响应速度。 此方法是对基于事件的 EDR 解决方案的补充，并可以提供更大的检测范围。

    有关无文件攻击检测警报的详细信息，请参阅[警报参考表](alerts-reference.md#alerts-windows)。

> [!TIP]
> 可以下载 [Azure 安全中心 Playbook：安全警报](https://gallery.technet.microsoft.com/Azure-Security-Center-f621a046)来模拟 Windows 警报。






## <a name="threat-protection-for-linux-machines"></a>针对 Linux 计算机的威胁防护 <a name="linux-machines"></a>

安全中心使用 **auditd**（最常见的 Linux 审核框架之一）从 Linux 计算机收集审核记录。 auditd 驻留在主线内核中。 

* **Linux auditd 警报和 Log Analytics 代理集成** <a name="linux-auditd"></a> - auditd 系统包含一个负责监视系统调用的内核级子系统。 该子系统会按照指定的规则集筛选这些调用，并将针对这些调用生成的消息写入到套接字。 安全中心在 Log Analytics 代理中集成了 auditd 包的功能。 通过这种集成，无需满足任何先决条件，就能在所有受支持的 Linux 发行版中收集 auditd 事件。

    可以使用适用于 Linux 的 Log Analytics 代理收集、扩充 auditd 记录并将其聚合到事件中。 安全中心会持续添加新分析功能，这些功能可以使用 Linux 信号来检测云和本地 Linux 计算机上的恶意行为。 类似于 Windows 中的功能，这些分析功能可以检测各种可疑进程、可疑登录企图、内核模块加载操作和其他活动。 这些活动可能表示计算机正在受到攻击或已遭入侵。  

    有关 Linux 警报的列表，请参阅[警报参考表](alerts-reference.md#alerts-linux)。

> [!TIP]
> 可以下载 [Azure 安全中心 Playbook：Linux 检测](https://gallery.technet.microsoft.com/Azure-Security-Center-0ac8a5ef)来模拟 Linux 警报。





<!--Not available in MC: ## Threat protection for Azure App Service-->
<!--Not available in MC: ## Threat protection for containers-->
## <a name="threat-protection-for-sql-database"></a>SQL 数据库的威胁防护 <a name="data-sql"></a>

Azure SQL 数据库的高级威胁防护可检测异常活动，指出有人在访问或利用数据库时的异常行为和可能有害的尝试。

出现可疑的数据库活动、潜在漏洞，或者 SQL 注入攻击以及异常的数据库访问和查询模式时，你会看到警报。

Azure SQL 数据库和 SQL 的高级威胁防护是[高级数据安全 (ADS)](/sql-database/sql-database-advanced-data-security) 统一包的一部分，提供高级 SQL 安全功能，覆盖 Azure SQL 数据库。

有关详细信息，请参阅：

* [如何为 Azure SQL 数据库启用高级威胁防护](/sql-database/sql-database-threat-detection-overview)
* [适用于 SQL 数据库和 Azure Synapse Analytics（以前称为 SQL 数据仓库）的威胁防护警报列表](alerts-reference.md#alerts-sql-db-and-warehouse)



<!--Not available in MC: ## Threat protection for Azure Storage-->
<!--Not avaiable in MC: ## Threat protection for Azure Cosmos DB (Preview)-->
## <a name="threat-protection-for-azure-network-layer"></a>Azure 网络层的威胁防护 <a name="network-layer"></a>

安全中心网络层分析基于示例 IPFIX 数据（即 Azure 核心路由器收集的数据包标头）。 根据此数据馈送，安全中心将使用机器学习模型来识别和标记恶意流量活动。 安全中心还使用 Microsoft 威胁情报数据库来扩充 IP 地址。

某些网络配置可能会限制安全中心对可疑网络活动生成警报。 要使安全中心生成网络警报，请确保：

- 虚拟机有一个公共 IP 地址（或位于使用公共 IP 地址的负载均衡器上）。

- 虚拟机的网络出口流量未被外部 IDS 解决方案阻止。

- 在发生可疑通信的整小时内，为虚拟机分配了同一个 IP 地址。 这也适用于作为托管服务（例如 AKS、Databricks）的一部分创建的 VM。

有关 Azure 网络层警报的列表，请参阅[警报参考表](alerts-reference.md#alerts-azurenetlayer)。




<!--Not available in MC: ## Threat protection for Azure management layer (Azure Resource Manager) (Preview)-->
<!--Not available in MC: ## Threat protection for Azure Key Vault (Preview)-->
## <a name="threat-protection-for-other-microsoft-services"></a>针对其他 Microsoft 服务的威胁防护 <a name="alerts-other"></a>

### <a name="threat-protection-for-azure-waf"></a>针对 Azure WAF 的威胁防护 <a name="azure-waf"></a>

Azure 应用程序网关提供的 Web 应用程序防火墙 (WAF) 可以对 Web 应用程序进行集中保护，避免其受到常见的攻击和漏洞伤害。

Web 应用程序已逐渐成为利用常见已知漏洞的恶意攻击的目标。 应用程序网关 WAF 基于开放 Web 应用程序安全项目中的核心规则集 3.0 或 2.2.9。 WAF 会自动更新，以便在出现新漏洞后提供保护。 

如果你有 Azure WAF 许可证，则无需进行额外的配置，就会将 WAF 警报流式传输到安全中心。 有关 WAF 生成的警报的详细信息，请参阅 [Web 应用程序防火墙 CRS 规则组和规则](../web-application-firewall/ag/application-gateway-crs-rulegroups-rules.md?tabs=owasp31#crs911-31)。


<!--Not available in MC: ### Threat protection for Azure DDoS Protection-->
## <a name="next-steps"></a>后续步骤
若要详细了解这些威胁防护功能生成的安全警报，请参阅以下文章：

* [所有 Azure 安全中心警报的参考表](alerts-reference.md)
* [Azure 安全中心中的安全警报](security-center-alerts-overview.md)
* [在 Azure 安全中心内管理和响应安全警报](security-center-managing-and-responding-alerts.md)
* [导出安全警报和建议（预览版）](continuous-export.md)

