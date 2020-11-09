---
title: 删除 Azure 网络观察程序中网络安全组流日志的存储 Blob | Azure
description: 本文介绍了如何在 Azure 网络观察程序中删除超过了其保留策略期限的网络安全组流日志存储 blob。
services: network-watcher
documentationcenter: na
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 08/16/2019
author: rockboyfor
ms.date: 11/09/2020
ms.testscope: yes
ms.testdate: 08/03/2020
ms.author: v-yeche
ms.openlocfilehash: a450aca7518b5b2965423f1cdcb968a1ba6f9ffe
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328099"
---
# <a name="delete-network-security-group-flow-log-storage-blobs-in-network-watcher"></a>在网络观察程序中删除网络安全组流日志存储 blob

如果需要从存储帐户中手动删除流日志，可以使用下面的 PowerShell 脚本。
此脚本仅删除比用户指定的现有保留策略旧的存储 blob。

## <a name="run-powershell-script-to-delete-nsg-flow-logs"></a>运行 PowerShell 脚本来删除 NSG 流日志

将以下脚本复制并保存到某个位置，例如当前工作目录。 

```powershell
# This powershell script deletes all NSG flow log blobs that should not be retained anymore as per configured retention policy.
# While configuring NSG flow logs on Azure portal, the user configures the retention period of NSG flow log blobs in
# their storage account (in days).
# This script reads all blobs and deletes blobs that are not to be retained (outside retention window)
# if the retention days are zero; all blobs are retained forever and hence no blobs are deleted.
#
#

param (
        [string] [Parameter(Mandatory=$true)]  $SubscriptionId,
        [string] [Parameter(Mandatory=$true)]  $Location,
        [switch] [Parameter(Mandatory=$false)] $Confirm
    )

Connect-AzAccount -Environment AzureChinaCloud

$SubId = Get-AzSubscription| Where-Object {$_.Id.contains($SubscriptionId.ToLower())}

if ($SubId.Count -eq 0)
{
    Write-Error 'The SubscriptionId does not exist' -ErrorAction Stop
}

Set-AzContext -SubscriptionId $SubscriptionId

$NsgList = Get-AzNetworkSecurityGroup | Where-Object {$_.Location -eq $Location}
$NW = Get-AzNetworkWatcher | Where-Object {$_.Location -eq $Location}

$FlowLogsList = @()
foreach ($Nsg in $NsgList)
{
    # Query Flow Log Status which are enabled
    $NsgFlowLog = Get-AzNetworkWatcherFlowLogStatus -NetworkWatcher $NW -TargetResourceId $Nsg.Id | Where-Object {$_.Enabled -eq "True"}
    if ($NsgFlowLog.Count -gt 0)
    {
        $FlowLogsList +=  $NsgFlowLog
        Write-Output ('Enabled NSG found: ' +  $NsgFlowLog.TargetResourceId)
    }
}

foreach ($Psflowlog in $FlowLogsList)
{
    $RetentionDays = $Psflowlog.RetentionPolicy.Days
    if ($RetentionDays -le 0)
    {
        continue
    }

    $Strings = $Psflowlog.StorageId -split '/'
    $RGName = $Strings[4]
    $StorageAccountName = $Strings[-1]

    $Key = (Get-AzStorageAccountKey -ResourceGroupName $RGName -Name $StorageAccountName).Value[1]
    $StorageAccount = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $Key

    $ContainerName = 'insights-logs-networksecuritygroupflowevent'  
    $BLobsList = Get-AzStorageBlob -Container $ContainerName -Context $StorageAccount.Context

    $TargetBLobsList = $BLobsList | Where-Object {$_.Name.contains($Psflowlog.TargetResourceId.ToUpper())}

    $RetentionDate = Get-Date
    $RetentionDate = $RetentionDate.AddDays(-1*$RetentionDays)
    $RetentionDateInUTC = $RetentionDate.ToUniversalTime()

    foreach ($Blob in $TargetBLobsList)
    {
        $BlobLastModifietedDTinUTC = [datetime]$Blob.LastModified.UtcDateTime

        if ($BlobLastModifietedDTinUTC -ge  $RetentionDateInUTC)
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> RETAINED')
            continue
        }

        if ($Confirm)
        {
            Write-Output (Blob to be deleted: $Blob.Name)
            $Confirmation = Read-Host "Are you sure you want to remove this blob (Y/N)?"
        }

        if ((-not $Confirm) -or ($Confirmation -eq 'Y'))
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> DELETED')
            Remove-AzStorageBlob -Container $ContainerName -Context $StorageAccount.Context -Blob $Blob.Name
        }
        else
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> RETAINED')
        }
    }
}

Write-Output ('Retention policy for all NSGs evaluated and completed successfully')
```

1. 根据需要在脚本中输入以下参数：
    - **SubscriptionId** [必需]：要从中删除 NSG 流日志 blob 的订阅 ID。
    - **Location** [必需]：要为其删除 NSG 流日志 blob 的 NSG 的区域 _位置字符串_ 。 可以在 Azure 门户上或 [GitHub](https://github.com/Azure/azure-extensions-cli/blob/beb3d3fe984cfa9c7798cb11a274c5337968cbc5/regions.go#L23) 上查看此信息。
    - **Confirm** [可选]：如果要手动确认每个存储 blob 的删除，请传递 confirm 标志。

1. 运行保存的脚本，如以下示例中所示，其中脚本文件保存为 **Delete-NsgFlowLogsBlobs.ps1** ：
    ```
    .\Delete-NsgFlowLogsBlobs.ps1 -SubscriptionId <subscriptionId> -Location  <location> -Confirm
    ```
    
## <a name="next-steps"></a>后续步骤
- 客户可以使用 [Azure 逻辑应用](../logic-apps/logic-apps-overview.md)或 [Azure 自动化](https://www.azure.cn/home/features/automation/)自动运行脚本

<!--Not Available on  [Azure Monitor logs for network security groups (NSGs)](../virtual-network/virtual-network-nsg-manage-log.md?toc=%2fnetwork-watcher%2ftoc.json)-->

<!-- Update_Description: update meta properties, wording update, update link -->