---
title: Azure 启动诊断
description: Azure 启动诊断和托管启动诊断概述
services: virtual-machines
ms.service: virtual-machines
ms.topic: conceptual
origin.date: 08/04/2020
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: bbfc16d491381c3040535d3ec2ddc0a46312d767
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105132"
---
# <a name="azure-boot-diagnostics"></a>Azure 启动诊断

启动诊断是 Azure 虚拟机 (VM) 的一项调试功能，可用于诊断 VM 启动故障。 启动诊断使用户能够通过收集串行日志信息和屏幕截图来观察其 VM 在启动时的状态。

## <a name="boot-diagnostics-storage-account"></a>启动诊断存储帐户
在 Azure 门户中创建 VM 时，默认情况下会启用“启动诊断”。 建议的启动诊断体验是使用托管存储帐户，因为它可以在创建 Azure VM 时显著提高性能。 这是因为将要使用 Azure 托管存储帐户，从而节省了创建新的用户存储帐户来存储启动诊断数据所需的时间。

另一种启动诊断体验是使用用户管理的存储帐户。 用户可以创建新的存储帐户，也可以使用现有的存储帐户。

> [!IMPORTANT]
> 将不会向 Azure 客户收取与使用托管存储帐户的启动诊断相关的存储费用，直至 2020 年 10 月底。
>
> 启动诊断数据 blob（包括日志和快照映像）存储在托管存储帐户中。 将仅对 blob 使用的 GiB 向客户收费，而不是根据磁盘的预配大小收费。 快照计量将用于托管存储帐户的计费。 由于托管帐户是在标准 LRS 或标准 ZRS 上创建的，因此，仅针对诊断数据 blob 的大小按每月 $0.05/GB 向客户收费。 有关此定价的详细信息，请参阅[托管磁盘定价](https://www.azure.cn/pricing/details/storage/managed-disks/)。 客户将看到这一费用与其 VM 资源 URI 相关联。 

## <a name="boot-diagnostics-view"></a>启动诊断视图
虚拟机边栏选项卡中的启动诊断选项位于 Azure 门户的“支持和故障排除”部分。 选择启动诊断会显示屏幕截图和串行日志信息。 串行日志包含内核消息，屏幕快照是 VM 当前状态的快照。 Windows 或 Linux 会根据 VM 是否正在运行来确定预期的屏幕快照会是什么样子。 Windows 用户会看到桌面背景，Linux 用户会看到登录提示。

:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-linux.png" alt-text="Linux 启动诊断的屏幕截图":::
:::image type="content" source="./media/boot-diagnostics/boot-diagnostics-windows.png" alt-text="Windows 启动诊断的屏幕截图":::

## <a name="limitations"></a>限制
- 启动诊断仅适用于 Azure 资源管理器 VM。 
- 启动诊断不支持高级存储帐户。如果将高级存储帐户用于启动诊断，则在启动 VM 时，用户会收到 `StorageAccountTypeNotSupported` 错误。 
- 资源管理器 API 版本“2020-06-01”及更高版本支持托管存储帐户。
- 当前只能通过 Azure 门户应用使用托管存储帐户的启动诊断。 

<!--Not Available on - Azure Serial Console is currently incompatible with a managed storage account for Boot Diagnostics. Learn more about [Azure Serial Console](./troubleshooting/serial-console-overview.md)-->

## <a name="next-steps"></a>后续步骤

详细了解如何使用启动诊断功能来[排查 Azure 中虚拟机的问题](./troubleshooting/boot-diagnostics.md)。

<!--Not Available on the [Azure Serial Console](/virtual-machines/troubleshooting/serial-console-overview)-->
<!-- Update_Description: update meta properties, wording update, update link -->