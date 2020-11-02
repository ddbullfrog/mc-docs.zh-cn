---
title: 适用于 Windows 的 Azure 诊断扩展
description: 使用 Azure 诊断扩展监视 Azure Windows VM
manager: ashwink
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
origin.date: 04/06/2018
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 49e415c9990bf19f9b8f367e01555446f55a3b95
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105762"
---
# <a name="azure-diagnostics-extension-for-windows-vms"></a>适用于 Windows VM 的 Azure 诊断扩展

## <a name="overview"></a>概述

Azure 诊断 VM 扩展可用来从 Windows VM 收集监视数据，例如性能计数器和事件日志。 你可以精细地指定要收集什么数据以及希望该数据出现在何处，例如 Azure 存储帐户或 Azure 事件中心。 还可以使用该数据在 Azure 门户中构建图表或创建指标警报。

## <a name="prerequisites"></a>先决条件

### <a name="operating-system"></a>操作系统

Azure 诊断扩展可以针对 Windows 10 客户端，Windows Server 2008 R2、2012、2012 R2 和 2016 运行。

### <a name="internet-connectivity"></a>Internet 连接

Azure 诊断扩展要求目标虚拟机已连接到 Internet。 

## <a name="extension-schema"></a>扩展架构

[本文档介绍了 Azure 诊断扩展架构和属性值。](../../azure-monitor/platform/diagnostics-extension-schema-windows.md)

## <a name="template-deployment"></a>模板部署

可使用 Azure Resource Manager 模板部署 Azure VM 扩展。 可在 Azure 资源管理器模板中使用上一部分中详细说明的 JSON 架构，以在 Azure 资源管理器模板部署期间运行 Azure 诊断扩展。 请参阅[将监视和诊断与 Windows VM 和 Azure 资源管理器模板配合使用](../extensions/diagnostics-template.md)。

## <a name="azure-cli-deployment"></a>Azure CLI 部署

可以使用 Azure CLI 将 Azure 诊断扩展部署到现有的虚拟机。 请将受保护的设置和设置属性替换为以上扩展架构中的有效 JSON。 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name IaaSDiagnostics \
  --publisher Microsoft.Azure.Diagnostics \
  --version 1.9.0.0 --protected-settings protected-settings.json \
  --settings public-settings.json 
```

## <a name="powershell-deployment"></a>PowerShell 部署

可以使用 `Set-AzVMDiagnosticsExtension` 命令将 Azure 诊断扩展添加到现有的虚拟机。 另请参阅[使用 PowerShell 在运行 Windows 的虚拟机中启用 Azure 诊断](../extensions/diagnostics-windows.md)。

```powershell
$vm_resourcegroup = "myvmresourcegroup"
$vm_name = "myvm"
$diagnosticsconfig_path = "DiagnosticsPubConfig.xml"

Set-AzVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup `
  -VMName $vm_name `
  -DiagnosticsConfigurationPath $diagnosticsconfig_path
```

## <a name="troubleshoot-and-support"></a>故障排除和支持

### <a name="troubleshoot"></a>故障排除

有关扩展部署状态的数据可以从 Azure 门户和使用 Azure CLI 进行检索。 若要查看给定 VM 的扩展部署状态，请使用 Azure CLI 运行以下命令。

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

有关 Azure 诊断扩展的更全面的故障排除指南，请[参阅此文章](../../azure-monitor/platform/diagnostics-extension-troubleshooting.md)。

### <a name="support"></a>支持

如果对本文中的任何观点存在疑问，可以联系 [Azure 支持](https://support.azure.cn/support/contact/)上的 Azure 专家。 或者，也可以提出 Azure 支持事件。 请转到 [Azure 支持站点](https://support.azure.cn/support/support-azure/)提交请求。 有关使用 Azure 支持的信息，请阅读 [Azure 支持常见问题](https://www.azure.cn/support/faq/)。

## <a name="next-steps"></a>后续步骤
* [详细了解 Azure 诊断扩展](../../azure-monitor/platform/diagnostics-extension-overview.md)
* [查看扩展架构和版本](../../azure-monitor/platform/diagnostics-extension-schema-windows.md)

<!-- Update_Description: update meta properties, wording update, update link -->