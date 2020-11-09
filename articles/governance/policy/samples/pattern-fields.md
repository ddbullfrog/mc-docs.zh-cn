---
title: 模式：策略定义中的字段属性
description: 此 Azure Policy 模式通过示例介绍了如何在策略定义中使用字段属性。
origin.date: 10/14/2020
ms.date: 11/06/2020
ms.author: v-tawe
ms.topic: sample
ms.openlocfilehash: 802a794efa0751c4a57e540db450915b10e57488
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327817"
---
# <a name="azure-policy-pattern-field-properties"></a>Azure Policy 模式：字段属性

[field](../concepts/definition-structure.md#fields) 运算符会对指定属性或[别名](../concepts/definition-structure.md#aliases)进行评估，针对为给定[条件](../concepts/definition-structure.md#conditions)提供的值。

## <a name="sample-policy-definition"></a>示例策略定义

此策略定义使你能够定义满足组织的地理位置要求的允许区域。 允许的资源在参数 **listOfAllowedLocations** （数组）  中定义。 与定义匹配的资源会被[拒绝](../concepts/effects.md#deny)。

```json
{
   "properties": {
       "displayName": "Allowed locations",
       "policyType": "BuiltIn",
       "description": "This policy enables you to restrict the locations your organization can specify when deploying resources. Use to enforce your geo-compliance requirements. Excludes resource groups, Microsoft.AzureActiveDirectory/b2cDirectories, and resources that use the 'global' region.",
       "mode": "Indexed",
       "parameters": {
           "listOfAllowedLocations": {
               "type": "Array",
               "metadata": {
                   "description": "The list of locations that can be specified when deploying resources.",
                   "strongType": "location",
                   "displayName": "Allowed locations"
               }
           }
       },
       "policyRule": {
           "if": {
               "allOf": [{
                       "field": "location",
                       "notIn": "[parameters('listOfAllowedLocations')]"
                   },
                   {
                       "field": "location",
                       "notEquals": "global"
                   },
                   {
                       "field": "type",
                       "notEquals": "Microsoft.AzureActiveDirectory/b2cDirectories"
                   }
               ]
           },
           "then": {
               "effect": "Deny"
           }
       }
   }
}
```

### <a name="explanation"></a>说明

```json
   "if": {
       "allOf": [{
               "field": "location",
               "notIn": "[parameters('listOfAllowedLocations')]"
           },
           {
               "field": "location",
               "notEquals": "global"
           },
           {
               "field": "type",
               "notEquals": "Microsoft.AzureActiveDirectory/b2cDirectories"
           }
       ]
   },
   "then": {
       "effect": "Deny"
   }
}
```

**field** 运算符在 [逻辑运算符](../concepts/definition-structure.md#logical-operators) **allOf** 中使用三次。

- 第一次使用时，会通过 **listOfAllowedLocations** 参数的 **notIn** 条件评估 `location` 属性。 **notIn** 适用是因为它预期的是数组  ，而参数为数组  。 如果创建的或更新的资源的 `location` 不在批准项列表中，则此元素的评估结果为 true。
- 第二次使用也评估 `location` 属性，但使用 **notEquals** 条件来查看资源是否为全局  资源。 如果创建的或更新的资源的 `location` 不是全局的，则此元素的评估结果为 true。 
- 最后一次使用评估 `type` 属性，并使用 **notEquals** 条件来验证资源类型是否为 _Microsoft.AzureActiveDirectory/b2cDirectories_ 。 如果为否，则此元素的评估结果为 true。

如果 **allOf** 逻辑运算符中的所有三个条件语句均为 true，则 Azure Policy 会阻止资源的创建或更新。

## <a name="next-steps"></a>后续步骤

- 查看其他[模式和内置定义](./index.md)。
- 查看 [Azure Policy 定义结构](../concepts/definition-structure.md)。
- 查看[了解策略效果](../concepts/effects.md)。