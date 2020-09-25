---
title: 在 Azure 数据工厂中进行迭代开发和调试
description: 了解如何在 ADF UX 中以迭代方式开发和调试数据工厂管道
origin.date: 08/28/2020
ms.date: 09/21/2020
ms.topic: conceptual
ms.service: data-factory
services: data-factory
documentationcenter: ''
ms.workload: data-services
author: WenJason
ms.author: v-jay
ms.openlocfilehash: 1a614972516526891720c74b849caa5384a9b0c2
ms.sourcegitcommit: f5d53d42d58c76bb41da4ea1ff71e204e92ab1a7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90523889"
---
# <a name="iterative-development-and-debugging-with-azure-data-factory"></a>使用 Azure 数据工厂进行迭代开发和调试
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

通过 Azure 数据工厂，你可以在开发数据集成解决方案时以迭代方式开发和调试数据工厂管道。 这些功能使你能够在创建拉取请求或将其发布到数据工厂服务之前测试更改。 

## <a name="debugging-a-pipeline"></a>调试管道

当你使用管道画布进行创作时，可使用“调试”功能测试活动。 执行测试运行时，在选择“调试”之前，不需要将更改发布到数据工厂。  当希望确保更改在你更新数据工厂工作流之前按预期工作时，此功能很有帮助。

![管道画布上的调试功能](media/iterative-development-debugging/iterative-development-1.png)

管道运行时，你可在管道画布的“输出”选项卡中查看每个活动的结果。

在管道画布的“输出”  窗口中查看测试运行的结果。

![管道画布“输出”窗口](media/iterative-development-debugging/iterative-development-2.png)

在测试运行成功后，向管道中添加更多活动并继续以迭代方式进行调试。 还可以在测试运行正在执行时将其**取消**。

> [!IMPORTANT]
> 选择”调试”  会实际运行管道。 例如，如果管道包含复制活动，则测试运行会将数据从源复制到目标。 因此，在调试时，建议在复制活动和其他活动中使用测试文件夹。 在调试管道后，切换到要在正常操作中使用的实际文件夹。

### <a name="setting-breakpoints"></a>设置断点

Azure 数据工厂还允许你一直调试管道，直到到达管道画布中的某个特定活动。 在活动上放置要测试到的断点，然后选择“调试”即可。 数据工厂会确保测试仅运行到管道画布上的断点活动。 如果不想测试整个管道，只想测试该管道内的一部分活动，则此“调试至”  功能非常有用。

![管道画布上的断点](media/iterative-development-debugging/iterative-development-3.png)

若要设置断点，请选择管道画布上的元素。 “调试至”  选项在元素的右上角显示为空心的红色圆圈。

![在所选元素上设置断点之前](media/iterative-development-debugging/iterative-development-4.png)

选择“调试至”选项后，它将变为实心的红色圆圈，以指示已启用断点  。

![在所选元素上设置断点之后](media/iterative-development-debugging/iterative-development-5.png)

## <a name="monitoring-debug-runs"></a>监视调试运行

运行管道调试运行时，结果将显示在管道画布的“输出”窗口中。 “输出”选项卡只包含在当前浏览器会话过程中出现的最新运行。 

![管道画布“输出”窗口](media/iterative-development-debugging/iterative-development-2.png)

若要查看调试运行的历史视图或查看所有活动调试运行的列表，你可以进入“监视器”体验。 

![选择查看活动调试运行图标](media/iterative-development-debugging/view-debug-runs.png)

> [!NOTE]
> Azure 数据工厂服务仅将调试运行历史记录保留 15 天。 

