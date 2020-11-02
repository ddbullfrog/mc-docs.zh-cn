---
title: 快速入门：使用 Azure REST API 和 cURL 检测图像中的人脸
titleSuffix: Azure Cognitive Services
description: 在本快速入门中，请使用 Azure 人脸 REST API 和 cURL 检测图像中的人脸。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: quickstart
origin.date: 07/03/2019
ms.date: 10/27/2020
ms.author: v-johya
ms.openlocfilehash: 0cdfe07345cca2a6fd0c50903be1e34388e4da0b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105450"
---
# <a name="quickstart-detect-faces-in-an-image-using-the-face-rest-api-and-curl"></a>快速入门：使用人脸 REST API 和 cURL 检测图像中的人脸

本快速入门将通过 cURL 使用 Azure 人脸 REST API 来检测图像中的人脸。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/details/cognitive-services/)。 

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [免费创建订阅](https://www.azure.cn/pricing/details/cognitive-services/)
* 拥有 Azure 订阅后，在 Azure 门户中<a href="https://portal.azure.cn/#create/Microsoft.CognitiveServicesFace"  title="创建人脸资源"  target="_blank">创建人脸资源 <span class="docon docon-navigate-external x-hidden-focus"></span></a>，获取密钥和终结点。 部署后，单击“转到资源”。
    * 需要从创建的资源获取密钥和终结点，以便将应用程序连接到人脸 API。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    * 可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。

## <a name="write-the-command"></a>编写命令
 
将使用如下所示的命令来调用人脸 API 并获取图像中的人脸属性数据。 首先，将代码复制到文本编辑器中&mdash;在运行它之前，需对命令的某些部分进行更改。

```shell
# <detection_model_2>
curl -H "Ocp-Apim-Subscription-Key: TODO_INSERT_YOUR_FACE_SUBSCRIPTION_KEY_HERE" "TODO_INSERT_YOUR_FACE_ENDPOINT_HERE/face/v1.0/detect?detectionModel=detection_02&returnFaceId=true&returnFaceLandmarks=false" -H "Content-Type: application/json" --data-ascii "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}"
# </detection_model_2>

# <detection_model_1>
curl -H "Ocp-Apim-Subscription-Key: TODO_INSERT_YOUR_FACE_SUBSCRIPTION_KEY_HERE" "TODO_INSERT_YOUR_FACE_ENDPOINT_HERE/face/v1.0/detect?detectionModel=detection_01&returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise" -H "Content-Type: application/json" --data-ascii "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}"
# </detection_model_1>

# <detection_model_2_with_stream>
curl --location --request POST 'TODO_INSERT_YOUR_FACE_ENDPOINT_HERE/face/v1.0/detect?detectionModel=detection_02&returnFaceId=true&returnFaceLandmarks=false' --header 'Ocp-Apim-Subscription-Key: TODO_INSERT_YOUR_FACE_SUBSCRIPTION_KEY_HERE' --header 'Content-Type: application/octet-stream' --data-binary 'TODO_INSERT_PATH_TO_IMAGE_FILE_HERE'
# </detection_model_2_with_stream>

# <detection_model_1_with_stream>
curl --location --request POST 'TODO_INSERT_YOUR_FACE_ENDPOINT_HERE/face/v1.0/detect?detectionModel=detection_01&returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise' --header 'Ocp-Apim-Subscription-Key: TODO_INSERT_YOUR_FACE_SUBSCRIPTION_KEY_HERE' --header 'Content-Type: application/octet-stream' --data-binary 'TODO_INSERT_PATH_TO_IMAGE_FILE_HERE'
# </detection_model_1_with_stream>
```

### <a name="subscription-key"></a>订阅密钥
将 `<Subscription Key>` 替换为有效的人脸订阅密钥。

### <a name="face-endpoint-url"></a>人脸终结点 URL

URL `https://<My Endpoint String>.cn/face/v1.0/detect` 指示要查询的 Azure 人脸终结点。 可能需更改此 URL 的第一部分，使之与订阅密钥的相应终结点匹配。

[!INCLUDE [subdomains-note](../../../../includes/cognitive-services-custom-subdomains-note.md)]

### <a name="image-source-url"></a>图像源 URL
源 URL 指示将要用作输入的图像。 可以更改此字段，使之指向要分析的任何图像。

```
https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg
``` 

## <a name="run-the-command"></a>运行命令

进行更改以后，请打开命令提示符并输入新的命令。 应该会在控制台窗口中看到显示为 JSON 数据的人脸信息。 例如：

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    }
  }
]  
```

## <a name="extract-face-attributes"></a>提取人脸属性
 
若要提取人脸属性，请使用检测模型 1 并添加 `returnFaceAttributes` 查询参数。 该命令现在应如下所示。 与之前一样，请插入你的人脸订阅密钥和终结点。

```shell
# <detection_model_2>
curl -H "Ocp-Apim-Subscription-Key: TODO_INSERT_YOUR_FACE_SUBSCRIPTION_KEY_HERE" "TODO_INSERT_YOUR_FACE_ENDPOINT_HERE/face/v1.0/detect?detectionModel=detection_02&returnFaceId=true&returnFaceLandmarks=false" -H "Content-Type: application/json" --data-ascii "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}"
# </detection_model_2>

