---
title: 使用 Java 通过 REST 调用获取意向
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 10/19/2020
ms.author: v-johya
ms.openlocfilehash: efefe0ff59484238240e69f9c3e345b492cdb752
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472468"
---
[参考文档](https://dev.cognitive.azure.cn/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c08) | [示例](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/LUIS/java-predict-with-rest/Predict.java)

## <a name="prerequisites"></a>先决条件

* [JDK SE](https://aka.ms/azure-jdks)（Java 开发工具包，标准版）
* [Visual Studio Code](https://code.visualstudio.com/) 或你喜欢用的 IDE

## <a name="create-pizza-app"></a>创建 Pizza 应用

[!INCLUDE [Create pizza app](get-started-get-intent-create-pizza-app.md)]

## <a name="get-intent-programmatically"></a>以编程方式获取意向

使用 Java 查询[预测终结点](https://aka.ms/luis-apim-v3-prediction)并获取预测结果。

1. 创建一个新文件夹以保存 Java 项目，例如 `java-predict-with-rest`。

1. 创建一个名为 `lib` 的子目录，并将以下 Java 库中的内容复制到 `lib` 子目录：

    * [commons-logging-1.2.jar](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-language-understanding/master/documentation-samples/quickstarts/analyze-text/java/lib/commons-logging-1.2.jar)
    * [httpclient-4.5.3.jar](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-language-understanding/master/documentation-samples/quickstarts/analyze-text/java/lib/httpclient-4.5.3.jar)
    * [httpcore-4.4.6.jar](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-language-understanding/master/documentation-samples/quickstarts/analyze-text/java/lib/httpcore-4.4.6.jar)

1. 复制以下代码以在名为 `Predict.java` 的文件中创建一个类：

```java
//
// This quickstart shows how to predict the intent of an utterance by using the LUIS REST APIs.
//

import java.io.*;
import java.net.URI;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

// To compile, execute this command at the console:
//      Windows: javac -cp ";lib/*" Predict.java
//      macOs: javac -cp ":lib/*" Predict.java
//      Linux: javac -cp ":lib/*" Predict.java

// To run, execute this command at the console:
//      Windows: java -cp ";lib/*" Predict
//      macOs: java -cp ":lib/*" Predict
//      Linux: java -cp ":lib/*" Predict

public class Predict {

    public static void main(String[] args)
    {
        HttpClient httpclient = HttpClients.createDefault();

        try
        {
            //////////
            // Values to modify.

            // YOUR-APP-ID: The App ID GUID found on the luis.azure.cn Application Settings page.
            String AppId = "YOUR-APP-ID";

            // YOUR-PREDICTION-KEY: Your LUIS authoring key, 32 character value.
            String Key = "YOUR-PREDICTION-KEY";

            // YOUR-PREDICTION-ENDPOINT: Replace this with your authoring key endpoint.
            // For example, "https://api.cognitive.azure.cn/"
            String Endpoint = "https://YOUR-PREDICTION-ENDPOINT/";

            // The utterance you want to use.
            String Utterance = "I want two large pepperoni pizzas on thin crust please";
            //////////

            // Begin building the endpoint URL.
            URIBuilder endpointURLbuilder = new URIBuilder(Endpoint + "luis/prediction/v3.0/apps/" + AppId + "/slots/production/predict?");

            // Create the query string params.
            endpointURLbuilder.setParameter("query", Utterance);
            endpointURLbuilder.setParameter("subscription-key", Key);
            endpointURLbuilder.setParameter("show-all-intents", "true");
            endpointURLbuilder.setParameter("verbose", "true");

            // Create the prediction endpoint URL.
            URI endpointURL = endpointURLbuilder.build();

            // Create the HTTP object from the URL.
            HttpGet request = new HttpGet(endpointURL);

            // Access the LUIS endpoint to analyze the text utterance.
            HttpResponse response = httpclient.execute(request);

            // Get the response.
            HttpEntity entity = response.getEntity();

            // Print the response on the console.
            if (entity != null)
            {
                System.out.println(EntityUtils.toString(entity));
            }
        }

        // Display errors if they occur.
        catch (Exception e)
        {
            System.out.println(e.getMessage());
        }
    }
}
```

1. 将以 `YOUR-` 开头的值替换为你自己的值。

    |信息|目的|
    |--|--|
    |`YOUR-APP-ID`|你的应用程序 ID。 位于 LUIS 门户中，你的应用的“应用程序设置”页。
    |`YOUR-PREDICTION-KEY`|32 字符预测密钥。 位于 LUIS 门户中，你的应用的“Azure 资源”页。
    |`YOUR-PREDICTION-ENDPOINT`| 预测 URL 终结点。 位于 LUIS 门户中，你的应用的“Azure 资源”页。<br>例如，`https://api.cognitive.azure.cn/`。|

1. 通过命令行编译 Java 程序。

    * 如果使用的是 Windows，请使用此命令：`javac -cp ";lib/*" Predict.java`
    * 如果使用的是 macOS 或 Linux，请使用此命令：`javac -cp ":lib/*" Predict.java`

1. 通过命令行运行 Java 程序：

    * 如果使用的是 Windows，请使用此命令：`java -cp ";lib/*" Predict`
    * 如果使用的是 macOS 或 Linux，请使用此命令：`java -cp ":lib/*" Predict`

1. 查看以 JSON 形式返回的预测响应：

    ```json
    {"query":"I want two large pepperoni pizzas on thin crust please","prediction":{"topIntent":"ModifyOrder","intents":{"ModifyOrder":{"score":1.0},"None":{"score":8.55E-09},"Greetings":{"score":1.82222226E-09},"CancelOrder":{"score":1.47272727E-09},"Confirmation":{"score":9.8125E-10}},"entities":{"Order":[{"FullPizzaWithModifiers":[{"PizzaType":["pepperoni pizzas"],"Size":[["Large"]],"Quantity":[2],"Crust":[["Thin"]],"$instance":{"PizzaType":[{"type":"PizzaType","text":"pepperoni pizzas","startIndex":17,"length":16,"score":0.9978157,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Size":[{"type":"SizeList","text":"large","startIndex":11,"length":5,"score":0.9984481,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Quantity":[{"type":"builtin.number","text":"two","startIndex":7,"length":3,"score":0.999770939,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"Crust":[{"type":"CrustList","text":"thin crust","startIndex":37,"length":10,"score":0.933985531,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}]}}],"$instance":{"FullPizzaWithModifiers":[{"type":"FullPizzaWithModifiers","text":"two large pepperoni pizzas on thin crust","startIndex":7,"length":40,"score":0.90681237,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}]}}],"ToppingList":[["Pepperoni"]],"$instance":{"Order":[{"type":"Order","text":"two large pepperoni pizzas on thin crust","startIndex":7,"length":40,"score":0.9047088,"modelTypeId":1,"modelType":"Entity Extractor","recognitionSources":["model"]}],"ToppingList":[{"type":"ToppingList","text":"pepperoni","startIndex":17,"length":9,"modelTypeId":5,"modelType":"List Entity Extractor","recognitionSources":["model"]}]}}}}
    ```

    已通过格式化提高可读性的 JSON 响应：

    ```JSON
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

完成本快速入门后，请从文件系统中删除项目文件夹。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用 Java 添加话语并进行训练](../get-started-get-model-rest-apis.md)

