---
title: ServicePrincipalSelector UI 元素
description: 介绍 Azure 门户的 Microsoft.Common.ServicePrincipalSelector UI 元素。 提供一个用于选择应用程序标识符的下拉列表，并提供一个用于输入密码或证书指纹的文本框。
ms.topic: conceptual
origin.date: 09/29/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: yes|no
ms.testdate: 10/26/2020null
ms.author: v-yeche
ms.openlocfilehash: fe5d0a13552b84136a581cd23b531a1274f00b8f
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470557"
---
<!--Verified successfully-->
# <a name="microsoftcommonserviceprincipalselector-ui-element"></a>Microsoft.Common.ServicePrincipalSelector UI 元素

一个可让用户选择现有服务主体或注册新服务主体的控件。 选择“新建”时，需执行注册新应用程序的步骤。 选择现有应用程序时，此控件将提供一个文本框，用于输入密码或证书指纹。

## <a name="ui-sample"></a>UI 示例

默认视图取决于 `defaultValue` 属性中的值。 如果 `principalId` 属性包含有效的全局唯一标识符 (GUID)，则此控件会搜索应用程序的对象 ID。 如果用户没有从下拉列表中进行选择，则会应用默认值。

:::image type="content" source="./media/managed-application-elements/microsoft-common-serviceprincipal-initial.png" alt-text="Microsoft.Common.ServicePrincipalSelector 初始视图":::

从下拉列表中选择“新建”或现有应用程序标识符时，会显示“身份验证类型”，用于在文本框中输入密码或证书指纹。

:::image type="content" source="./media/managed-application-elements/microsoft-common-serviceprincipal-selection.png" alt-text="Microsoft.Common.ServicePrincipalSelector 初始视图":::

## <a name="schema"></a>架构

```json
{
  "name": "ServicePrincipal",
  "type": "Microsoft.Common.ServicePrincipalSelector",
  "label": {
    "principalId": "App Id",
    "password": "Password",
    "certificateThumbprint": "Certificate thumbprint",
    "authenticationType": "Authentication Type",
    "sectionHeader": "Service Principal"
  },
  "toolTip": {
    "principalId": "App Id",
    "password": "Password",
    "certificateThumbprint": "Certificate thumbprint",
    "authenticationType": "Authentication Type"
  },
  "defaultValue": {
    "principalId": "<default guid>",
    "name": "(New) default App Id"
  },
  "constraints": {
    "required": true,
    "regex": "^[a-zA-Z0-9]{8,}$",
    "validationMessage": "Password must be at least 8 characters long, contain only numbers and letters"
  },
  "options": {
    "hideCertificate": false
  },
  "visible": true
}
```

## <a name="remarks"></a>备注

- 必需的属性包括：
  - `name`
  - `type`
  - `label`
  - `defaultValue`：指定默认的 `principalId` 和 `name`。

- 可选属性包括：
  - `toolTip`：将工具提示 `infoBalloon` 附加到每个标签。
  - `visible`：隐藏或显示控件。
  - `options`：指定是否应使证书指纹选项可用。
  - `constraints`：用于密码验证的正则表达式约束。

## <a name="example"></a>示例

下面是 `Microsoft.Common.ServicePrincipalSelector` 控件的一个示例。 `defaultValue` 属性将 `principalId` 设置为 `<default guid>`（默认应用程序标识符 GUID 的占位符）。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [],
    "steps": [
      {
        "name": "SPNcontrol",
        "label": "SPNcontrol",
        "elements": [
          {
            "name": "ServicePrincipal",
            "type": "Microsoft.Common.ServicePrincipalSelector",
            "label": {
              "principalId": "App Id",
              "password": "Password",
              "certificateThumbprint": "Certificate thumbprint",
              "authenticationType": "Authentication Type",
              "sectionHeader": "Service Principal"
            },
            "toolTip": {
              "principalId": "App Id",
              "password": "Password",
              "certificateThumbprint": "Certificate thumbprint",
              "authenticationType": "Authentication Type"
            },
            "defaultValue": {
              "principalId": "<default guid>",
              "name": "(New) default App Id"
            },
            "constraints": {
              "required": true,
              "regex": "^[a-zA-Z0-9]{8,}$",
              "validationMessage": "Password must be at least 8 characters long, contain only numbers and letters"
            },
            "options": {
              "hideCertificate": false
            },
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "appId": "[steps('SPNcontrol').ServicePrincipal.appId]",
      "objectId": "[steps('SPNcontrol').ServicePrincipal.objectId]",
      "password": "[steps('SPNcontrol').ServicePrincipal.password]",
      "certificateThumbprint": "[steps('SPNcontrol').ServicePrincipal.certificateThumbprint]",
      "newOrExisting": "[steps('SPNcontrol').ServicePrincipal.newOrExisting]",
      "authenticationType": "[steps('SPNcontrol').ServicePrincipal.authenticationType]"
    }
  }
}
```

## <a name="example-output"></a>示例输出

`appId` 是所选的或所创建的应用程序注册的 ID。 `objectId` 是为所选应用程序注册配置的服务主体的 objectId 数组。

未从下拉列表中进行选择时，`newOrExisting` 属性值将为“new”：

```json
{
  "appId": {
    "value": "<default guid>"
  },
  "objectId": {
    "value": ["<default guid>"]
  },
  "password": {
    "value": "<password>"
  },
  "certificateThumbprint": {
    "value": ""
  },
  "newOrExisting": {
    "value": "new"
  },
  "authenticationType": {
    "value": "password"
  }
}
```

从下拉列表中选择了“新建”或现有应用程序标识符时，`newOrExisting` 属性值将为“existing”：

```json
{
  "appId": {
    "value": "<guid>"
  },
  "objectId": {
    "value": ["<guid>"]
  },
  "password": {
    "value": "<password>"
  },
  "certificateThumbprint": {
    "value": ""
  },
  "newOrExisting": {
    "value": "existing"
  },
  "authenticationType": {
    "value": "password"
  }
}
```

## <a name="next-steps"></a>后续步骤

- 有关创建 UI 定义的简介，请参阅 [CreateUiDefinition 入门](create-uidefinition-overview.md)。
- 有关 UI 元素中的公用属性的说明，请参阅 [CreateUiDefinition 元素](create-uidefinition-elements.md)。

<!-- Update_Description: new article about microsoft common serviceprincipalselector -->
<!--NEW.date: 10/26/2020-->