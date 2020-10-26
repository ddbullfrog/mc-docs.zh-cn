---
ms.openlocfilehash: 1c0b8b24cc9e56c0624110a2d7bcc67709845121
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92223133"
---
## <a name="add-a-consumer-group-to-your-iot-hub"></a>将使用者组添加到 IoT 中心

应用程序使用使用者组从 Azure IoT 中心提取数据。 在本教程中，你将创建一个使用者组，供即将到来的 Azure 服务用来从 IoT 中心读取数据。

要将使用者组添加到 IoT 中心，请执行以下步骤：

1. 在 [Azure 门户](https://portal.azure.cn/)中打开 IoT 中心。
2. 在左窗格中单击“内置终结点”  ，在顶部窗格中选择“事件”  ，在右窗格的“使用者组”  下面输入名称。 更改 **默认 TTL** 值并将其恢复为原始值后，单击“保存”。

   ![在 IoT 中心创建使用者组](./media/iot-hub-get-started-create-consumer-group/iot-hub-create-consumer-group-azure.png)


   