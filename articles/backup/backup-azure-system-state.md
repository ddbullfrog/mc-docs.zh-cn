---
title: 将 Windows 系统状态备份到 Azure
description: 了解如何将 Windows Server 计算机的系统状态备份到 Azure。
ms.topic: conceptual
author: Johnnytechn
origin.date: 05/23/2018
ms.date: 09/28/2020
ms.author: v-johya
ms.openlocfilehash: a60d79797b5ce686dd76e14b73bdd51f13791d7b
ms.sourcegitcommit: 80567f1c67f6bdbd8a20adeebf6e2569d7741923
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871406"
---
# <a name="back-up-windows-system-state-to-azure"></a>将 Windows 系统状态备份到 Azure

本文介绍了如何将 Windows Server 系统状态备份到 Azure。 它旨在引导完成基本操作。

如果想要深入了解 Azure 备份，请阅读此 [概述](backup-overview.md)。

如果没有 Azure 订阅，可以先创建一个 [试用帐户](https://www.azure.cn/pricing/1rmb-trial/) ，这样就可以访问任何 Azure 服务。

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="set-storage-redundancy-for-the-vault"></a>为保管库设置存储冗余

在创建恢复服务保管库时，请确保存储冗余配置为所希望的方式。
<!--Customized from backup-create-rs-vault.md-->

1. 从“恢复服务保管库”边栏选项卡中，单击新保管库  。 在“设置”部分下，单击“属性” 。
2. 在“属性”中，在“备份配置”下，单击“更新”。  

3. 选择存储复制类型，然后单击“保存”。

     ![设置新保管库的存储配置](./media/backup-try-azure-backup-in-10-mins/recovery-services-vault-backup-configuration.png)

   - 如果使用 Azure 作为主要备份存储终结点，则我们建议继续使用默认的“异地冗余”设置。
   - 如果不使用 Azure 作为主要的备份存储终结点，则请选择“本地冗余”，减少 Azure 存储费用。
   - 详细了解[异地冗余](../storage/common/storage-redundancy-grs.md)和[本地冗余](../storage/common/storage-redundancy-lrs.md)。

> [!NOTE]
> 在保管库中配置备份之前，必须更改恢复服务保管库的存储复制类型（本地冗余/异地冗余）。 配置备份后，将禁用修改选项。
>
>- 如果尚未配置备份，请遵循[这些步骤](/backup/backup-create-rs-vault#set-storage-redundancy)检查和修改设置。
>- 如果已配置备份，并且必须从 GRS 迁移到 LRS，请查看[解决方法](backup-azure-backup-faq.md)。

<!--Customized from backup-create-rs-vault.md-->
创建保管库之后，即可对其进行配置，以便备份 Windows 系统状态。

## <a name="configure-the-vault"></a>配置保管库

<!--Customized in MC-->
<!--Not available in MC: Open backup goal blade-->

1. 在“恢复服务保管库”窗格（对应于刚创建的保管库）的“开始”部分，选择“备份”。

    此时将打开“备份”窗格。

    ![打开备份目标窗格](./media/backup-try-azure-backup-in-10-mins/backup-goal-blade.png)

2. 从“工作负荷的运行位置?”下拉菜单中选择“本地” 。

    之所以选择“本地”，是因为 Windows Server 或 Windows 计算机是不在 Azure 中的物理计算机。

3. 从“要备份哪些内容?”菜单中选择“系统状态”。 

    ![配置文件和文件夹](./media/backup-azure-system-state/backup-goal-system-state.png)

    选择“系统状态”后，将打开“准备基础结构”窗格 。

    ![已配置备份目标，接下来准备基础结构](./media/backup-try-azure-backup-in-10-mins/backup-goal-configed.png)

4. 在“准备基础结构”窗格上，选择“下载适用于 Windows Server 或 Windows 客户端的代理” 。

    ![准备基础结构](./media/backup-try-azure-backup-in-10-mins/choose-agent-for-server-client.png)

    如果使用的是 Windows Server Essential，则选择下载适用于 Windows Server Essential 的代理。 弹出菜单会提示用户运行或保存 MARSAgentInstaller.exe。

    ![MARSAgentInstaller 对话框](./media/backup-try-azure-backup-in-10-mins/mars-installer-run-save.png)

5. 在下载弹出窗口中选择“保存”。

    默认情况下，**MARSagentinstaller.exe** 文件将保存到 Downloads 文件夹。 下载完安装程序以后，会显示一个弹出窗口，询问用户是要运行安装程序，还是要打开文件夹。

    ![MARS 安装程序已完成](./media/backup-try-azure-backup-in-10-mins/mars-installer-complete.png)

    此时还不需要安装代理。 下载保管库凭据之后，即可安装代理。

6. 在“准备基础结构”窗格上，选择“下载” 。

    ![下载保管库凭据](./media/backup-try-azure-backup-in-10-mins/download-vault-credentials.png)

    保管库凭据下载到“下载”文件夹。 下载完保管库凭据以后，会显示一个弹出窗口，询问用户是要打开还是要保存凭据。 选择“保存”。  如果意外选择了“打开”，可以让尝试打开保管库凭据的对话框关闭。 你将无法打开保管库凭据。 继续执行下一步。 保管库凭据位于“下载”文件夹中。

    ![已下载保管库凭据](./media/backup-try-azure-backup-in-10-mins/vault-credentials-downloaded.png)
   > [!NOTE]
   > 保管库凭据只能保存到打算使用代理的 Windows Server 的本地位置。
   >

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

<!--Customized in MC-->
## <a name="install-and-register-the-agent"></a>安装并注册代理

> [!NOTE]
> 无法通过 Azure 门户启用备份功能。 使用 Azure 恢复服务代理备份 Windows Server 的系统状态。
>

1. 在 Downloads 文件夹（或其他保存位置）中找到并双击 **MARSagentinstaller.exe**。

    安装程序会在提取、安装和注册恢复服务代理时提供一系列消息。

    ![运行恢复服务代理安装程序凭据](./media/backup-try-azure-backup-in-10-mins/mars-installer-registration.png)

2. 完成 Azure 恢复服务代理安装向导。 若要完成该向导，需完成以下操作：

   * 选择安装和缓存文件夹的位置。
   * 如果使用代理服务器来连接 Internet，请提供代理服务器信息。
   * 如果使用经过身份验证的代理，请提供用户名和密码详细信息。
   * 提供已下载的保管库凭据
   * 将加密通行短语保存在安全的位置。

     > [!NOTE]
     > 如果丢失或忘记了通行短语，Azure 无法帮助你恢复备份数据。 请将文件保存在安全的位置。 还原备份时需要用到此文件。
     >
     >

现已安装代理，且已向保管库注册计算机。 接下来可以配置和计划备份。

## <a name="back-up-windows-server-system-state"></a>备份 Windows Server 系统状态

初始备份包括两个任务：

* 计划备份
* 首次备份系统状态

若要完成初始备份，请使用 Azure 恢复服务代理。

> [!NOTE]
> 可以在 Windows Server 2008 R2 到 Windows Server 2016 上备份系统状态。 客户端 SKU 不支持系统状态备份。 系统状态不会显示为 Windows 客户端或 Windows Server 2008 SP2 计算机选项。
>
>

### <a name="to-schedule-the-backup-job"></a>计划备份作业

1. 打开 Azure 恢复服务代理。 可以通过在计算机中搜索“Azure 备份”找到该代理。

    ![启动 Azure 恢复服务代理](./media/backup-try-azure-backup-in-10-mins/snap-in-search.png)

2. 在恢复服务代理中，选择“计划备份”。

    ![计划 Windows Server 备份](./media/backup-try-azure-backup-in-10-mins/schedule-first-backup.png)

3. 在计划备份向导的“开始使用”页上，选择“下一步” 。

4. 在“选择要备份的项”页上，选择“添加项” 。

5. 选择“系统状态”，并选择“确定” 。

6. 选择“**下一页**”。

7. 在后续页中选择系统状态备份所需的备份频率和保留策略。

8. 在“确认”页上复查信息，然后选择“完成”。

9. 在向导完成创建备份计划后，选择“关闭”。

### <a name="to-back-up-windows-server-system-state-for-the-first-time"></a>首次备份 Windows Server 系统状态

1. 请确保需要重启的 Windows Server 没有挂起的更新。

2. 在恢复服务代理中选择“立即备份”，以通过网络完成初始种子设定。

    ![立即备份 Windows Server](./media/backup-try-azure-backup-in-10-mins/backup-now.png)

3. 在出现的“选择备份项”屏幕上选择“系统状态”，然后选择“下一步”  。

4. 在“确认”页上复查“立即备份向导”用于备份计算机的设置。 然后选择“备份”。

5. 选择“关闭”以关闭向导。 如果在备份过程完成之前关闭向导，向导将继续在后台运行。
    > [!NOTE]
    > 在每次系统状态备份之前，MARS 代理都会在预检查过程中触发 `SFC /verifyonly`。 这是为了确保作为系统状态一部分备份的文件具有与 Windows 版本相对应的正确版本。 在[本文](https://docs.microsoft.com/windows-server/administration/windows-commands/sfc)中详细了解系统文件检查器 (SFC)。
    >

完成初始备份后，备份控制台中显示“**作业已完成**”状态。

  ![IR 完成](./media/backup-try-azure-backup-in-10-mins/ircomplete.png)


<!--Not available in MC: https://feedback.azure.com/forums/258995-azure-backup-->
## <a name="next-steps"></a>后续步骤

* 详细了解如何 [备份 Windows 计算机](backup-windows-with-mars-agent.md)。
* 至此，你已备份 Windows Server 系统状态，接下来可以[管理保管库和服务器](backup-azure-manage-windows-server.md)了。
* 如果需要还原备份，请参阅[将文件还原到 Windows 计算机](backup-azure-restore-windows-server.md)一文。

<!-- Update_Description: wording update -->
