---
title: 使用 Go 通过 REST 调用获取意向
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 10/19/2020
ms.author: v-johya
ms.openlocfilehash: 482746ae8b0d1d2c20e88b06dfa0bdb5352f93a7
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472471"
---
[参考文档](https://dev.cognitive.azure.cn/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c08) | [示例](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/LUIS/go-rest-predict/predict.go)

## <a name="prerequisites"></a>先决条件

* [Go](https://golang.org/) 编程语言
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="create-pizza-app"></a>创建 Pizza 应用

[!INCLUDE [Create pizza app](get-started-get-intent-create-pizza-app.md)]

## <a name="get-intent-programmatically"></a>以编程方式获取意向

使用 Go 查询[预测终结点](https://aka.ms/luis-apim-v3-prediction)并获取预测结果。

1. 创建名为 `predict.go` 的新文件。 添加以下代码：

```go
//
// This quickstart shows how to predict the intent of an utterance by using the LUIS REST APIs.
//

package main

// Import dependencies.
import (
    "fmt"
    "net/http"
    "net/url"
    "io/ioutil"
    "log"
)

func main() {

    //////////
    // Values to modify.

    // YOUR-APP-ID: The App ID GUID found on the luis.azure.cn Application Settings page.
    var appID = "YOUR-APP-ID"

    // YOUR-PREDICTION-KEY: Your LUIS authoring key, 32 character value.
    var predictionKey = "YOUR-PREDICTION-KEY"

    // YOUR-PREDICTION-ENDPOINT: Replace with your authoring key endpoint.
    // For example, "https://api.cognitive.azure.cn/"
    var predictionEndpoint = "https://YOUR-PREDICTION-ENDPOINT/"

    // utterance for public app
    var utterance = "I want two large pepperoni pizzas on thin crust please"
    //////////

    // Call the prediction endpoint.
    endpointPrediction(appID, predictionKey, predictionEndpoint, utterance)
}

// Calls the prediction endpoint and displays the prediction results on the console.
func endpointPrediction(appID string, predictionKey string, predictionEndpoint string, utterance string) {

    var endpointUrl = fmt.Sprintf("%sluis/prediction/v3.0/apps/%s/slots/production/predict?subscription-key=%s&verbose=true&show-all-intents=true&query=%s", predictionEndpoint, appID, predictionKey, url.QueryEscape(utterance))

    response, err := http.Get(endpointUrl)

    if err != nil {
        // handle error
        fmt.Println("error from Get")
        log.Fatal(err)
    }

    response2, err2 := ioutil.ReadAll(response.Body)

    if err2 != nil {
        // handle error
        fmt.Println("error from ReadAll")
        log.Fatal(err2)
    }

    fmt.Println("response")
    fmt.Println(string(response2))
}
```

1. 将以 `YOUR-` 开头的值替换为你自己的值。

    |信息|目的|
    |--|--|
    |`YOUR-APP-ID`|你的应用程序 ID。 位于 LUIS 门户中，你的应用的“应用程序设置”页。
    |`YOUR-PREDICTION-KEY`|32 字符预测密钥。 位于 LUIS 门户中，你的应用的“Azure 资源”页。
    |`YOUR-PREDICTION-ENDPOINT`| 预测 URL 终结点。 位于 LUIS 门户中，你的应用的“Azure 资源”页。<br>例如，`https://api.cognitive.azure.cn/`。|

1. 在创建该文件的同一目录中，在命令提示符下输入以下命令来编译 Go 文件：

    ```console
    go build predict.go
    ```

1. 通过在命令提示符下输入以下文本从命令行运行 Go 应用程序：

    ```console
    go run predict.go
    ```

1. 查看以 JSON 形式返回的预测响应：

    ```console
    response
    {"query":"I want two large pepperoni pizzas on thin crust please","prediction":{"topIntent":"ModifyOrder","intents":{"ModifyOrder":{"score":1.0},"None":{"score":8.55E-09},"Greetings":{"score":1.82222226E-09},"CancelOrder":{"score":1.47272727E-09},"Confirmation":{"score":9.8125E-10}},"entities":{"Order":[{"FullPizzaWithModifiers":[{"PizzaType":["pepperoni pizzas"],"Size":[["Large"]],"Quantity":[2],"Crust":[["Thin"]],"$instance":{"PizzaType":[{"type":"PizzaType","text":"pepperoni pizzas","startIndex":17,"length":16,"score":0.9978157,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Size":[{"type":"SizeList","text":"large","startIndex":11,"length":5,"score":0.9984481,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Quantity":[{"type":"builtin.number","text":"two","startIndex":7,"length":3,"score":0.999770939,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Crust":[{"type":"CrustList","text":"thin crust","startIndex":37,"length":10,"score":0.933985531,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}]}}],"$instance":{"FullPizzaWithModifiers":[{"type":"FullPizzaWithModifiers","text":"two large pepperoni pizzas on thin crust","startIndex":7,"length":40,"score":0.90681237,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}]}}],"ToppingList":[["Pepperoni"]],"$instance":{"Order":[{"type":"Order","text":"two large pepperoni pizzas on thin crust","startIndex":7,"length":40,"score":0.9047088,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"ToppingList":[{"type":"ToppingList","text":"pepperoni","startIndex":17,"length":9,"modelTypeId":5,"modelType":"List Entity Extractor","recognitionSources":["model"]}]}}}}
    ```

    已通过格式化提高可读性的 JSON 响应：

    ```json
    response
    {
      "query": "I want two large pepperoni pizzas on thin crust please",
      "prediction": {
        "topIntent": "ModifyOrder",
        "intents": {
          "ModifyOrder": {
            "score": 1
          },
          "None": {
            "score": 8.55e-9
          },
          "Greetings": {
            "score": 1.82222226e-9
          },
          "CancelOrder": {
            "score": 1.47272727e-9
          },
          "Confirmation": {
            "score": 9.8125e-10
          }
        },
        "entities": {
          "Order": [
            {
              "FullPizzaWithModifiers": [
                {
                  "PizzaType": [
                    "pepperoni pizzas"
                  ],
                  "Size": [
                    [
                      "Large"
                    ]
                  ],
                  "Quantity": [
                    2
                  ],
                  "Crust": [
                    [
                      "Thin"
                    ]
                  ],
                  "$instance": {
                    "PizzaType": [
                      {
                        "type": "PizzaType",
                        "text": "pepperoni pizzas",
                        "startIndex": 17,
                        "length": 16,
                        "score": 0.9978157,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                          "model"
                        ]
                      }
                    ],
                    "Size": [
                      {
                        "type": "SizeList",
                        "text": "large",
                        "startIndex": 11,
                        "length": 5,
                        "score": 0.9984481,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                          "model"
                        ]
                      }
                    ],
                    "Quantity": [
                      {
                        "type": "builtin.number",
                        "text": "two",
                        "startIndex": 7,
                        "length": 3,
                        "score": 0.999770939,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                          "model"
                        ]
                      }
                    ],
                    "Crust": [
                      {
                        "type": "CrustList",
                        "text": "thin crust",
                        "startIndex": 37,
                        "length": 10,
                        "score": 0.933985531,
                        "modelTypeId": 1,
                        "modelType": "Entity Extractor",
                        "recognitionSources": [
                          "model"
                        ]
                      }
                    ]
                  }
                }
              ],
              "$instance": {
                "FullPizzaWithModifiers": [
                  {
                    "type": "FullPizzaWithModifiers",
                    "text": "two large pepperoni pizzas on thin crust",
                    "startIndex": 7,
                    "length": 40,
                    "score": 0.90681237,
                    "modelTypeId": 1,
                    "modelType": "Entity Extractor",
                    "recognitionSources": [
                      "model"
                    ]
                  }
                ]
              }
            }
          ],
          "ToppingList": [
            [
              "Pepperoni"
            ]
          ],
          "$instance": {
            "Order": [
              {
                "type": "Order",
                "text": "two large pepperoni pizzas on thin crust",
                "startIndex": 7,
                "length": 40,
                "score": 0.9047088,
                "modelTypeId": 1,
                "modelType": "Entity Extractor",
                "recognitionSources": [
                  "model"
                ]
              }
            ],
            "ToppingList": [
              {
                "type": "ToppingList",
                "text": "pepperoni",
                "startIndex": 17,
                "length": 9,
                "modelTypeId": 5,
                "modelType": "List Entity Extractor",
                "recognitionSources": [
                  "model"
                ]
              }
            ]
          }
        }
      }
    }
    ```

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，请从文件系统中删除该文件。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [添加言语并进行训练](../get-started-get-model-rest-apis.md)

