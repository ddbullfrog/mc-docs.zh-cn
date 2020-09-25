---
title: 确定导致非符合性的原因
description: 如果资源不符合，可能有很多原因。 找出导致非符合性的原因。
ms.author: v-tawe
origin.date: 07/06/2020
ms.date: 09/15/2020
ms.topic: how-to
ms.openlocfilehash: cc3de4611400e4fb57f50e384c9efeb43832bbfc
ms.sourcegitcommit: f5d53d42d58c76bb41da4ea1ff71e204e92ab1a7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90523728"
---
# <a name="determine-causes-of-non-compliance"></a>确定导致非符合性的原因

当 Azure 资源被确定为不符合策略规则时，了解该资源的哪一部分不符合规则很有用。 这也有助于了解哪些更改内容更改了以前符合的资源，使其变得不符合。 可通过两种方法查找此信息：

- [符合性详细系信息](#compliance-details)

<!-- no change history in mc portal -->
<!-- > - [Change history (Preview)](#change-history) -->

## <a name="compliance-details"></a>合规性详细信息

当资源不符合时，“策略符合性”页中将提供该资源的符合性详细信息。 符合性详细信息窗格包含以下信息：

- 名称、类型、位置和资源 ID 等资源详细信息
- 当前策略分配的上一个计算的符合性状态和时间戳
- 资源不符合性的原因列表

> [!IMPORTANT]
> 由于不符合资源的符合性详细信息显示该资源属性的当前值，因此用户必须对资源类型进行读取操作。 例如，如果不符合资源为 Microsoft.Compute/virtualMachines，则用户必须进行 Microsoft.Compute/virtualMachines/read 操作。 如果用户没有进行所需操作，则会显示访问错误。

若要查看符合性详细信息，请执行以下步骤：

1. 在 Azure 门户中选择“所有服务”，然后搜索并选择“策略”，启动 Azure Policy 服务。 

1. 在“概览”或“符合性”页，选择“符合性状态”为“不符合”的策略  。

1. 在“策略符合性”页的“资源符合性”选项卡下，右键单击或选择“符合性状态”为“不符合”的资源的省略号。 然后选择“查看符合性详细信息”。

   :::image type="content" source="../media/determine-non-compliance/view-compliance-details.png" alt-text="“资源符合性”选项卡上的“查看符合性详细信息”链接的屏幕截图。" border="false":::

1. “符合性详细信息”窗格显示对当前策略分配最近进行的计算得出的信息。 在此示例中，发现字段“Microsoft.Sql/servers/version”为“12.0”，而策略定义预期为“14.0”。 如果资源不符合有多种原因，则此窗格将列出每个原因。

   :::image type="content" source="../media/determine-non-compliance/compliance-details-pane.png" alt-text="符合性详细信息”窗格的屏幕截图，其中显示不符合的原因：当前值为 12，目标值为 14。" border="false":::

   对于“auditIfNotExists”或“deployIfNotExists”策略定义，详细信息包括“details.type”属性和任何可选属性。 有关列表，请参阅 [auditIfNotExists 属性](../concepts/effects.md#auditifnotexists-properties)和 [deployIfNotExists 属性](../concepts/effects.md#deployifnotexists-properties)。 “上一个计算资源”为定义的“详细信息”部分中的相关资源。

   部分“deployIfNotExists”定义示例：

   ```json
   {
       "if": {
           "field": "type",
           "equals": "[parameters('resourceType')]"
       },
       "then": {
           "effect": "DeployIfNotExists",
           "details": {
               "type": "Microsoft.Insights/metricAlerts",
               "existenceCondition": {
                   "field": "name",
                   "equals": "[concat(parameters('alertNamePrefix'), '-', resourcegroup().name, '-', field('name'))]"
               },
               "existenceScope": "subscription",
               "deployment": {
                   ...
               }
           }
       }
   }
   ```

   :::image type="content" source="../media/determine-non-compliance/compliance-details-pane-existence.png" alt-text="ifNotExists 的“符合性详细信息”窗格的屏幕截图，其中包括计算的资源计数。" border="false":::

> [!NOTE]
> 为保护数据，当属性值为“secret”时，当前值显示星号。

这些详细信息将解释资源当前不合规的原因，但不显示何时对该资源做出了更改，导致它不合规。

### <a name="compliance-reasons"></a>合规性原因

以下矩阵将每个可能原因映射到策略定义中的负责[条件](../concepts/definition-structure.md#conditions)：

|Reason | 条件 |
|-|-|
|当前值必须包含目标值作为关键值。 |containsKey 或不为 notContainsKey |
|当前值必须包含目标值。 |contains 或不为 notContains |
|当前值必须等于目标值。 |equals 或不为 notEquals |
|当前值必须小于目标值。 |less 或不为 greaterOrEquals |
|当前值必须大于或等于目标值。 |greaterOrEquals 或不为 less |
|当前值必须大于目标值。 |greater 或不为 lessOrEquals |
|当前值必须小于或等于目标值。 |lessOrEquals 或不为 greater |
|当前值必须存在。 |exists |
|当前值必须在目标值中。 |in 或不为 notIn |
|当前值必须与目标值类似。 |like 或不为 notLike |
|当前值必须与目标值匹配（区分大小写）。 |match 或不为 notMatch |
|当前值必须与目标值匹配（不区分大小写）。 |matchInsensitively 或不为 notMatchInsensitively |
|当前值不得包含目标值作为关键值。 |notContainsKey 或不为 containsKey|
|当前值不得包含目标值。 |notContains 或不为 contains |
|当前值不得等于目标值。 |notEquals 或不为 equals |
|不能存在当前值。 |不能为 exists  |
|当前值不得存在于目标值中。 |notIn 或不为 in |
|当前值不得与目标值类似。 |notLike 或不为 like |
|当前值不得与目标值匹配（区分大小写）。 |notMatch 或不为 match |
|当前值不得与目标值匹配（不区分大小写）。 |notMatchInsensitively 或不为 matchInsensitively |
|没有与策略定义中的效果详细信息匹配的相关资源。 |类型在“then.details.type”中定义，且与策略规则“if”部分定义的资源相关的资源不存在。 |

## <a name="compliance-details-for-guest-configuration"></a>来宾配置的符合性详细信息

对于"来宾配置”类别中的“auditIfNotExists”策略，VM 内可能有多个计算设置，需要查看各个设置的详细信息。 例如，如果你正在审核一个密码策略列表，其中只有一个密码策略的状态为“不符合”，这时你需要了解具体哪些密码策略不符合以及不符合的原因。

你也可能无权直接登录到 VM，但需要报告 VM 不符合的原因。

### <a name="azure-portal"></a>Azure 门户

首先遵循上述部分中的相同步骤查看策略符合性详细信息。

在“符合性详细信息”窗格视图中，选择“上次计算的资源”链接。

:::image type="content" source="../media/determine-non-compliance/guestconfig-auditifnotexists-compliance.png" alt-text="查看 auditIfNotExists 定义符合性详细信息的屏幕截图。" border="false":::

“来宾分配”页显示所有可用的符合性详细信息。 视图中的每一行都代表在计算机中执行的计算。 “原因”列中显示描述来宾分配“不符合”原因的短语。 例如，如果要审核密码策略，“原因”列将显示包含每个设置当前值的文本。

:::image type="content" source="../media/determine-non-compliance/guestconfig-compliance-details.png" alt-text="来宾分配符合性详细信息的屏幕截图。" border="false":::

### <a name="azure-powershell"></a>Azure PowerShell

还可以在 Azure PowerShell 上查看符合性详细信息。 首先，请确保已安装来宾配置模块。

```powershell
Install-Module Az.GuestConfiguration
```

可以使用以下命令查看 VM 所有来宾分配的当前状态：

```powershell
Get-AzVMGuestPolicyStatus -ResourceGroupName <resourcegroupname> -VMName <vmname>
```

```output
PolicyDisplayName                                                         ComplianceReasons
-----------------                                                         -----------------
Audit that an application is installed inside Windows VMs                 {[InstalledApplication]bwhitelistedapp}
Audit that an application is not installed inside Windows VMs.            {[InstalledApplication]NotInstalledApplica...
```

若要仅查看描述 VM 不符合原因的原因短语，只需返回原因子属性。

```powershell
Get-AzVMGuestPolicyStatus -ResourceGroupName <resourcegroupname> -VMName <vmname> | % ComplianceReasons | % Reasons | % Reason
```

```output
The following applications are not installed: '<name>'.
```

还可以输出计算机范围内来宾分配的符合性历史记录。 此命令的输出包含 VM 的每个报告的详细信息。

> [!NOTE]
> 输出可能会返回大量数据。 建议将输出存储在变量中。

```powershell
$guestHistory = Get-AzVMGuestPolicyStatusHistory -ResourceGroupName <resourcegroupname> -VMName <vmname>
$guestHistory
```

```output
PolicyDisplayName                                                         ComplianceStatus ComplianceReasons StartTime              EndTime                VMName LatestRepor
                                                                                                                                                                  tId
-----------------                                                         ---------------- ----------------- ---------              -------                ------ -----------
[Preview]: Audit that an application is installed inside Windows VMs      NonCompliant                       02/10/2019 12:00:38 PM 02/10/2019 12:00:41 PM VM01  ../17fg0...
<truncated>
```

若要简化视图，请使用“ShowChanged”参数。 此命令的输出仅包括报告，后接合规性状态的变化。

```powershell
$guestHistory = Get-AzVMGuestPolicyStatusHistory -ResourceGroupName <resourcegroupname> -VMName <vmname> -ShowChanged
$guestHistory
```

```output
PolicyDisplayName                                                         ComplianceStatus ComplianceReasons StartTime              EndTime                VMName LatestRepor
                                                                                                                                                                  tId
-----------------                                                         ---------------- ----------------- ---------              -------                ------ -----------
Audit that an application is installed inside Windows VMs                 NonCompliant                       02/10/2019 10:00:38 PM 02/10/2019 10:00:41 PM VM01  ../12ab0...
Audit that an application is installed inside Windows VMs.                Compliant                          02/09/2019 11:00:38 AM 02/09/2019 11:00:39 AM VM01  ../e3665...
Audit that an application is installed inside Windows VMs                 NonCompliant                       02/09/2019 09:00:20 AM 02/09/2019 09:00:23 AM VM01  ../15ze1...
```

<!-- no change history tag -->
<!-- ## <a name="change-history"/>Change history (Preview) -->
## <a name="next-steps"></a>后续步骤

- 在 [Azure Policy 示例](../samples/index.md)中查看示例。
- 查看 [Azure Policy 定义结构](../concepts/definition-structure.md)。
- 查看[了解策略效果](../concepts/effects.md)。
- 了解如何[以编程方式创建策略](programmatically-create.md)。
- 了解如何[获取符合性数据](get-compliance-data.md)。
- 了解如何[修正不符合的资源](remediate-resources.md)。
- 参阅[使用 Azure 管理组来组织资源](../../management-groups/overview.md)，了解什么是管理组。
