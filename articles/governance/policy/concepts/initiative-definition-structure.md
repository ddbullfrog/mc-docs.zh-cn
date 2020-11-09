---
title: 计划定义结构的详细信息
description: 描述如何使用策略计划定义对策略定义进行分组，以便部署到组织中的 Azure 资源。
origin.date: 10/07/2020
ms.date: 11/06/2020
ms.author: v-tawe
ms.topic: conceptual
ms.openlocfilehash: 129a1b5680e2bb95553a1192082103e2ed52b05b
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327743"
---
# <a name="azure-policy-initiative-definition-structure"></a>Azure Policy 计划定义结构

使用计划可组合多个相关策略定义，以简化分配和管理，因为可将组作为单个项使用。 例如，可以将相关标记策略组合为单个计划。 将应用计划，而非单独分配每个策略。

使用 JSON 创建策略计划定义。 策略计划定义包含以下各项的元素：

- 显示名称
- description
- metadata
- parameters
- 策略定义

<!-- - policy groups (this property is part of the [Regulatory Compliance (Preview) feature](./regulatory-compliance.md)) -->

下面的示例演示如何创建用于处理 `costCenter` 和 `productName` 这两个标记的计划。 它使用两个内置策略来应用默认标记值。

```json
{
    "properties": {
        "displayName": "Billing Tags Policy",
        "policyType": "Custom",
        "description": "Specify cost Center tag and product name tag",
        "metadata": {
            "version": "1.0.0",
            "category": "Tags"
        },
        "parameters": {
            "costCenterValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for Cost Center tag"
                },
                "defaultValue": "DefaultCostCenter"
            },
            "productNameValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for product Name tag"
                },
                "defaultValue": "DefaultProduct"
            }
        },
        "policyDefinitions": [{
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            }
        ]
    }
}
```

Azure Policy 内置项和模式位于 [Azure Policy 示例](../samples/index.md)。

## <a name="metadata"></a>Metadata

可选 `metadata` 属性存储关于策略计划定义的信息。
客户可在 `metadata` 中定义对其组织有用的任何属性和值。 但是，Azure Policy 和内置项使用一些常见属性。

### <a name="common-metadata-properties"></a>常见元数据属性

- `version`（字符串）：跟踪有关策略计划定义的内容版本的详细信息。
- `category`（字符串）：确定在 Azure 门户中的哪个类别下显示策略定义。

<!-- 
  > [!NOTE]
  > For a [Regulatory Compliance](./regulatory-compliance.md) initiative, the `category` must be
  > **Regulatory Compliance**.
-->

- `preview`（布尔值）：如果策略计划定义为“预览版”，则为 true 或 false 标志。
- `deprecated`（布尔值）：如果策略计划定义被标记为“已弃用”，则为 true 或 false 标志。

