---
ms.openlocfilehash: d358836e66fdc972ad5df62e3c0ecdd033d8ff0e
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327299"
---
> [!NOTE]
> 当我们从 `https://raw.githubusercontent.com/` 网站下载特定模板文件时，有时会遇到某些问题。
> 
> ![githubusercontent 前缀模板部署问题](./media/azure-raw-githubusercontent-azurechinacloud-environment-notice/githubusercontent_issue.png)
>
> 我们可以按照指示修改模板 URI，然后在本地计算机上下载特定的模板文件。
> 1. 复制模板 URI，通过更改前缀、中缀和模板文件名来转换 URI。
>
>     例如，源 URI 是 `https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-cosmosdb-sql-autoscale/azuredeploy.json`
>
>     | 类别 | 原始值 | 转换后的值 |  操作  |
>     |----------|----------------|-----------------|----------|
>     | 前缀   | `https://raw.githubusercontent.com`  |  `https://github.com`  | 更新 |
>     | 中辍    |                | `blob`          |  在 `master` 之前添加，它是 git 存储库的默认分支名称。 |
>     | 模板文件名  |azuredeploy.json | 你的下载模板文件名 | update |
>
>     修改后，转换后的 URI 看起来将类似于 `https://github.com/Azure/azure-quickstart-templates/blob/master/101-cosmosdb-sql-autoscale/azuredeploy.json`。
>     
> 2. 复制转换后的 URI，并在 Internet 浏览器中手动下载特定的模板内容。
> 3. （可选）如果模板文件太大而无法在原始内容中显示，我们可以选择带工具提示 `Open this file in GitHub Desktop` 的计算机图标，以便在 GitHub 桌面应用程序中克隆存储库。