# <detection_model_1>
curl -H "Ocp-Apim-Subscription-Key: TODO_INSERT_YOUR_FACE_SUBSCRIPTION_KEY_HERE" "TODO_INSERT_YOUR_FACE_ENDPOINT_HERE/face/v1.0/detect?detectionModel=detection_01&returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise" -H "Content-Type: application/json" --data-ascii "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}"
# </detection_model_1>

# <detection_model_2_with_stream>
curl --location --request POST 'TODO_INSERT_YOUR_FACE_ENDPOINT_HERE/face/v1.0/detect?detectionModel=detection_02&returnFaceId=true&returnFaceLandmarks=false' --header 'Ocp-Apim-Subscription-Key: TODO_INSERT_YOUR_FACE_SUBSCRIPTION_KEY_HERE' --header 'Content-Type: application/octet-stream' --data-binary 'TODO_INSERT_PATH_TO_IMAGE_FILE_HERE'
# </detection_model_2_with_stream>

# <detection_model_1_with_stream>
curl --location --request POST 'TODO_INSERT_YOUR_FACE_ENDPOINT_HERE/face/v1.0/detect?detectionModel=detection_01&returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise' --header 'Ocp-Apim-Subscription-Key: TODO_INSERT_YOUR_FACE_SUBSCRIPTION_KEY_HERE' --header 'Content-Type: application/octet-stream' --data-binary 'TODO_INSERT_PATH_TO_IMAGE_FILE_HERE'
# </detection_model_1_with_stream>
```

返回的人脸信息现在包含人脸属性。 例如：

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    },
    "faceAttributes": {
      "smile": 0,
      "headPose": {
        "pitch": 0,
        "roll": 0.1,
        "yaw": -32.9
      },
      "gender": "female",
      "age": 22.9,
      "facialHair": {
        "moustache": 0,
        "beard": 0,
        "sideburns": 0
      },
      "glasses": "NoGlasses",
      "emotion": {
        "anger": 0,
        "contempt": 0,
        "disgust": 0,
        "fear": 0,
        "happiness": 0,
        "neutral": 0.986,
        "sadness": 0.009,
        "surprise": 0.005
      },
      "blur": {
        "blurLevel": "low",
        "value": 0.06
      },
      "exposure": {
        "exposureLevel": "goodExposure",
        "value": 0.67
      },
      "noise": {
        "noiseLevel": "low",
        "value": 0
      },
      "makeup": {
        "eyeMakeup": true,
        "lipMakeup": true
      },
      "accessories": [],
      "occlusion": {
        "foreheadOccluded": false,
        "eyeOccluded": false,
        "mouthOccluded": false
      },
      "hair": {
        "bald": 0,
        "invisible": false,
        "hairColor": [
          {
            "color": "brown",
            "confidence": 1
          },
          {
            "color": "black",
            "confidence": 0.87
          },
          {
            "color": "other",
            "confidence": 0.51
          },
          {
            "color": "blond",
            "confidence": 0.08
          },
          {
            "color": "red",
            "confidence": 0.08
          },
          {
            "color": "gray",
            "confidence": 0.02
          }
        ]
      }
    }
  }
]
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你编写了一个 cURL 命令，该命令调用 Azure 人脸服务检测图像中的人脸并返回其属性。 接下来，请浏览人脸 API 参考文档，以便进行详细的了解。

> [!div class="nextstepaction"]
> [人脸 API](https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)

