---
title: 使用 MARS 代理备份 Windows 计算机
description: 使用 Microsoft Azure 恢复服务 (MARS) 代理备份 Windows 计算机。
ms.topic: conceptual
author: lingliw
origin.date: 06/04/2019
ms.date: 03/06/2020
ms.author: v-lingwu
ms.openlocfilehash: ec2135ed096d4bb04134407820bceecbe652f4a3
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470562"
---
# <a name="back-up-windows-machines-by-using-the-azure-backup-mars-agent"></a>使用 Azure 备份 MARS 代理备份 Windows 计算机

本文介绍如何使用 [Azure 备份](backup-overview.md)服务和 Microsoft Azure 恢复服务 (MARS) 代理备份 Windows 计算机。 MARS 也称为 Azure 备份代理。

在本文中，学习如何：

> [!div class="checklist"]
>
> * 验证先决条件，并创建恢复服务保管库。
> * 下载并设置 MARS 代理。
> * 创建备份策略和计划。
> * 执行按需备份。

## <a name="about-the-mars-agent"></a>关于 MARS 代理

Azure 备份使用 MARS 代理备份本地计算机和 Azure VM 中的文件、文件夹和系统状态。 这些备份存储在 Azure 中的恢复服务保管库中。 可在以下位置运行代理：

* 直接在本地 Windows 计算机上。 这些计算机可直接备份到 Azure 中的备份恢复服务保管库。
* 在运行 Windows（与 Azure VM 备份扩展一起运行）的 Azure VM 上。 代理将备份 VM 上的特定文件和文件夹。
* 在 Microsoft Azure 备份服务器 (MABS) 实例或 System Center Data Protection Manager (DPM) 服务器上。 在此方案中，计算机和工作负荷将备份到 MABS 或 Data Protection Manager。 然后 MABS 或 Data Protection Manager 将通过 MARS 代理备份到 Azure 中的保管库。

可备份的数据取决于代理的安装位置。

