---
title: 快速入门：使用 REST API 和 Go 检测图像中的人脸
titleSuffix: Azure Cognitive Services
description: 在本快速入门中，你将使用人脸服务和 Go 检测图像中的人脸。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: quickstart
origin.date: 07/03/2019
ms.date: 10/27/2020
ms.author: v-johya
ms.openlocfilehash: f57b0f753478c02c5ca009dfc4b750c19a0414ed
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105445"
---
# <a name="quickstart-detect-faces-in-an-image-using-the-rest-api-and-go"></a>快速入门：使用 REST API 和 Go 检测图像中的人脸

本快速入门将通过 Go 使用 Azure 人脸 REST API 来检测图像中的人脸。

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [免费创建订阅](https://www.azure.cn/pricing/details/cognitive-services/)
* 拥有 Azure 订阅后，在 Azure 门户中<a href="https://portal.azure.cn/#create/Microsoft.CognitiveServicesFace"  title="创建人脸资源"  target="_blank">创建人脸资源 <span class="docon docon-navigate-external x-hidden-focus"></span></a>，获取密钥和终结点。 部署后，单击“转到资源”。
    * 需要从创建的资源获取密钥和终结点，以便将应用程序连接到人脸 API。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    * 可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。
- 代码编辑器，如 [Visual Studio Code](https://code.visualstudio.com/download)

## <a name="write-the-script"></a>编写脚本

创建一个新文件 ( _faceDetection.go_ )，并添加以下代码。 这将根据给定的图像 URL 调用人脸 API。

```go
package main

import (
    "crypto/tls"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "os"
    "strings"
    "time"
)

func main() {
    // Add FACE_SUBSCRIPTION_KEY and FACE_ENDPOINT to your environment variables.
    subscriptionKey := os.Getenv("FACE_SUBSCRIPTION_KEY")
    endpoint := os.Getenv("FACE_ENDPOINT")

    const imageUrl =
      "https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg"

    const params = "?detectionModel=detection_02"
    uri := endpoint + "/face/v1.0/detect" + params
    const imageUrlEnc = "{\"url\":\"" + imageUrl + "\"}"

    reader := strings.NewReader(imageUrlEnc)

    //Configure TLS, etc.
    tr := &http.Transport{
        TLSClientConfig: &tls.Config{
            InsecureSkipVerify: true,
        },
    }
    
    // Create the Http client
    client := &http.Client{
        Transport: tr,
        Timeout: time.Second * 2,
    }

    // Create the Post request, passing the image URL in the request body
    req, err := http.NewRequest("POST", uri, reader)
    if err != nil {
        panic(err)
    }

    // Add headers
    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)

    // Send the request and retrieve the response
    resp, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer resp.Body.Close()

    // Read the response body.
    // Note, data is a byte array
    data, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        panic(err)
    }

    // Parse the Json data
    var f interface{}
    json.Unmarshal(data, &f)

    // Format and display the Json result
    jsonFormatted, _ := json.MarshalIndent(f, "", "  ")
    fmt.Println(string(jsonFormatted))
}
```

需使用订阅密钥更新 `subscriptionKey` 值，并更改 `uriBase` 字符串，使之包含正确的终结点字符串。

[!INCLUDE [subdomains-note](../../../../includes/cognitive-services-custom-subdomains-note.md)]

你可能还希望更改 `imageUrl` 字段以指向自己的输入图像。 你还可能想要更改 `returnFaceAttributes` 字段，该字段指定要检索的人脸属性。

## <a name="run-the-script"></a>运行脚本

打开命令提示符并使用以下命令生成程序：

```shell
go build faceDetection.go
```

然后运行该程序：

```shell
detect-face
```

应该会看到输出到控制台的已检测到人脸数据的 JSON 字符串。 下面是成功的 JSON 响应的示例。

```json
[
  {
    "faceId": "ae8952c1-7b5e-4a5a-a330-a6aa351262c9",
    "faceRectangle": {
      "top": 621,
      "left": 616,
      "width": 195,
      "height": 195
    }
  }
]
```

## <a name="extract-face-attributes"></a>提取人脸属性
 
若要提取人脸属性，请使用检测模型 1 并添加 `returnFaceAttributes` 查询参数。

```go
const params = "?detectionModel=detection_01&returnFaceAttributes=age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise"
```

响应现在包含人脸属性。 例如：

```json
[
  {
    "faceId": "ae8952c1-7b5e-4a5a-a330-a6aa351262c9",
    "faceRectangle": {
      "top": 621,
      "left": 616,
      "width": 195,
      "height": 195
    },
    "faceAttributes": {
      "smile": 0,
      "headPose": {
        "pitch": 0,
        "roll": 6.8,
        "yaw": 3.7
      },
      "gender": "male",
      "age": 37,
      "facialHair": {
        "moustache": 0.4,
        "beard": 0.4,
        "sideburns": 0.1
      },
      "glasses": "NoGlasses",
      "emotion": {
        "anger": 0,
        "contempt": 0,
        "disgust": 0,
        "fear": 0,
        "happiness": 0,
        "neutral": 0.999,
        "sadness": 0.001,
        "surprise": 0
      },
      "blur": {
        "blurLevel": "high",
        "value": 0.89
      },
      "exposure": {
        "exposureLevel": "goodExposure",
        "value": 0.51
      },
      "noise": {
        "noiseLevel": "medium",
        "value": 0.59
      },
      "makeup": {
        "eyeMakeup": true,
        "lipMakeup": false
      },
      "accessories": [],
      "occlusion": {
        "foreheadOccluded": false,
        "eyeOccluded": false,
        "mouthOccluded": false
      },
      "hair": {
        "bald": 0.04,
        "invisible": false,
        "hairColor": [
          {
            "color": "black",
            "confidence": 0.98
          },
          {
            "color": "brown",
            "confidence": 0.87
          },
          {
            "color": "gray",
            "confidence": 0.85
          },
          {
            "color": "other",
            "confidence": 0.25
          },
          {
            "color": "blond",
            "confidence": 0.07
          },
          {
            "color": "red",
            "confidence": 0.02
          }
        ]
      }
    }
  },
  {
    "faceId": "b1bb3cbe-5a73-4f8d-96c8-836a5aca9415",
    "faceRectangle": {
      "top": 693,
      "left": 1503,
      "width": 180,
      "height": 180
    },
    "faceAttributes": {
      "smile": 0.003,
      "headPose": {
        "pitch": 0,
        "roll": 2,
        "yaw": -2.2
      },
      "gender": "female",
      "age": 56,
      "facialHair": {
        "moustache": 0,
        "beard": 0,
        "sideburns": 0
      },
      "glasses": "NoGlasses",
      "emotion": {
        "anger": 0,
        "contempt": 0.001,
        "disgust": 0,
        "fear": 0,
        "happiness": 0.003,
        "neutral": 0.984,
        "sadness": 0.011,
        "surprise": 0
      },
      "blur": {
        "blurLevel": "high",
        "value": 0.83
      },
      "exposure": {
        "exposureLevel": "goodExposure",
        "value": 0.41
      },
      "noise": {
        "noiseLevel": "high",
        "value": 0.76
      },
      "makeup": {
        "eyeMakeup": false,
        "lipMakeup": false
      },
      "accessories": [],
      "occlusion": {
        "foreheadOccluded": false,
        "eyeOccluded": false,
        "mouthOccluded": false
      },
      "hair": {
        "bald": 0.06,
        "invisible": false,
        "hairColor": [
          {
            "color": "black",
            "confidence": 0.99
          },
          {
            "color": "gray",
            "confidence": 0.89
          },
          {
            "color": "other",
            "confidence": 0.64
          },
          {
            "color": "brown",
            "confidence": 0.34
          },
          {
            "color": "blond",
            "confidence": 0.07
          },
          {
            "color": "red",
            "confidence": 0.03
          }
        ]
      }
    }
  }
]
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你编写了一个 Go 控制台应用程序，该应用程序调用 Azure 人脸服务检测图像中的人脸并返回其属性。 接下来，请浏览人脸 API 参考文档，以便进行详细的了解。

> [!div class="nextstepaction"]
> [人脸 API](https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)

