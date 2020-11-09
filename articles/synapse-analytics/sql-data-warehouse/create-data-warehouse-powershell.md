---
title: 快速入门：使用 Azure PowerShell 创建 Synapse SQL 池
description: 通过 Azure PowerShell 使用服务器级防火墙规则快速创建 Synapse SQL 池。
services: synapse-analytics
author: WenJason
manager: digimobile
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
origin.date: 4/11/2019
ms.date: 11/09/2020
ms.author: v-jay
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse    , devx-track-azurepowershell
ms.openlocfilehash: c3695e9dab87bf3c7f916d59377d93040ea119d1
ms.sourcegitcommit: b217474b15512b0f40b2eaae66bd3c521383d321
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/05/2020
ms.locfileid: "93375725"
---
# <a name="quickstart-create-a-synapse-sql-pool-with-azure-powershell"></a>快速入门：使用 Azure PowerShell 创建 Synapse SQL 池

使用 Azure PowerShell 在 Azure Synapse Analytics 中创建 Synapse SQL 池（数据仓库）。

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://www.azure.cn/zh-cn/pricing/1rmb-trial-full/?form-type=identityauth)。

> [!IMPORTANT]
> 创建 SQL 池可能会产生新的计费服务。  有关详细信息，请参阅 [Azure Synapse Analytics 定价](https://azure.cn/pricing/details/sql-data-warehouse/)。

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="sign-in-to-azure"></a>登录 Azure

使用 [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

```powershell
Connect-AzAccount -EnvironmentName AzureChinaCloud
```

若要查看正在使用的订阅，请运行 [Get-AzSubscription](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。

```powershell
Get-AzSubscription
```

如果需要使用与默认订阅不同的订阅，请运行 [Set-AzContext](https://docs.microsoft.com/powershell/module/az.accounts/set-azcontext?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。

```powershell
Set-AzContext -SubscriptionName "MySubscription"
```

## <a name="create-variables"></a>创建变量

定义在本快速入门的脚本中使用的变量。

```powershell
# The data center and resource name for your resources
$resourcegroupname = "myResourceGroup"
$location = "ChinaEast"
# The server name: Use a random value or replace with your own value (don't capitalize)
$servername = "server-$(Get-Random)"
# Set an admin name and password for your database
# The sign-in information for the server
$adminlogin = "ServerAdmin"
$password = "ChangeYourAdminPassword1"
# The ip address range that you want to allow to access your server - change as appropriate
$startip = "0.0.0.0"
$endip = "0.0.0.0"
# The database name
$databasename = "mySampleDataWarehouse"
```

## <a name="create-a-resource-group"></a>创建资源组

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 命令创建 [Azure 资源组](../../azure-resource-manager/management/overview.md?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 资源组是在其中以组的形式部署和管理 Azure 资源的容器。 以下示例在 `chinaeast` 位置创建名为 `myResourceGroup` 的资源组。

```powershell
New-AzResourceGroup -Name $resourcegroupname -Location $location
```

## <a name="create-a-server"></a>创建服务器

使用 [New-AzSqlServer](https://docs.microsoft.com/powershell/module/az.sql/new-azsqlserver?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 命令创建[逻辑 SQL 服务器](../../azure-sql/database/logical-servers.md?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 服务器包含作为组进行管理的一组数据库。 以下示例使用管理员用户名 `ServerAdmin` 和密码 `ChangeYourAdminPassword1` 在资源组中创建随机命名的服务器。 根据需要替换这些预定义的值。

```powershell
New-AzSqlServer -ResourceGroupName $resourcegroupname `
    -ServerName $servername `
    -Location $location `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
```

## <a name="configure-a-server-level-firewall-rule"></a>配置服务器级防火墙规则

使用 [New-AzSqlServerFirewallRule](https://docs.microsoft.com/powershell/module/az.sql/new-azsqlserverfirewallrule?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) 命令创建[服务器级防火墙规则](../../azure-sql/database/firewall-configure.md?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 服务器级防火墙规则允许外部应用程序（例如 SQL Server Management Studio 或 SQLCMD 实用程序）通过 SQL 池服务防火墙连接到 SQL 池。

在以下示例中，防火墙仅对其他 Azure 资源开放。 要启用外部连接，请将 IP 地址更改为适合你环境的地址。 若要开放所有 IP 地址，请使用 0.0.0.0 作为起始 IP 地址，使用 255.255.255.255 作为结束地址。

```powershell
New-AzSqlServerFirewallRule -ResourceGroupName $resourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowSome" -StartIpAddress $startip -EndIpAddress $endip
```

> [!NOTE]
> SQL 终结点通过端口 1433 进行通信。 如果尝试从企业网络内部进行连接，则该网络的防火墙可能不允许经端口 1433 的出站流量。 如果是这样，则无法连接到服务器，除非 IT 部门打开了端口 1433。
>

## <a name="create-a-sql-pool"></a>创建 SQL 池

以下示例使用以前定义的变量创建 SQL 池。  它将服务目标指定为 DW100c，这是针对 SQL 池的低成本起点。

```Powershell
New-AzSqlDatabase `
    -ResourceGroupName $resourcegroupname `
    -ServerName $servername `
    -DatabaseName $databasename `
    -Edition "DataWarehouse" `
    -RequestedServiceObjectiveName "DW100c" `
    -CollationName "SQL_Latin1_General_CP1_CI_AS" `
    -MaxSizeBytes 10995116277760
```

所需的参数有：

* **RequestedServiceObjectiveName** ：请求的 [数据仓库单位](what-is-a-data-warehouse-unit-dwu-cdwu.md)的数量。 增加此数量会增加计算成本。 有关支持值的列表，请参阅[内存和并发限制](memory-concurrency-limits.md)。
* **DatabaseName** ：要创建的 SQL 池的名称。
* **ServerName** ：用于创建过程的服务器名称。
* **ResourceGroupName** ：要使用的资源组。 若要查找订阅中可用的资源，请使用 Get-AzureResource。
* **Edition** ：必须是“DataWarehouse”才能创建 SQL 池。

可选参数有：

* **CollationName** ：在不指定的情况下，默认排序规则是 SQL_Latin1_General_CP1_CI_AS。 在数据库上不能更改排序规则。
* **MaxSizeBytes** ：数据库的默认最大大小为 240TB。 最大大小限制行存储数据。 列数据的存储不受限制。

有关参数选项的详细信息，请参阅 [New-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/new-azsqldatabase?toc=/synapse-analytics/sql-data-warehouse/toc.json&bc=/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。

## <a name="clean-up-resources"></a>清理资源

本系列中的其他快速入门教程是在本快速入门的基础上制作的。

> [!TIP]
> 如何打算继续学习后续快速入门教程，请不要清除本快速入门中创建的资源。 如果不打算继续，请在 Azure 门户中执行以下步骤，删除本快速入门创建的所有资源。
>

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="next-steps"></a>后续步骤

现已创建 SQL 池，创建防火墙规则并连接到 SQL 池。 若要了解详细信息，请继续阅读[将数据加载到 SQL 池](load-data-from-azure-blob-storage-using-polybase.md)一文。