> [!NOTE]
> 通常我们会使用 VM 上的 Azure 备份扩展来备份 Azure VM。 此方法将备份整个 VM。 若要备份 VM 上的特定文件和文件夹，请安装 MARS 代理并将其与该扩展一起使用。 有关详细信息，请参阅[内置 Azure VM 备份的体系结构](backup-architecture.md#architecture-built-in-azure-vm-backup)。

![备份过程的步骤](./media/backup-configure-vault/initial-backup-process.png)

## <a name="before-you-start"></a>开始之前

* 了解 [Azure 备份如何使用 MARS 代理备份 Windows 计算机](backup-architecture.md#architecture-direct-backup-of-on-premises-windows-server-machines-or-azure-vm-files-or-folders)。
* 了解在辅助 MABS 或 Data Protection Manager 服务器上运行 MARS 代理的[备份体系结构](backup-architecture.md#architecture-back-up-to-dpmmabs)。
* 查看 MARS 代理[支持的操作以及可备份的内容](backup-support-matrix-mars-agent.md)。
* 如果需要将服务器或客户端备份到 Azure，请确保有一个 Azure 帐户。 如果你没有帐户，只需花费几分钟就能创建一个[试用帐户](https://wd.azure.cn/zh-cn/pricing/1rmb-trial-full)。
* 验证要备份的计算机的 Internet 访问权限。

### <a name="verify-internet-access"></a>验证 Internet 访问

如果计算机的 Internet 访问状态受限，请确保计算机或代理上的防火墙设置允许以下 URL 和 IP 地址：

#### <a name="urls"></a>URL

- www\.msftncsi.com
- *.Microsoft.com
- *.WindowsAzure.cn
- *.microsoftonline.com
- *.chinacloudapi.cn

* IP 地址

* 20.190.128.0/18
* 40.126.0.0/18

对上面列出的所有 URL 和 IP 地址的访问都使用端口 443 上的 HTTPS 协议。

## <a name="create-a-recovery-services-vault"></a>创建恢复服务保管库

恢复服务保管库存储你在不同时间创建的所有备份和恢复点。 它还包含针对已备份的计算机的备份策略。 

若要创建保管库：

1. 使用 Azure 订阅登录到 [Azure 门户](https://portal.azure.cn/)。

1. 搜索并选择“恢复服务保管库”  。

    ![选择恢复服务保管库](./media/backup-configure-vault/open-recovery-services-vaults.png)

1. 在“恢复服务保管库”菜单中，选择“添加”。  

    ![添加恢复服务保管库](./media/backup-try-azure-backup-in-10-mins/rs-vault-menu.png)

1. 对于“名称”，请输入一个友好名称以标识保管库  。

   * 名称对于 Azure 订阅需要是唯一的。
   * 该名称可以包含 2 到 50 个字符。
   * 必须以字母开头。
   * 该名称只能包含字母、数字和连字符。

1. 选择要在其中创建保管库的 Azure 订阅、资源组和地理区域。 备份数据将发送到保管库。 选择“创建”  。

    ![添加恢复服务保管库的标识信息](./media/backup-try-azure-backup-in-10-mins/rs-vault-step-3.png)

    系统可能需要花费一段时间来创建保管库。 可以在门户的右上区域中监视状态通知。 如果在几分钟后未看到该保管库，请选择“刷新”。 

    ![选择“刷新”按钮](./media/backup-try-azure-backup-in-10-mins/refresh-button.png)

### <a name="set-storage-redundancy"></a>设置存储冗余

Azure 备份会自动处理保管库的存储。 指定如何复制该存储。

1. 在“恢复服务保管库”下，选择新保管库。  在“设置”下，选择“属性”。  
1. 在“属性”中的“备份配置”下，选择“更新”。   

1. 选择存储复制类型，然后选择“保存”。 

      ![设置新保管库的存储配置](./media/backup-try-azure-backup-in-10-mins/recovery-services-vault-backup-configuration.png)

如果使用 Azure 作为主要备份存储终结点，则我们建议继续使用默认的“异地冗余”设置。  如果不使用 Azure 作为主要的备份存储终结点，请选择“本地冗余”，以减少 Azure 存储费用。 

有关详细信息，请参阅[异地冗余](../storage/common/storage-redundancy.md#geo-redundant-storage)和[本地冗余](../storage/common/storage-redundancy.md#locally-redundant-storage)。

## <a name="download-the-mars-agent"></a>下载 MARS 代理

下载 MARS 代理，以便可将其安装到所要备份的计算机上。

如果已在任何计算机上安装该代理，请确保运行该代理的最新版本。 在门户中找到最新版本，或直接转到[下载](https://aka.ms/azurebackup_agent)。

1. 在保管库中的“开始使用”下，选择“备份”。  

    ![打开备份代理](./media/backup-try-azure-backup-in-10-mins/open-backup-settings.png)

1. 在“工作负荷的运行位置?”下，选择“本地”。   即使你要在 Azure VM 上安装 MARS 代理，也应选择此选项。
1. 在“要备份哪些内容?”下，选择“文件和文件夹”。   还可以选择“系统状态”。  还有其他一些可用选项，但仅当运行的是辅助备份服务器时，这些选项才受支持。 选择“准备基础结构”。 

    ![配置文件和文件夹](./media/backup-try-azure-backup-in-10-mins/set-file-folder.png)

1. 对于“准备基础结构”，请在“安装恢复服务代理”下，下载 MARS 代理。  

    ![准备基础结构](./media/backup-try-azure-backup-in-10-mins/choose-agent-for-server-client.png)

1. 在下载菜单中选择“保存”。  默认情况下， *MARSagentinstaller.exe* 文件将保存到 Downloads 文件夹。

1. 选择“已下载或使用最新的恢复服务代理”，然后下载保管库凭据。 

    ![下载保管库凭据](./media/backup-try-azure-backup-in-10-mins/download-vault-credentials.png)

1. 选择“保存”  。 文件将下载到“下载”文件夹。 无法打开保管库凭据文件。

## <a name="install-and-register-the-agent"></a>安装并注册代理

1. 在要备份的计算机上运行 *MARSagentinstaller.exe* 文件。
1. 在 MARS 代理安装向导中选择“安装设置”。  在此处选择代理的安装位置以及缓存位置。 然后，选择“下一步”  。
   * Azure 备份在将数据快照发送到 Azure 之前，会使用缓存来存储这些快照。
   * 缓存位置的可用空间应该至少为所要备份的数据大小的 5%。

    ![在 MARS 代理安装向导中选择安装设置](./media/backup-configure-vault/mars1.png)

1. 对于“代理配置”中，请指定 Windows 计算机上运行的代理如何连接到 Internet。  然后，选择“下一步”  。

   * 如果使用自定义代理，请指定任何所需的代理设置和凭据。
   * 请记住，代理需要有权访问[特定的 URL](#before-you-start)。

    ![在 MARS 向导中设置 Internet 访问](./media/backup-configure-vault/mars2.png)

1. 对于“安装”，请复查先决条件，然后选择“安装”。  
1. 安装代理后，选择“继续注册”。 
1. 在“注册服务器向导” > “保管库标识”中，浏览并选择已下载的凭据文件。   然后，选择“下一步”  。

    ![使用注册服务器向导添加保管库凭据](./media/backup-configure-vault/register1.png)

1. 在“加密设置”页上，指定用于加密和解密计算机备份的通行短语。 

    * 将通行短语保存在安全位置。 还原备份时需要用到它。
    * 如果你丢失或忘记了该通行短语，Microsoft 将无法帮助你恢复备份数据。 

1. 选择“完成”。  现已安装代理，且已向保管库注册计算机。 接下来可以配置和计划备份。

## <a name="create-a-backup-policy"></a>创建备份策略

备份策略指定何时创建数据快照来创建恢复点。 它还指定恢复点的保留期限。 使用 MARS 代理配置备份策略。
 
Azure 备份不会自动考虑夏令时 (DST)。 此默认设置可能会导致实际时间与计划的备份时间存在一定的偏差。

创建备份策略：
1. 下载并注册 MARS 代理后，打开代理控制台。 可以通过在计算机中搜索 **Microsoft Azure 备份** 找到该代理。  
1. 在“操作”下，选择“计划备份”。  

    ![计划 Windows Server 备份](./media/backup-configure-vault/schedule-first-backup.png)
1. 在计划备份向导中，选择“开始” > “下一步”。  
1. 在“选择要备份的项”下，选择“添加项”。  

    ![添加要备份的项](./media/backup-azure-manage-mars/select-item-to-backup.png)

1. 在“选择项”框中，选择要备份的项，然后选择“确定”。  

    ![选择要备份的项](./media/backup-azure-manage-mars/selected-items-to-backup.png)

1. 在“选择要备份的项”页上，选择“下一步”。  
1. 在“指定备份计划”页上，指定何时创建每日或每周备份。  然后，选择“下一步”  。

    * 创建备份时会创建一个恢复点。
    * 在环境中创建的恢复点数目取决于备份计划。
    * 可以计划每日备份，每天最多备份三次。 在以下示例中，每日会执行两次备份，一次发生在午夜，另一次发生在下午 6:00。

        ![设置每日备份计划](./media/backup-configure-vault/day-schedule.png)


    * 也可以运行每周备份。 在以下示例中，将每隔两周在星期日和星期三上午 9:30 和凌晨 1:00 创建备份。

        ![设置每周备份计划](./media/backup-configure-vault/week-schedule.png)


1. 在“选择保留策略”页上，指定如何存储数据的历史副本。  然后，选择“下一步”  。

    * 保留设置指定要存储哪些恢复点，以及要存储多长时间。 
    * 对于每日保留设置，可以指明在针对每日保留指定的时间，要将最新恢复点保留指定的天数。 或者，可以指定每月保留策略，指明在每个月的 30 日创建的恢复点应保留 12 个月。
    * 每日和每周恢复点的保留期通常与备份计划相一致。 因此，当计划触发备份时，备份创建的恢复点的存储持续时间与每日或每周保留策略指定的持续时间相一致。
    * 在以下示例中：

        * 在午夜和下午 6:00 创建的每日备份将保留 7 天。
        * 在星期六的午夜和下午 6:00 创建的备份将保留 4 周。
        * 在当月最后一个星期六的午夜和下午 6:00 创建的备份将保留 12 个月。
        * 在三月份最后一个星期六创建的备份将保留 10 年。

        ![保留策略的示例](./media/backup-configure-vault/retention-example.png)


1. 在“选择初始备份类型”页上，确定如何通过网络或使用脱机备份创建初始备份。  若要通过网络创建初始备份，请选择“自动通过网络” > “下一步”。  

    ![选择初始备份类型](./media/backup-azure-manage-mars/choose-initial-backup-type.png)


1. 在“确认”页上复查信息，然后选择“完成”。  

    ![确认备份类型](./media/backup-azure-manage-mars/confirm-backup-type.png)


1. 在向导完成创建备份计划后，选择“关闭”。 

    ![查看备份计划的进度](./media/backup-azure-manage-mars/confirm-modify-backup-process.png)

在安装了代理的每台计算机上创建一个策略。

### <a name="do-the-initial-backup-offline"></a>执行脱机初始备份

可以通过网络自动运行初始备份，也可以使用脱机备份。 如果需要消耗大量的网络带宽来传输大量的数据，则初始备份的脱机种子设定非常有用。 

若要执行脱机传输：

1. 将备份数据写入暂存位置。
1. 使用 AzureOfflineBackupDiskPrep 工具将暂存位置中的数据复制到一个或多个 SATA 磁盘。 

    该工具会创建 Azure 导入作业。 有关详细信息，请参阅[什么是 Azure 导入/导出服务](/storage/common/storage-import-export-service)。

1. 将 SATA 磁盘寄送到 Azure 数据中心。 

    在数据中心，磁盘数据将复制到 Azure 存储帐户。 Azure 备份将数据从存储帐户复制到保管库，并计划增量备份。

### <a name="enable-network-throttling"></a>启用网络限制

可以通过启用网络限制，来控制 MARS 代理使用网络带宽的方式。 如果你需要在工作时间备份数据，但想要控制备份和还原活动占用的带宽量，则限制会很有帮助。

Azure 备份中的网络限制在本地操作系统上使用[服务质量 (QoS)](https://docs.microsoft.com/windows-server/networking/technologies/qos/qos-policy-top)。

针对备份的网络限制适用于 Windows Server 2012 和更高版本，以及 Windows 8 和更高版本。 操作系统应该运行最新的服务包。

若要启用网络限制：

1. 在 MARS 代理中，选择“更改属性”。 
1. 在“限制”  选项卡上，选择“为备份操作启用 Internet 带宽使用限制”  。

    ![针对备份操作设置网络限制](./media/backup-configure-vault/throttling-dialog.png)
1. 指定在工作时间和下班时间允许的带宽。 带宽值最小为 512 Kbps，最大为 1,023 MBps。 然后选择“确定”。 

## <a name="run-an-on-demand-backup"></a>运行按需备份

1. 在 MARS 代理中，选择“立即备份”。 

    ![在 Windows Server 中立即备份](./media/backup-configure-vault/backup-now.png)

1. 如果 MARS 代理版本为 2.0.9169.0 或更高，则你可以设置自定义保留日期。 在“备份保留截止日期”部分，从日历中选择一个日期。 

   ![使用日历自定义保留日期](./media/backup-configure-vault/mars-ondemand.png)

1. 在“确认”页上复查设置，然后选择“备份”。  
1. 选择“关闭”以关闭向导。  如果在备份完成之前关闭向导，向导将继续在后台运行。

完成初始备份后，备份控制台中显示“作业已完成”状态。 

## <a name="set-up-on-demand-backup-policy-retention-behavior"></a>设置按需备份策略保留行为

> [!NOTE]
> 此信息仅适用于低于 2.0.9169.0 的 MARS 代理版本。
>

| 备份计划选项 | 数据保留持续时间
| -- | --
| 日期 | **默认保留期** ：相当于“每日备份的保留期(以天为单位)”。 <br/><br/> **例外** ：如果为长期保留（周、月、年）设置的每日计划备份失败，则会考虑将失败后立即触发的按需备份进行长期保留。 否则，将考虑进行下一次计划备份以进行长期保留。<br/><br/> **示例方案** ：在星期四上午 8:00 进行的计划备份失败。 考虑每周、每月或每年保留此备份。 因此，在下一次于星期五上午 8:00 执行计划备份之前触发的第一个按需备份将自动标记为每周、每月或每年保留。 此备份会替代星期四上午 8:00 的备份。
| Week | **默认保留期** ：一天。 为使用每周备份策略的数据源创建的按需备份将在第二天删除。 即使它们是该数据源的最新备份，也仍会将其删除。 <br/><br/> **例外** ：如果为长期保留（周、月、年）设置的每周计划备份失败，则会考虑将失败后立即触发的按需备份进行长期保留。 否则，将考虑进行下一次计划备份以进行长期保留。 <br/><br/> **示例方案** ：在星期四上午 8:00 进行的计划备份失败。 考虑每月或每年保留此备份。 因此，在下一次于星期四上午 8:00 执行计划备份之前触发的第一个按需备份将自动标记为每月或每年保留。 此备份会替代星期四上午 8:00 的备份。

有关详细信息，请参阅[创建备份策略](#create-a-backup-policy)。
## <a name="next-steps"></a>后续步骤

了解如何[在 Azure 中还原文件](backup-azure-restore-windows-server.md)。
<!-- Update_Description: update metedata properties -->