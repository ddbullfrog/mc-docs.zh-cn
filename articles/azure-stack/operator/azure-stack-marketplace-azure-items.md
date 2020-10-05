---
title: 可用于 Azure Stack Hub 的 Azure 市场项
description: 了解哪些 Azure 市场项可以在 Azure Stack Hub 中使用。
author: WenJason
ms.topic: article
ms.service: azure-stack
origin.date: 09/09/2020
ms.date: 10/12/2020
ms.author: v-jay
ms.reviewer: ihcherie
ms.lastreviewed: 01/21/2019
ms.openlocfilehash: 3ea69a71bc4fd0e64d9465b69a3a1d201b9074b9
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91437688"
---
# <a name="azure-marketplace-items-available-for-azure-stack-hub"></a>可用于 Azure Stack Hub 的 Azure 市场项

## <a name="virtual-machine-extensions"></a>虚拟机扩展

只要有所用虚拟机 (VM) 扩展的更新，就应下载它们。 随产品一起提供的扩展不会在普通的修补和更新过程中更新，因此请经常查看更新。 其他扩展只能通过市场管理获取。

| 映像 | 项名称 | 说明 | 发布者 | OS 类型 |
| --- | --- | --- | --- | --- |
|![SQL IaaS 扩展(SqlIaasExtension)](media/azure-stack-marketplace-azure-items/cse.png) | [SQL IaaS 扩展 (SqlIaasExtension)](/virtual-machines/windows/sql/virtual-machines-windows-sql-server-agent-extension)| **下载此扩展以部署任何“Windows 上的 SQL Server”市场项 - 此扩展是必需的。** | Microsoft | Windows |
|![自定义脚本扩展](media/azure-stack-marketplace-azure-items/cse.png) | [自定义脚本扩展](/virtual-machines/windows/extensions-customscript)| **请下载此更新，此更新针对用于 Windows 的自定义脚本扩展的内置版本。** | Microsoft | Windows |
| ![PowerShell DSC 扩展](media/azure-stack-marketplace-azure-items/dsc.png) | [PowerShell DSC 扩展](/virtual-machines/windows/extensions-dsc-overview)| **请将此更新下载到 PowerShell DSC 扩展的内置版本。更新为支持 TLS v1.2。** | Microsoft | Windows |
| ![Microsoft 反恶意软件扩展](media/azure-stack-marketplace-azure-items/cse.png) | [Microsoft 反恶意软件扩展](/security/azure-security-antimalware)| 用于 Azure 的 Microsoft Antimalware 是一个针对应用和租户环境提供的单一代理解决方案，可在后台运行而无需人工干预。 **请将此更新下载到 Antimalware 扩展的内置版本。** | Microsoft | Windows |
| ![Azure 诊断扩展](media/azure-stack-marketplace-azure-items/cse.png) | [Azure 诊断扩展](/virtual-machines/extensions/diagnostics-windows)| Azure 诊断是 Azure 中可对部署的应用启用诊断数据收集的功能。 **请下载此更新，此更新针对用于 Windows 的诊断扩展的内置版本。** | Microsoft | Windows |
| ![“Azure Monitor 更新和配置管理”扩展](media/azure-stack-marketplace-azure-items/cse.png) | [“Azure Monitor 更新和配置管理”扩展](/virtual-machines/extensions/oms-windows)| “Azure Monitor 更新和配置管理”扩展可与 Log Analytics、Azure 安全中心和 Azure Sentinel 一起使用，以提供 VM 监视功能。 **请下载此更新，此更新针对用于 Windows 的 Monitoring Agent 扩展的内置版本。** | Microsoft | Windows |
|![自定义脚本扩展](media/azure-stack-marketplace-azure-items/cse.png) | - [自定义脚本扩展（版本 1，已弃用）](/virtual-machines/extensions/custom-script-linuxostc)</b> - [自定义脚本扩展（版本 2）](/virtual-machines/extensions/custom-script-linux) |**请将此更新下载到 Linux 的自定义脚本扩展的内置版本。此扩展有多个版本，应下载 1.5.2.1 和 2.0.x。** | Microsoft | Linux |
| ![适用于 Linux 的 VM 访问权限](media/azure-stack-marketplace-azure-items/cse.png) | [适用于 Linux 的 VM 访问权限](https://azure.microsoft.com/blog/using-vmaccess-extension-to-reset-login-credentials-for-linux-vm/)| **请下载此更新，此更新针对适用于 Linux 的 VM 访问权限扩展的内置版本。如果计划使用 Debian Linux VM，此更新很重要。** | Microsoft | Linux |

## <a name="virtual-machine-images-and-solution-templates"></a>虚拟机映像和解决方案模板

Azure Stack Hub 支持下述 Azure 市场 VM 和解决方案模板。 请根据说明单独下载任何依赖项。 SQL Server 和 Machine Learning Server 之类的应用需要适当的许可，除非已标记为“免费”或“试用”。

| 映像 | 项名称 | 说明 | 发布者 |
| --- | --- | --- | --- |
| ![SharePoint Server 2013 试用版](media/azure-stack-marketplace-azure-items/sharepoint.png) | [SharePoint Server 2013 试用版](https://market.azure.cn/zh-cn/marketplace/apps/microsoftsharepoint.microsoftsharepointserver?tab=Overview) | Windows Server 2012 Datacenter 和 Visual Studio 2019 社区版上的 Microsoft SharePoint Server 2013 试用版。 | Microsoft |
| ![SharePoint Server 2016 试用版](media/azure-stack-marketplace-azure-items/sharepoint.png) | [SharePoint Server 2016 试用版](https://market.azure.cn/zh-cn/marketplace/apps/microsoftsharepoint.microsoftsharepointserver?tab=Overview) | Windows Server 2016 Datacenter 上的 Microsoft SharePoint Server 2016 试用版。 | Microsoft |
| ![Windows Server 2012 R2 上的 SQL Server 2014 SP3](media/azure-stack-marketplace-azure-items/sql.png) | [Windows Server 2012 R2 上的 SQL Server 2014 SP3](https://market.azure.cn/zh-cn/marketplace/apps/Microsoft.SQLServer2012SP3?tab=Overview) | SQL Server 2014 Service Pack 2。 **所需的下载：** SQL IaaS 扩展。 | Microsoft |
| ![Windows Server 2016 上的 Microsoft Machine Learning Server 9.3.0](media/azure-stack-marketplace-azure-items/microsoft.png) | [Windows Server 2016 上的 Microsoft Machine Learning Server 9.3.0](https://market.azure.cn/zh-cn/marketplace/apps/Microsoft.MicrosoftMachineLearningServer930onWindowsServer2016) | Windows Server 2016 上的 Microsoft Machine Learning Server 9.3.0。 | Microsoft |
| ![Ubuntu 16.04 上的 Microsoft Machine Learning Server 9.3.0](media/azure-stack-marketplace-azure-items/microsoft.png) | [Ubuntu 16.04 上的 Microsoft Machine Learning Server 9.3.0](https://market.azure.cn/zh-cn/marketplace/apps/Microsoft.MicrosoftMachineLearningServer930onUbuntu1604) | Ubuntu 16.04 上的 Microsoft Machine Learning Server 9.3.0。 | Microsoft + Canonical |

## <a name="linux-distributions"></a>Linux 发行版

| 映像 | 项名称 | 说明 | 发布者 |
| --- | --- | --- | --- |
| ![Clear Linux OS](media/azure-stack-marketplace-azure-items/clearlinux.png) | [Clear Linux OS](https://market.azure.cn/zh-cn/marketplace/apps/CoreOS.CoreOS?tab=Overview) | 一个针对 Intel 体系结构优化的参考性 Linux 发行版 | Clear Linux Project |
| ![CoreOS Linux (Stable)](media/azure-stack-marketplace-azure-items/coreos.png) | [CoreOS Linux (Stable)](https://market.azure.cn/zh-cn/marketplace/apps/CoreOS.CoreOS?tab=Overview) | CoreOS 是一种新式的最小型 Linux 发行版，可以方便地运行容器、管理群集以及无缝地更新服务器 - 所有组件都支持仓库级计算。 | CoreOS |
| ![Ubuntu Server](media/azure-stack-marketplace-azure-items/ubuntu.png) | [Ubuntu Server](https://market.azure.cn/zh-cn/marketplace/apps/Canonical.UbuntuServer?tab=Overview) | Ubuntu Server 是全球流行的 Linux 云环境。 | Canonical |
| ![Debian 8 "Jessie"](media/azure-stack-marketplace-azure-items/debian8.png) | [Debian 8“Jessie”](https://market.azure.cn/zh-cn/marketplace/apps/credativ.Debian?tab=Overview) | Debian GNU/Linux 是最流行的 Linux 分发版之一。 | credativ |
| ![基于 CentOS 的 7.5](media/azure-stack-marketplace-azure-items/roguewave.png) | [基于 CentOS 的 7.5](https://market.azure.cn/zh-cn/marketplace/apps/RogueWave.CentOSbased?tab=Overview) | 此 Linux 发行版基于 CentOS ，由 Rogue Wave Software 提供。 | Rogue Wave Software（之前为 OpenLogic） |
| ![基于 CentOS 的 7.5-LVM](media/azure-stack-marketplace-azure-items/roguewave.png) | [基于 CentOS 的 7.5-LVM](https://market.azure.cn/zh-cn/marketplace/apps/RogueWave.CentOSbased?tab=PlansAndPrice) | 此 Linux 发行版基于 CentOS ，由 Rogue Wave Software 提供。 | Rogue Wave Software（之前为 OpenLogic） |
| ![基于 CentO 的 HPC](media/azure-stack-marketplace-azure-items/roguewave.png) | [基于 CentOS 的 HPC](https://market.azure.cn/zh-cn/marketplace/apps/openlogic.centos-hpc?tab=Overview) | 此 Linux 发行版基于 CentOS ，由 Rogue Wave Software 提供。 | Rogue Wave Software（之前为 OpenLogic）  |
| ![SLES 11 SP4 (BYOS)](media/azure-stack-marketplace-azure-items/suse.png) | [SLES 11 SP4 (BYOS)](https://market.azure.cn/zh-cn/marketplace/apps/suse.sles-byos?tab=Overview) | SUSE Linux Enterprise Server 11 SP4。 | SUSE |
| ![SLES 12 SP3 (BYOS)](media/azure-stack-marketplace-azure-items/suse.png) | [SLES 12 SP4 (BYOS)](https://market.azure.cn/zh-cn/marketplace/apps/suse.sles-byos?tab=Overview) | SUSE Linux Enterprise Server 12 SP4。 | SUSE |
| ![SLES 15 (BYOS)](media/azure-stack-marketplace-azure-items/suse.png) | [SLES 15 (BYOS)](https://market.azure.cn/zh-cn/marketplace/apps/suse.sles-byos?tab=Overview) | SUSE Linux Enterprise Server 15。 | SUSE |

## <a name="third-party-byol-free-trial-images-and-solution-templates"></a>第三方 BYOL 版、免费版和试用版映像以及解决方案模板

| 映像 | 项名称 | 说明 | 发布者 |
| --- | --- | --- | --- |
| ![A10 vThunder ADC](media/azure-stack-marketplace-azure-items/a10.png) | [A10 vThunder ADC](https://market.azure.cn/zh-cn/marketplace/apps/a10networks-cn.a10-thunder-adc-411-p2?tab=PlansAndPrice) | 适用于 Azure 的 A10 Networks vThunder ADC（应用程序交付控制器）专为实现高性能、灵活性和易于部署的应用交付和服务器负载均衡而构建，并经过优化以在 Azure 云中本机运行。 | A10 Networks |
| ![Barracuda Email Security Gateway](media/azure-stack-marketplace-azure-items/barracuda.png) | [Barracuda Email Security Gateway](https://market.azure.cn/zh-cn/marketplace/apps/vstecscloud.barracuda_email_security_gateway_2?tab=Overview) | 一种电子邮件安全网关，用于防范入站电子邮件产生的威胁。 | Barracuda Networks, Inc. |
| ![Barracuda Web 应用程序防火墙 (WAF)](media/azure-stack-marketplace-azure-items/barracuda.png) | [Barracuda Web 应用程序防火墙 (WAF)](https://market.azure.cn/zh-cn/marketplace/apps/vstecscloud.barracuda_web_application_firewall_formigration?tab=Overview) | 一种安全性和 DDoS 防护，防范自动化针对性攻击。 | Barracuda Networks, Inc. |
| ![Barracuda CloudGen 防火墙控制中心](media/azure-stack-marketplace-azure-items/barracuda.png) | [Barracuda CloudGen 防火墙控制中心](https://market.azure.cn/zh-cn/marketplace/apps/vstecscloud.cloudgen_firewall_controlcenter?tab=Overview) | 集中管理数百个 Barracuda CloudGen 防火墙，不管其位置和外形规格。 | Barracuda Networks, Inc. |
| ![用于 Azure 的 Barracuda CloudGen 防火墙](media/azure-stack-marketplace-azure-items/barracuda.png) | [用于 Azure 的 Barracuda CloudGen 防火墙](https://market.azure.cn/zh-cn/marketplace/apps/vstecscloud.barracuda_cloudgenfirewall?tab=Overview) | 在应用和数据驻留的位置提供防火墙保护，而不是仅仅在连接终止的位置提供保护。 | Barracuda Networks, Inc. |
| :::image type="icon" source="media/azure-stack-marketplace-azure-items/commvault.png" border="false"::: | [Commvault](https://market.azure.cn/zh-cn/marketplace/apps/vstecscloud.commvault_backup_suite?tab=Overview) | 一个综合性解决方案，适用于备份和恢复、将应用和 VM 迁移到 Azure Stack Hub，以及在单个解决方案中针对 Azure Stack Hub 环境进行灾难恢复。 | Commvault |
| ![FortiGate 下一代防火墙](media/azure-stack-marketplace-azure-items/fortinetsquare.png) | [FortiGate 下一代防火墙](https://market.azure.cn/zh-cn/marketplace/apps/fortinet-cn.fortinet_fortigate-vm_v6_0?tab=Overview) | 一种防火墙技术，通过一个包含强大安全功能的综合性套件提供完整的内容和网络保护。 可以协调使用应用控制、防病毒、IPS、Web 筛选、VPN 以及多种高级功能（例如漏洞管理），以便确定并缓解最新且复杂的安全威胁。 | Fortinet |
| :::image type="icon" source="media/azure-stack-marketplace-azure-items/kubernetes.png" border="false"::: | [Kubernetes](azure-stack-aks-engine.md) | 此解决方案会使用 AKS-Engine 生成的模板部署一个 Kubernetes 群集，该群集以独立群集的形式运行。<br>此解决方案模板还需要 Ubuntu Server 16.04 LTS 以及适用于 Linux 2.0 的自定义脚本。| Microsoft |
| ![Service Fabric 群集](media/azure-stack-marketplace-azure-items/servicefrabric.png) | [Service Fabric 群集](https://market.azure.cn/zh-cn/marketplace/apps/Microsoft.ServiceFabricCluster?tab=Overview) | 此解决方案将 Service Fabric 部署为在虚拟机规模集上运行的独立群集。 <br>**此解决方案模板还需要下载 Windows Server 2016 Datacenter**| Microsoft |
| ![Palo Alto VM 系列下一代防火墙](media/azure-stack-marketplace-azure-items/paloalto.png) | [Palo Alto VM 系列下一代防火墙](https://market.azure.cn/zh-cn/marketplace/apps/WestconSolutionsChina.vmseries800?tab=Overview) | VM 系列下一代防火墙可以让客户安全地将其应用和数据迁移到 Azure Stack Hub，使用应用允许列表和威胁防范策略保护客户免受已知和未知的威胁。 **此映像需要模板来部署；请参阅此[文章](https://www.paloaltonetworks.com/documentation/81/virtualization/virtualization/set-up-the-vm-series-firewall-on-azure/deploy-the-vm-series-firewalls-on-azure-stack)以获取重要信息。**| Palo Alto Networks, Inc. |
| ![Quest Rapid Recovery](media/azure-stack-marketplace-azure-items/quest.png) | [Quest Rapid Recovery Core](https://market.azure.cn/zh-cn/marketplace/apps/questchina.rapidrecoverycore?tab=Overview) | Rapid Recovery 高级数据保护在单个易用的软件解决方案中集中了备份、复制和恢复功能。 | Quest Software |
| ![SIOS DataKeeper Cluster Edition](media/azure-stack-marketplace-azure-items/sioslogo.png) | [SIOS DataKeeper Cluster Edition](https://market.azure.cn/zh-cn/marketplace/apps/vstecscloud.sios_datakeeper_cluster_edition?tab=Overview) | SIOS DataKeeper 在 Azure Stack Hub 中提供高可用性 (HA) 和灾难恢复 (DR)。 只需将 SIOS DataKeeper 软件作为组件添加到 Azure Stack Hub 部署中的 Windows Server 故障转移群集 (WSFC) 环境即可，无需共享存储。 | SIOS Technology Corp. |
| ![SUSE Manager 3.1 Proxy (BYOS)](media/azure-stack-marketplace-azure-items/suse.png) | [SUSE Manager 3.1 Proxy (BYOS)](https://market.azure.cn/zh-cn/marketplace/apps/suse.suse-manager-proxy-byos?tab=Overview) | 同类最佳开源基础结构管理。 | SUSE |
| ![Veeam Backup & Replication](media/azure-stack-marketplace-azure-items/veeam.png) | [Veeam Backup & Replication](https://market.azure.cn/zh-cn/marketplace/apps/vstecscloud.veeam-backup-replication?tab=Overview) | Veeam&reg; Backup & Replication&trade; 帮助企业实现对所有工作负荷（虚拟的、物理的和基于云的）的综合性数据保护。 可以通过单个控制台对所有应用和数据进行快速、灵活且可靠的备份、恢复和复制。 | Veeam Software |
