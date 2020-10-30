---
title: 向指标顾问服务提供异常反馈
titleSuffix: Azure Cognitive Services
description: 了解如何就指标顾问实例发现的异常发送反馈，并优化结果。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 10/22/2020
ms.author: v-johya
ms.openlocfilehash: 76aa2d734c068c37d318843ec039e639227eb9db
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472940"
---
# <a name="adjust-anomaly-detection-using-feedback"></a>使用反馈调整异常检测

如果你对指标监视器提供的某些异常情况检测结果不满意，可以手动添加反馈来影响应用于数据的模型。 

使用页面右上角的按钮激活反馈批注模式。

:::image type="content" source="../media/feedback/annotation-mode.png" alt-text="反馈批注模式。":::

反馈批注模式激活后，便可为单个点或多个连续点提供反馈。

## <a name="give-feedback-for-one-point"></a>为单个点提供反馈 

反馈批注模式激活后，单击某个点打开“添加反馈”面板。 可设置要应用的反馈类型。 此反馈将融入将来点的检测中。  

* 如果你认为该点被指标监视器错误地标记，请选择“异常”。 可以指定某个点是否应为异常。 
* 如果你认为该点是趋势变化的开始，请选择“ChangePoint”。
* 选择“时间段”以指示季节性。 指标监视器可以自动检测季节性的间隔，也可以手动指定。 

请考虑同时在“注释”文本框中留下注释，然后单击“保存”以保存反馈 。

:::image type="content" source="../media/feedback/feedback-menu.png" alt-text="反馈批注模式。":::

## <a name="give-feedback-for-multiple-continuous-points"></a>为多个连续点提供反馈

可通过单击鼠标并将鼠标拖动到要批注的点上，同时为多个连续点提供反馈。 你将看到与上面相同的反馈菜单。 单击“保存”后，相同的反馈将应用于所有选定的点。

:::image type="content" source="../media/feedback/continuous-points.png" alt-text="反馈批注模式。":::

## <a name="how-to-view-my-feedback"></a>如何查看我的反馈

若要查看某个点的异常情况检测是否已更改，请将鼠标悬停在该点上。 如果检测已更改，工具提示将显示“受反馈影响: true”。 如果显示“False”，则对该点的反馈计算已完成，但异常情况检测结果未更改。

:::image type="content" source="../media/feedback/affected-point.png" alt-text="反馈批注模式。":::

## <a name="when-should-i-annotate-an-anomaly-as-normal"></a>何时应将异常批注为“正常”

当你可能认为异常是错误警报时，有很多原因。 如果满足以下任一情况，请考虑使用以下指标顾问功能：


|方案  |建议 |
|---------|---------|
|异常是由已知数据源变化（例如系统变化）引起的。     | 如果希望此方案不会定期重演，请勿批注此异常情况。        |
|异常是由假日导致的。     | 使用[预设事件](configure-metrics.md#preset-events)指定时间标记异常情况检测。       |
|检测到异常有一个常规模式（例如在周末），它们不应为异常。      |使用反馈功能或预设事件。        |

## <a name="next-steps"></a>后续步骤
- [诊断事件](diagnose-incident.md)。
- [配置指标并微调检测配置](configure-metrics.md)
- [配置警报并使用挂钩获取通知](../how-tos/alerts.md)

