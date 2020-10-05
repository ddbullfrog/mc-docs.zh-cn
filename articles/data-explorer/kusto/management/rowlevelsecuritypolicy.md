---
title: 行级别安全性（预览版）- Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的行级别安全性（预览版）。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 03/25/2020
ms.date: 09/24/2020
ms.openlocfilehash: 2d54235d357bba209540fa5eff3420552aa3b39c
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146708"
---
# <a name="row-level-security-preview"></a>行级别安全性（预览版）

使用组成员资格或执行上下文来控制对数据库表中行的访问权限。

行级别安全性 (RLS) 简化了安全性的设计和编码。 它允许在应用程序中对数据行访问进行限制。 例如，限制用户，仅允许其访问与其部门相关的行；或限制客户，仅允许其访问与其公司相关的数据。

访问限制逻辑位于数据库层中，而不是在远离数据的另一个应用层中。 数据库系统会在用户每次尝试从任何层进行数据访问时应用访问限制。 此逻辑减少了安全系统的外围应用，使安全系统变得更加可靠和稳健。

使用 RLS，你可以向其他应用程序和用户仅提供对表的某个部分的访问权限。 例如，您可能希望：

* 仅授予对满足某些条件的行的访问权限
* 将某些列中的数据匿名化
* 以上都是

有关详细信息，请参阅[用于管理行级别安全性策略的控制命令](../management/row-level-security-policy.md)。

> [!TIP]
> 以下函数通常用于 row_level_security 查询：
> * [current_principal()](../query/current-principalfunction.md)
> * [current_principal_details()](../query/current-principal-detailsfunction.md)
> * [current_principal_is_member_of()](../query/current-principal-ismemberoffunction.md)

## <a name="limitations"></a>限制

对于可以配置行级别安全性策略的表的数量没有限制。

不能在符合以下条件的表上启用 RLS 策略：
* 配置了[连续数据导出](../management/data-export/continuous-data-export.md)。
* 被[更新策略](./updatepolicy.md)的查询所引用。
* 配置了[受限视图访问策略](./restrictedviewaccesspolicy.md)。

## <a name="examples"></a>示例

### <a name="limit-access-to-sales-table"></a>限制对 Sales 表的访问

在名为 `Sales` 的表中，每行都包含有关销售的详细信息。 其中一列包含销售人员的姓名。 在 `Sales` 表上启用行级别安全性策略，而不是授予销售人员对此表中所有记录的访问权限，以便仅返回销售人员是当前用户的记录：

```kusto
Sales | where SalesPersonAadUser == current_principal()
```

你还可以掩盖信用卡号码：

```kusto
Sales | where SalesPersonAadUser == current_principal() | extend CreditCardNumber = "****"
```

如果希望每个销售人员都能看到某个特定国家/地区的所有销售项，则可以定义类似于以下内容的查询：

```kusto
let UserToCountryMapping = datatable(User:string, Country:string)
[
  "john@domain.com", "USA",
  "anna@domain.com", "France"
];
Sales
| where Country in (UserToCountryMapping | where User == current_principal_details()["UserPrincipalName"] | project Country)
```

如果你有一个包含经理的组，则可能需要向他们授予对所有行的访问权限。 查询行级别安全策略。

```kusto
let IsManager = current_principal_is_member_of('aadgroup=sales_managers@domain.com');
let AllData = Sales | where IsManager;
let PartialData = Sales | where not(IsManager) and (SalesPersonAadUser == current_principal());
union AllData, PartialData
| extend CreditCardNumber = "****"
```

### <a name="expose-different-data-to-members-of-different-azure-ad-groups"></a>将不同数据公开给不同 Azure AD 组的成员

如果你有多个 Azure AD 组，并且希望每个组的成员可以查看不同的数据子集，请将此结构用于 RLS 查询。 假定一个用户只能属于单个 Azure AD 组。

```kusto
let IsInGroup1 = current_principal_is_member_of('aadgroup=group1@domain.com');
let IsInGroup2 = current_principal_is_member_of('aadgroup=group2@domain.com');
let IsInGroup3 = current_principal_is_member_of('aadgroup=group3@domain.com');
let DataForGroup1 = Customers | where IsInGroup1 and <filtering specific for group1>;
let DataForGroup2 = Customers | where IsInGroup2 and <filtering specific for group2>;
let DataForGroup3 = Customers | where IsInGroup3 and <filtering specific for group3>;
union DataForGroup1, DataForGroup2, DataForGroup3
```

### <a name="apply-the-same-rls-function-on-multiple-tables"></a>在多个表上应用同一 RLS 函数

首先，定义一个函数，使该函数接收表名作为字符串参数，并使用 `table()` 运算符引用该表。 

