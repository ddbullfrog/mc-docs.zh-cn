---
title: 从 Azure 自动化更新管理中删除 VM
description: 本文介绍如何删除使用“更新管理”管理的计算机。
services: automation
ms.topic: conceptual
origin.date: 09/09/2020
ms.date: 09/28/2020
ms.custom: mvc
ms.openlocfilehash: 093822389b736178090a01e0dea19dec75f1bfc5
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246824"
---
# <a name="remove-vms-from-update-management"></a>从“更新管理”中删除 VM

完成对环境中 VM 的更新管理后，可以停止通过[更新管理](update-mgmt-overview.md)功能管理 VM。 若要停止管理 VM，需在关联到自动化帐户的 Log Analytics 工作区中编辑已保存的搜索查询 `MicrosoftDefaultComputerGroup`。

## <a name="sign-into-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.cn)。

## <a name="to-remove-your-vms"></a>删除 VM


1. 使用以下命令识别希望从管理中删除的计算机的 UUID。

    ```azurecli
    az vm show -g MyResourceGroup -n MyVm -d
    ```

2. 在 Azure 门户中，导航到 Log Analytics 工作区。 从列表中选择你的工作区。

3. 在 Log Analytics 工作区中，选择“日志”，然后从顶部的操作菜单中选择“查询资源管理器”。

4. 从右侧窗格中的查询资源管理器展开“已保存的查询\更新”，然后选择已保存的搜索查询 `MicrosoftDefaultComputerGroup` 进行编辑。

5. 在查询编辑器中，查看该查询并找到 VM 的 UUID。 删除 VM 的 UUID，并对要删除的任何其他 VM 重复这些步骤。

6. 完成编辑操作后，通过从顶部栏中选择“保存”来保存搜索。

>[!NOTE]
>取消注册后，系统仍会显示这些计算机，因为我们会报告在过去 24 小时内评估的所有计算机。 删除计算机后，需要等待 24 小时，系统才不会再次列出这些计算机。

## <a name="next-steps"></a>后续步骤

若要重新启用对虚拟机的管理，请参阅[从 Azure VM 启用更新管理](update-mgmt-enable-vm.md)。