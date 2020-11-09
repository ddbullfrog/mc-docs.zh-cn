---
title: 模式：策略定义的效果
description: 此 Azure Policy 模式通过示例介绍了如何使用策略定义的不同效果。
origin.date: 10/14/2020
ms.date: 11/06/2020
ms.author: v-tawe
ms.topic: sample
ms.openlocfilehash: d39c6fbc4faf92b0d8f3d83bcb8668a7ca9bee7b
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327822"
---
# <a name="azure-policy-pattern-effects"></a>Azure Policy 模式：效果

Azure Policy 有很多[效果](../concepts/effects.md)，这些效果决定了服务对不合规资源的反应。 某些效果很简单，并且在策略定义中无需其他属性，而另一些效果则需要多个属性。

## <a name="sample-1-simple-effect"></a>示例 1：简单效果

此策略定义检查评估的资源上是否存在参数 **tagName** 中定义的标记。 如果标记尚不存在，则会触发 [modify](../concepts/effects.md#modify) 效果，添加一个使用参数 **tagValue** 中的值的标记。

```json
{
   "properties": {
       "displayName": "Add a tag to resource groups",
       "policyType": "BuiltIn",
       "mode": "All",
       "description": "Adds the specified tag and value when any resource group missing this tag is created or updated. Existing resource groups can be remediated by triggering a remediation task. If the tag exists with a different value it will not be changed.",
       "metadata": {
           "version": "1.0.0",
           "category": "Tags"
       },
       "parameters": {
           "tagName": {
               "type": "String",
               "metadata": {
                   "displayName": "Tag Name",
                   "description": "Name of the tag, such as 'environment'"
               }
           },
           "tagValue": {
               "type": "String",
               "metadata": {
                   "displayName": "Tag Value",
                   "description": "Value of the tag, such as 'production'"
               }
           }
       },
       "policyRule": {
           "if": {
               "allOf": [{
                       "field": "type",
                       "equals": "Microsoft.Resources/subscriptions/resourceGroups"
                   },
                   {
                       "field": "[concat('tags[', parameters('tagName'), ']')]",
                       "exists": "false"
                   }
               ]
           },
           "then": {
               "effect": "modify",
               "details": {
                   "roleDefinitionIds": [
                       "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
                   ],
                   "operations": [{
                       "operation": "add",
                       "field": "[concat('tags[', parameters('tagName'), ']')]",
                       "value": "[parameters('tagValue')]"
                   }]
               }
           }
       }
   }
}
```

### <a name="sample-1-explanation"></a>示例 1：说明

```json
"then": {
"effect": "modify",
"details": {
   "roleDefinitionIds": [
       "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
   ],
   "operations": [{
       "operation": "add",
       "field": "[concat('tags[', parameters('tagName'), ']')]",
       "value": "[parameters('tagValue')]"
   }]
}
```

modify  效果需要 policyRule.then.details  块，该块定义 roleDefinitionIds  和 operations  。 这些参数会告知 Azure Policy 需要哪些角色才能添加标记和修正资源，以及要使用哪些“修改”操作。 在此示例中，“add”和“operation”等参数用于设置标记和值。

## <a name="sample-2-complex-effect"></a>示例 2：复杂效果

此策略定义会审核每个虚拟机中是否存在参数 **publisher** 和 **type** 中定义的扩展不存在的情况。 它使用 [auditIfNotExists](../concepts/effects.md#auditifnotexists) 检查与虚拟机相关的资源，看是否存在与定义的参数匹配的实例。 此示例检查“扩展”  类型。

```json
{
   "type": "Microsoft.Authorization/policyDefinitions",
   "name": "audit-vm-extension",
   "properties": {
       "displayName": "Audit if extension does not exist",
       "description": "This policy audits if a required extension doesn't exist.",
       "parameters": {
           "publisher": {
               "type": "String",
               "metadata": {
                   "description": "The publisher of the extension",
                   "displayName": "Extension Publisher"
               }
           },
           "type": {
               "type": "String",
               "metadata": {
                   "description": "The type of the extension",
                   "displayName": "Extension Type"
               }
           }
       },
       "policyRule": {
           "if": {
               "allOf": [{
                       "field": "type",
                       "equals": "Microsoft.Compute/virtualMachines"
                   },
                   {
                       "field": "Microsoft.Compute/imagePublisher",
                       "in": [
                           "MicrosoftWindowsServer"
                       ]
                   },
                   {
                       "field": "Microsoft.Compute/imageOffer",
                       "in": [
                           "WindowsServer"
                       ]
                   }
               ]
           },
           "then": {
               "effect": "auditIfNotExists",
               "details": {
                   "type": "Microsoft.Compute/virtualMachines/extensions",
                   "existenceCondition": {
                       "allOf": [{
                               "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
                               "equals": "[parameters('publisher')]"
                           },
                           {
                               "field": "Microsoft.Compute/virtualMachines/extensions/type",
                               "equals": "[parameters('type')]"
                           }
                       ]
                   }
               }
           }
       }
   }
}
```

### <a name="sample-2-explanation"></a>示例 2：说明

```json
"details": {
   "type": "Microsoft.Compute/virtualMachines/extensions",
   "existenceCondition": {
       "allOf": [{
               "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
               "equals": "[parameters('publisher')]"
           },
           {
               "field": "Microsoft.Compute/virtualMachines/extensions/type",
               "equals": "[parameters('type')]"
           }
       ]
   }
}
```

**auditIfNotExists** 效果需要 **policyRule.then.details** 块来定义要查找的 **type** 和 **existenceCondition** 。 **existenceCondition** 使用策略语言元素（如 [逻辑运算符](../concepts/definition-structure.md#logical-operators)）来确定是否存在匹配的相关资源。 在此示例中，对照每个[别名](../concepts/definition-structure.md#aliases)检查的值在参数中定义。

## <a name="next-steps"></a>后续步骤

- 查看其他[模式和内置定义](./index.md)。
- 查看 [Azure Policy 定义结构](../concepts/definition-structure.md)。
- 查看[了解策略效果](../concepts/effects.md)。