例如：

```kusto
.create-or-alter function RLSForCustomersTables(TableName: string) {
    table(TableName)
    | ...
}
```

然后通过以下方式在多个表上配置 RLS：


```kusto
.alter table Customers1 policy row_level_security enable "RLSForCustomersTables('Customers1')"
.alter table Customers2 policy row_level_security enable "RLSForCustomersTables('Customers2')"
.alter table Customers3 policy row_level_security enable "RLSForCustomersTables('Customers3')"
```

### <a name="produce-an-error-upon-unauthorized-access"></a>在进行未经授权的访问时生成错误

如果希望未经授权的表用户收到错误而不希望返回空表，请使用 [`assert()`](../query/assert-function.md) 函数。 以下示例演示如何在 RLS 函数中生成此错误：

```kusto
.create-or-alter function RLSForCustomersTables() {
    MyTable
    | where assert(current_principal_is_member_of('aadgroup=mygroup@mycompany.com') == true, "You don't have access")
}
```

可以将此方法与其他示例组合在一起。 例如，可以向不同 AAD 组中的用户显示不同的结果，并为所有其他人生成一个错误。

### <a name="control-permissions-on-follower-databases"></a>控制对后继数据库的权限

你在生产数据库上配置的 RLS 策略还将在随从数据库中生效。 不能在生产数据库和随从数据库上配置不同的 RLS 策略。 但是，你可以在 RLS 查询中使用 [`current_cluster_endpoint()`](../query/current-cluster-endpoint-function.md) 函数来达到与在后继表中使用不同 RLS 查询相同的效果。

例如：

```kusto
.create-or-alter function RLSForCustomersTables() {
    let IsProductionCluster = current_cluster_endpoint() == "mycluster.eastus.kusto.windows.net";
    let DataForProductionCluster = TempTable | where IsProductionCluster;
    let DataForFollowerClusters = TempTable | where not(IsProductionCluster) | extend CreditCardNumber = "****";
    union DataForProductionCluster, DataForFollowerClusters
}
```

## <a name="more-use-cases"></a>更多用例

* 呼叫中心支持人员通过身份证号或信用卡号的几个数字就可以辨识呼叫者。 不应向支持人员完全公开这些号码。 可以在表上应用 RLS 策略以掩盖任意查询的结果集中任何身份证号或信用卡号除最后四位数以外的其他所有数字。
* 设置对个人身份信息 (PII) 进行掩盖的 RLS 策略，使开发人员能够在不违反合规性法规的情况下，对生产环境进行查询，以便进行故障排除。
* 医院可以设置 RLS 策略，以允许护士仅查看自己患者的数据行。
* 银行可以设置 RLS 策略，以根据员工的业务部门或角色来限制对财务数据行的访问权限。
* 多租户应用程序可以将来自多个租户的数据存储在单个表集中（这非常高效）。 它们将使用 RLS 策略来强制对每个租户的数据行与所有其他租户的行进行逻辑分离，使每个租户只能看到其自己的数据行。

## <a name="performance-impact-on-queries"></a>对查询的性能影响

在表上启用 RLS 策略后，对访问该表的查询会产生一定的性能影响。 对表的访问实际上将被替换为在该表上定义的 RLS 查询。 RLS 查询的性能影响通常包括两部分：

* Azure Active Directory 中的成员身份检查
* 应用于数据的筛选器

例如：

```kusto
let IsRestrictedUser = current_principal_is_member_of('aadgroup=some_group@domain.com');
let AllData = MyTable | where not(IsRestrictedUser);
let PartialData = MyTable | where IsRestrictedUser and (...);
union AllData, PartialData
```

如果用户不属于 *some_group@domain.com* ，则 `IsRestrictedUser` 的评估结果将为 `false`。 要评估的查询类似于以下查询：

```kusto
let AllData = MyTable;           // the condition evaluates to `true`, so the filter is dropped
let PartialData = <empty table>; // the condition evaluates to `false`, so the whole expression is replaced with an empty table
union AllData, PartialData       // this will just return AllData, as PartialData is empty
```

同样，如果 `IsRestrictedUser` 的评估结果为 `true`，则只会评估对 `PartialData` 的查询。

### <a name="improve-query-performance-when-rls-is-used"></a>提高使用 RLS 时的查询性能

* 如果对高基数列（例如 DeviceID）应用了筛选器，请考虑使用[分区策略](./partitioningpolicy.md)或[行顺序策略](./roworderpolicy.md)
* 如果对中低基数列应用了筛选器，请考虑使用[行顺序策略](./roworderpolicy.md)

## <a name="performance-impact-on-ingestion"></a>对引入的性能影响

对引入没有任何性能影响。