> [!NOTE]
> Azure Policy 服务会使用 `version`、`preview` 和 `deprecated` 属性，将变更级别传达给内置策略定义或计划和状态。 `version` 的格式为：`{Major}.{Minor}.{Patch}`。 特定状态（例如“弃用”或“预览版”）会附加至 `version` 属性，或另一个属性中附加为“布尔值”。 有关 Azure Policy 版本内置方式的详细信息，请参阅[内置版本控制](https://github.com/Azure/azure-policy/blob/master/built-in-policies/README.md)。

## <a name="parameters"></a>parameters

参数可减少策略定义的数量，有助于简化策略管理。 使用类似窗体中字段的参数 - `name`、`address`、`city`、`state`。 这些参数始终不变，但其值会基于窗体中的各填写内容变化。
构建策略计划时，参数同样适用。 通过在策略计划定义中包含参数，可以在包含的策略中重复使用该参数。

> [!NOTE]
> 分配计划后，不能更改计划级别参数。 因此，建议在定义参数时设置 defaultValue。

### <a name="parameter-properties"></a>参数属性

参数有下述可以在策略计划定义中使用的属性：

- `name`：参数的名称。 由策略规则中的 `parameters` 部署函数使用。 有关详细信息，请参阅[使用参数值](#passing-a-parameter-value-to-a-policy-definition)。
- `type`：确定参数是“字符串”、“数组”、“对象”、“布尔值”、“整数”、“浮点数”，还是“日期/时间”。
- `metadata`：定义主要由 Azure 门户用来显示用户友好信息的子属性：
  - `description`：说明参数的用途。 可以用来提供可接受值的示例。
  - `displayName`：在门户中显示的用于参数的友好名称。
  - `strongType`：（可选）通过门户分配策略定义时使用。 提供上下文感知列表。 有关详细信息，请参阅 [strongType](#strongtype)。
- `defaultValue`：（可选）设置分配的参数的值（如果值未给定）。
- `allowedValues`：（可选）提供参数在分配过程中所接受值的数组。

例如，可以定义策略计划定义，以限制各种包含的策略定义中的资源位置。 allowedLocations 可以是该策略计划定义的一个参数。 然后，该参数可用于每个包含的策略定义，并在分配策略计划期间定义。

```json
"parameters": {
    "init_allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": [ "chinaeast2" ],
        "allowedValues": [
            "chinaeast",
            "chinanorth2",
            "chinanorth"
        ]
    }
}
```

### <a name="passing-a-parameter-value-to-a-policy-definition"></a>将参数值传递到策略定义

声明要将哪些计划参数传递到计划定义的 [policyDefinitions](#policy-definitions) 数组中的哪些包含的策略定义。 尽管参数名称可以是相同的，但在计划中使用与策略定义中不同的名称可以简化代码以提升可读性。

例如，可以将之前定义的 init_allowedLocations 计划参数传递到多个包含的策略定义及其参数 sql_locations 和 vm_locations，如下所示  ：

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0ec8fc28-d5b7-4603-8fec-39044f00a92b",
        "policyDefinitionReferenceId": "allowedLocationsSQL",
        "parameters": {
            "sql_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/aa09bd0f-aa5f-4343-b6ab-a33a6a6304f3",
        "policyDefinitionReferenceId": "allowedLocationsVMs",
        "parameters": {
            "vm_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    }
]
```

此示例引用 init_allowedLocations 参数，该参数已在[参数属性](#parameter-properties)中演示过。

### <a name="strongtype"></a>strongType

在 `metadata` 属性中，可以使用 **strongType** 提供 Azure 门户中的选项多选列表。 strongType 可以是受支持的资源类型，也可以是允许值。 若要确定资源类型是否对 strongType有效，请使用 [Get-AzResourceProvider](https://docs.microsoft.com/powershell/module/az.resources/get-azresourceprovider)。

支持部分不是由 Get-AzResourceProvider 返回的资源类型。 这些资源类型为：

- `Microsoft.RecoveryServices/vaults/backupPolicies`

strongType 的非资源类型允许值有：

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`

## <a name="policy-definitions"></a>策略定义

计划定义的 `policyDefinitions` 部分是一个数组，其中现有策略定义包含在该计划中。 如[将参数值传递到策略定义](#passing-a-parameter-value-to-a-policy-definition)中所述，此属性是将[计划参数](#parameters)传递到策略定义的位置。

### <a name="policy-definition-properties"></a>策略定义属性

每个表示策略定义的数组元素具有以下属性：

- `policyDefinitionId`（字符串）：要包含的自定义或内置策略定义的 ID。
- `policyDefinitionReferenceId`（字符串）：包含的策略定义的短名称。
- `parameters`：（可选）用于将计划参数作为该策略定义中的属性传递到包含的策略定义的名称/值对。 有关详细信息，请参阅[参数](#parameters)。

<!-- - `groupNames` (array of strings): (Optional) The group the policy definition is a member of. For
  more information, see [Policy groups](#policy-definition-groups). -->

下面是 `policyDefinitions` 的一个示例，，它有两个包含的策略定义，会向这两个策略定义分别传递相同的计划参数：

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0ec8fc28-d5b7-4603-8fec-39044f00a92b",
        "policyDefinitionReferenceId": "allowedLocationsSQL",
        "parameters": {
            "sql_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/aa09bd0f-aa5f-4343-b6ab-a33a6a6304f3",
        "policyDefinitionReferenceId": "allowedLocationsVMs",
        "parameters": {
            "vm_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    }
]
```

<!-- 
## <a name="policy-definition-groups"></a>Policy definitions groups (Preview)

As part of Azure Policy's [Regulatory Compliance](./regulatory-compliance.md) (Preview) feature,
policy definitions in an initiative definition can be grouped. This information is defined in the
`policyDefinitionGroups` _array_ property. These groupings have additional details such as the
**control** and **compliance domain** that the policy definition provides coverage towards.
Additional grouping details may be found in a **policyMetadata** object created by Microsoft. For
information, see [metadata objects](#metadata-objects).

### Policy definition groups parameters

Each _array_ element in `policyDefinitionGroups` must have both of the following properties:

- `name` (string) \[required\]: The short name for the **control**. The value of this property is
  used by `groupNames` in `policyDefinitions`.
- `category` (string): The **compliance domain** of the control.
- `displayName` (string): The friendly name for the **control**. Used by the portal.
- `description` (string): A description of what the **control** does.
- `additionalMetadataId` (string): The location of the [policyMetadata](#metadata-objects) object
  that has additional details about the **control** and **compliance domain**.

  > [!NOTE]
  > Customers may point to an existing [policyMetadata](#metadata-objects) object. However, these
  > objects are _read-only_ and only created by Microsoft.

An example of the `policyDefinitionGroups` property from the NIST built-in initiative definition
looks like this:

```json
"policyDefinitionGroups": [
    {
        "name": "NIST_SP_800-53_R4_AC-1",
        "additionalMetadataId": "/providers/Microsoft.PolicyInsights/policyMetadata/NIST_SP_800-53_R4_AC-1"
    }
]
```

### Metadata objects

Regulatory Compliance built-ins created by Microsoft have additional information about each control.
This information is:

- Displayed in the Azure portal on the overview of a **control** on a Regulatory Compliance
  initiative.
- Available via REST API. See the `Microsoft.PolicyInsights` resource provider and the
  [policyMetadata operation group](https://docs.microsoft.com/rest/api/policy-insights/policymetadata/getresource).
- Available via Azure CLI. See the [az policy metadata](/cli/policy/metadata) command.

> [!IMPORTANT]
> Metadata objects for Regulatory Compliance are _read-only_ and can't be created by customers.

The metadata for a policy grouping has the following information in the `properties` node:

- `metadataId`: The **Control ID** the grouping relates to.
- `category` (required): The **compliance domain** the **control** belongs to.
- `title` (required): The friendly name of the **Control ID**.
- `owner` (required): Identifies who has responsibility for the control in Azure: _Customer_,
  _Microsoft_, _Shared_.
- `description`: Additional information about the control.
- `requirements`: Details about responsibility of the implementation of the control.
- `additionalContentUrl`: A link to more information about the control. This property is typically a
  link to the section of documentation that covers this control in the compliance standard.

Below is an example of the **policyMetadata** object. This example metadata belongs to the _NIST SP
800-53 R4 AC-1_ control.

```json
{
  "properties": {
    "metadataId": "NIST SP 800-53 R4 AC-1",
    "category": "Access Control",
    "title": "Access Control Policy and Procedures",
    "owner": "Shared",
    "description": "**The organization:**    \na. Develops, documents, and disseminates to [Assignment: organization-defined personnel or roles]:  \n1. An access control policy that addresses purpose, scope, roles, responsibilities, management commitment, coordination among organizational entities, and compliance; and  \n2. Procedures to facilitate the implementation of the access control policy and associated access controls; and  \n  
\nb. Reviews and updates the current:  \n1. Access control policy [Assignment: organization-defined frequency]; and  \n2. Access control procedures [Assignment: organization-defined frequency].",
    "requirements": "**a.**  The customer is responsible for developing, documenting, and disseminating access control policies and procedures. The customer access control policies and procedures address access to all customer-deployed resources and customer system access (e.g., access to customer-deployed virtual machines, access to customer-built applications).  \n**b.**  The customer is responsible for reviewing and updating access control policies and procedures in accordance with FedRAMP requirements.",
    "additionalContentUrl": "https://nvd.nist.gov/800-53/Rev4/control/AC-1"
  },
  "id": "/providers/Microsoft.PolicyInsights/policyMetadata/NIST_SP_800-53_R4_AC-1",
  "name": "NIST_SP_800-53_R4_AC-1",
  "type": "Microsoft.PolicyInsights/policyMetadata"
}
```
-->

## <a name="next-steps"></a>后续步骤

- 请参阅[定义结构](./definition-structure.md)
- 在 [Azure Policy 示例](../samples/index.md)中查看示例。
- 查看[了解策略效果](effects.md)。
- 了解如何[以编程方式创建策略](../how-to/programmatically-create.md)。
- 了解如何[获取符合性数据](../how-to/get-compliance-data.md)。
- 了解如何[修正不符合的资源](../how-to/remediate-resources.md)。
- 参阅[使用 Azure 管理组来组织资源](../../management-groups/overview.md)，了解什么是管理组。
