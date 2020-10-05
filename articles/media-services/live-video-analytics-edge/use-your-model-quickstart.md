---
title: 使用自己的 HTTP 模型分析实时视频 - Azure
description: 在本快速入门中，你将应用计算机视觉来分析来自（模拟）IP 相机的实时视频源。
ms.topic: quickstart
author: WenJason
ms.author: v-jay
ms.service: media-services
origin.date: 04/27/2020
ms.date: 09/28/2020
zone_pivot_groups: ams-lva-edge-programming-languages
ms.openlocfilehash: 179955ff687a061c12b6393ec077f6ca106ed7da
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245601"
---
# <a name="quickstart-analyze-live-video-by-using-your-own-http-model"></a>快速入门：使用自己的 HTTP 模型分析实时视频

本快速入门介绍了如何在 IoT Edge 上使用实时视频分析来分析来自（模拟）IP 相机中的实时视频源。 你将了解如何应用计算机视觉模型来检测对象。 实时视频源中的一部分帧被发送到推理服务。 结果将发送到 IoT Edge 中心。 

此快速入门将 Azure VM 用作 IoT Edge 设备，并使用模拟的实时视频流。 它基于用 C# 编写的示例代码，并以[检测运动并发出事件](detect-motion-emit-events-quickstart.md)快速入门为基础。 

# <a name="c"></a><a name="programming-language-csharp"></a>[C#](#tab/programming-language-csharp)
[!INCLUDE [header](includes/analyze-live-video-your-http-model-quickstart/csharp/header.md)]

# <a name="python"></a><a name="programming-language-python"></a>[Python](#tab/programming-language-python)
[!INCLUDE [header](includes/analyze-live-video-your-http-model-quickstart/python/header.md)]
---

## <a name="prerequisites"></a>先决条件

# <a name="c"></a><a name="programming-language-csharp"></a>[C#](#tab/programming-language-csharp)
[!INCLUDE [prerequisites](includes/analyze-live-video-your-http-model-quickstart/csharp/prerequisites.md)]

# <a name="python"></a><a name="programming-language-python"></a>[Python](#tab/programming-language-python)
[!INCLUDE [prerequisites](includes/analyze-live-video-your-http-model-quickstart/python/prerequisites.md)]
---

## <a name="review-the-sample-video"></a>观看示例视频

# <a name="c"></a><a name="programming-language-csharp"></a>[C#](#tab/programming-language-csharp)
[!INCLUDE [review-sample-video](includes/analyze-live-video-your-http-model-quickstart/csharp/review-sample-video.md)]

# <a name="python"></a><a name="programming-language-python"></a>[Python](#tab/programming-language-python)
[!INCLUDE [review-sample-video](includes/analyze-live-video-your-http-model-quickstart/python/review-sample-video.md)]
---

## <a name="overview"></a>概述

# <a name="c"></a><a name="programming-language-csharp"></a>[C#](#tab/programming-language-csharp)
[!INCLUDE [overview](includes/analyze-live-video-your-http-model-quickstart/csharp/overview.md)]

# <a name="python"></a><a name="programming-language-python"></a>[Python](#tab/programming-language-python)
[!INCLUDE [overview](includes/analyze-live-video-your-http-model-quickstart/python/overview.md)]
---

## <a name="create-and-deploy-the-media-graph"></a>创建和部署媒体图

# <a name="c"></a><a name="programming-language-csharp"></a>[C#](#tab/programming-language-csharp)
[!INCLUDE [create-deploy-media-graph](includes/analyze-live-video-your-http-model-quickstart/csharp/create-deploy-media-graph.md)]

# <a name="python"></a><a name="programming-language-python"></a>[Python](#tab/programming-language-python)
[!INCLUDE [create-deploy-media-graph](includes/analyze-live-video-your-http-model-quickstart/python/create-deploy-media-graph.md)]
---

## <a name="interpret-results"></a>解释结果

# <a name="c"></a><a name="programming-language-csharp"></a>[C#](#tab/programming-language-csharp)
[!INCLUDE [interpret-results](includes/analyze-live-video-your-http-model-quickstart/csharp/interpret-results.md)]

# <a name="python"></a><a name="programming-language-python"></a>[Python](#tab/programming-language-python)
[!INCLUDE [interpret-results](includes/analyze-live-video-your-http-model-quickstart/python/interpret-results.md)]
---

## <a name="clean-up-resources"></a>清理资源

如果计划学习其他快速入门，请保留创建的资源。 否则，请转到 Azure 门户，再转到资源组，选择运行本快速入门所用的资源组，并删除所有资源。

## <a name="next-steps"></a>后续步骤

* 试用[安全版本的 YoloV3 模型](https://github.com/Azure/live-video-analytics/blob/master/utilities/video-analysis/tls-yolov3-onnx/readme.md)并将其部署到 IoT Edge 设备。 

查看高级用户面临的其他挑战：

* 使用支持 RTSP 的 [IP 相机](https://en.wikipedia.org/wiki/IP_camera)，而不是使用 RTSP 模拟器。 可以在 [ONVIF 符合标准的产品](https://www.onvif.org/conformant-products/)页面上搜索支持 RTSP 的 IP 摄像机。 查找符合配置文件 G、S 或 T 的设备。
* 使用 AMD64 或 X64 Linux 设备，而不是 Azure Linux VM。 此设备必须与 IP 相机位于同一网络中。 可以按照[在 Linux 上安装 Azure IoT Edge 运行时](../../iot-edge/how-to-install-iot-edge-linux.md)中的说明进行操作。 然后按照[将首个 IoT Edge 模块部署到虚拟 Linux 设备](../../iot-edge/quickstart-linux.md)中的说明，将设备注册到 Azure IoT 中心。
