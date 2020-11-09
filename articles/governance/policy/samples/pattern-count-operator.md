---
title: 模式：策略定义中的 count 运算符
description: 此 Azure Policy 模式提供了有关如何在策略定义中使用 count 运算符的示例。
origin.date: 10/14/2020
ms.date: 11/06/2020
ms.author: v-tawe
ms.topic: sample
ms.openlocfilehash: a903c9f37b9082356e522723aa9b53a553f2f0f1
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327706"
---
# <a name="azure-policy-pattern-the-count-operator"></a>Azure Policy 模式：count 运算符

[count](../concepts/definition-structure.md#count) 运算符评估 \[\*\] 别名的成员。

## <a name="sample-policy-definition"></a>示例策略定义

此策略定义[审核](../concepts/effects.md#audit)配置为允许入站远程桌面协议 (RDP) 流量的网络安全组。

```json
{
   "properties": {
       "mode": "all",
       "displayName": "Audit Network Security Groups for RDP",
       "description": "This policy audits NSGs with RDP ports enabled",
       "policyRule": {
           "if": {
               "allOf": [{
                       "field": "type",
                       "equals": "Microsoft.Network/networkSecurityGroups"
                   },
                   {
                       "count": {
                           "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
                           "where": {
                               "allOf": [{
                                       "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].direction",
                                       "equals": "Inbound"
                                   },
                                   {
                                       "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].access",
                                       "equals": "Allow"
                                   },
                                   {
                                       "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRange",
                                       "equals": "3389"
                                   }
                               ]
                           }
                       },
                       "greater": 0
                   }
               ]
           },
           "then": {
               "effect": "audit"
           }
       }
   }
}
```

### <a name="explanation"></a>说明

**count** 运算符的核心组件是 _field_ 、 _where_ 和条件。 下面的代码片段中突出显示了每个组件。

- _field_ 告诉 count 要评估哪个 [别名](../concepts/definition-structure.md#aliases)的成员。 下面，让我们看一下网络安全组的 **securityRules\[\*\]** 别名数组  。
- _where_ 使用策略语言来定义哪些数组  成员满足条件。 在此示例中， **allOf** 逻辑运算符将别名数组  属性的以下三个不同条件评估分组到一起： _direction_ 、 _access_ 和 _destinationPortRange_ 。
- 此示例中的 count 条件为 **greater** 。 当别名数组  的一个或多个成员与 _where_ 子句匹配时，Count 的评估结果为 true。

```json
{
   "count": {
       "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
       "where": {
           "allOf": [{
                   "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].direction",
                   "equals": "Inbound"
               },
               {
                   "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].access",
                   "equals": "Allow"
               },
               {
                   "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRange",
                   "equals": "3389"
               }
           ]
       }
   },
   "greater": 0
}
```

## <a name="next-steps"></a>后续步骤

- 查看其他[模式和内置定义](./index.md)。
- 查看 [Azure Policy 定义结构](../concepts/definition-structure.md)。
- 查看[了解策略效果](../concepts/effects.md)。