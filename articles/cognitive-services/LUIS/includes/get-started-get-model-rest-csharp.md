---
title: 使用 C# 通过 REST 调用获取模型
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 10/19/2020
ms.author: v-johya
ms.custom: devx-track-csharp
ms.openlocfilehash: 8aa32124efe8e6903fd5d63dad891edb53b13e19
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472518"
---
[参考文档](https://dev.cognitive.azure.cn/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c45) | [示例](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/LanguageUnderstanding/csharp-model-with-rest/Program.cs)

## <a name="prerequisites"></a>先决条件

* [.NET Core 3.1](https://dotnet.microsoft.com/download)
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="example-utterances-json-file"></a>示例话语 JSON 文件

[!INCLUDE [Quickstart explanation of example utterance JSON file](get-started-get-model-json-example-utterances.md)]

## <a name="create-pizza-app"></a>创建披萨应用

[!INCLUDE [Create pizza app](get-started-get-model-create-pizza-app.md)]

## <a name="change-model-programmatically"></a>以编程方式更改模型

1. 使用项目和名为 `csharp-model-with-rest` 的文件夹创建一个面向 C# 语言的新控制台应用程序。

    ```console
    dotnet new console -lang C# -n csharp-model-with-rest
    ```

1. 更改为创建的 `csharp-model-with-rest` 目录，并使用以下命令安装所需的依赖项：

    ```console
    cd csharp-model-with-rest
    dotnet add package System.Net.Http
    dotnet add package JsonFormatterPlus
    ```

1. 将 Program.cs 改写为以下代码：

```csharp
//
// This quickstart shows how to add utterances to a LUIS model using the REST APIs.
//

using System;
using System.IO;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using System.Collections.Generic;
using System.Linq;

// 3rd party NuGet packages
using JsonFormatterPlus;

namespace AddUtterances
{
    class Program
    {
        //////////
        // Values to modify.

        // YOUR-APP-ID: The App ID GUID found on the luis.azure.cn Application Settings page.
        static string appID = "YOUR-APP-ID";

        // YOUR-AUTHORING-KEY: Your LUIS authoring key, 32 character value.
        static string authoringKey = "YOUR-AUTHORING-KEY";

        // YOUR-AUTHORING-ENDPOINT: Replace this endpoint with your authoring key endpoint.
        // For example, "https://your-resource-name.cognitiveservices.azure.cn/"
        static string authoringEndpoint = "https://YOUR-AUTHORING-ENDPOINT/";

        // NOTE: Replace this your version number.
        static string appVersion = "0.1";
        //////////

        static string host = String.Format("{0}luis/authoring/v3.0-preview/apps/{1}/versions/{2}/", authoringEndpoint, appID, appVersion);

        // GET request with authentication
        async static Task<HttpResponseMessage> SendGet(string uri)
        {
            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage())
            {
                request.Method = HttpMethod.Get;
                request.RequestUri = new Uri(uri);
                request.Headers.Add("Ocp-Apim-Subscription-Key", authoringKey);
                return await client.SendAsync(request);
            }
        }

        // POST request with authentication
        async static Task<HttpResponseMessage> SendPost(string uri, string requestBody)
        {
            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage())
            {
                request.Method = HttpMethod.Post;
                request.RequestUri = new Uri(uri);

                if (!String.IsNullOrEmpty(requestBody))
                {
                    request.Content = new StringContent(requestBody, Encoding.UTF8, "text/json");
                }

                request.Headers.Add("Ocp-Apim-Subscription-Key", authoringKey);
                return await client.SendAsync(request);
            }
        }

        // Add utterances as string with POST request
        async static Task AddUtterances(string utterances)
        {
            string uri = host + "examples";

            var response = await SendPost(uri, utterances);
            var result = await response.Content.ReadAsStringAsync();
            Console.WriteLine("Added utterances.");
            Console.WriteLine(JsonFormatter.Format(result));
        }

        // Train app after adding utterances
        async static Task Train()
        {
            string uri = host  + "train";

            var response = await SendPost(uri, null);
            var result = await response.Content.ReadAsStringAsync();
            Console.WriteLine("Sent training request.");
            Console.WriteLine(JsonFormatter.Format(result));
        }

        // Check status of training
        async static Task Status()
        {
            var response = await SendGet(host  + "train");
            var result = await response.Content.ReadAsStringAsync();
            Console.WriteLine("Requested training status.");
            Console.WriteLine(JsonFormatter.Format(result));
        }

        // Add utterances, train, check status
        static void Main(string[] args)
        {
            string utterances = @"
            [
                {
                    'text': 'order a pizza',
                    'intentName': 'ModifyOrder',
                    'entityLabels': [
                        {
                            'entityName': 'Order',
                            'startCharIndex': 6,
                            'endCharIndex': 12
                        }
                    ]
                },
                {
                    'text': 'order a large pepperoni pizza',
                    'intentName': 'ModifyOrder',
                    'entityLabels': [
                        {
                            'entityName': 'Order',
                            'startCharIndex': 6,
                            'endCharIndex': 28
                        },
                        {
                            'entityName': 'FullPizzaWithModifiers',
                            'startCharIndex': 6,
                            'endCharIndex': 28
                        },
                        {
                            'entityName': 'PizzaType',
                            'startCharIndex': 14,
                            'endCharIndex': 28
                        },
                        {
                            'entityName': 'Size',
                            'startCharIndex': 8,
                            'endCharIndex': 12
                        }
                    ]
                },
                {
                    'text': 'I want two large pepperoni pizzas on thin crust',
                    'intentName': 'ModifyOrder',
                    'entityLabels': [
                        {
                            'entityName': 'Order',
                            'startCharIndex': 7,
                            'endCharIndex': 46
                        },
                        {
                            'entityName': 'FullPizzaWithModifiers',
                            'startCharIndex': 7,
                            'endCharIndex': 46
                        },
                        {
                            'entityName': 'PizzaType',
                            'startCharIndex': 17,
                            'endCharIndex': 32
                        },
                        {
                            'entityName': 'Size',
                            'startCharIndex': 11,
                            'endCharIndex': 15
                        },
                        {
                            'entityName': 'Quantity',
                            'startCharIndex': 7,
                            'endCharIndex': 9
                        },
                        {
                            'entityName': 'Crust',
                            'startCharIndex': 37,
                            'endCharIndex': 46
                        }
                    ]
                }
            ]
            ";

            AddUtterances(utterances).Wait();
            Train().Wait();
            Status().Wait();
        }
    }
}
```

1. 将以 `YOUR-` 开头的值替换为你自己的值。

    |信息|目的|
    |--|--|
    |`YOUR-APP-ID`| LUIS 应用 ID。 |
    |`YOUR-AUTHORING-KEY`|32 字符创作密钥。|
    |`YOUR-AUTHORING-ENDPOINT`| 创作 URL 终结点。 例如，`https://replace-with-your-resource-name.api.cognitive.azure.cn/`。 在创建资源时设置资源名称。|

    分配的密钥和资源可以在 LUIS 门户的“Azure 资源”页上的“管理”部分中看到。 应用 ID 可以在“应用程序设置”页的同一“管理”部分中找到。

1. 生成控制台应用程序。

    ```console
    dotnet build
    ```

1. 运行控制台应用程序。

    ```console
    dotnet run
    ```

1. 查看创作响应：

    ```console
    Added utterances.
    [
        {
            "value": {
                "ExampleId": 1137150691,
                "UtteranceText": "order a pizza"
            },
            "hasError": false
        },
        {
            "value": {
                "ExampleId": 1137150692,
                "UtteranceText": "order a large pepperoni pizza"
            },
            "hasError": false
        },
        {
            "value": {
                "ExampleId": 1137150693,
                "UtteranceText": "i want two large pepperoni pizzas on thin crust"
            },
            "hasError": false
        }
    ]
    Sent training request.
    {
        "statusId": 9,
        "status": "Queued"
    }
    Requested training status.
    [
        {
            "modelId": "edb46abf-0000-41ab-beb2-a41a0fe1630f",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "a5030be2-616c-4648-bf2f-380fa9417d37",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "3f2b1f31-a3c3-4fbd-8182-e9d9dbc120b9",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "e4b6704b-1636-474c-9459-fe9ccbeba51c",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "031d3777-2a00-4a7a-9323-9a3280a30000",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "9250e7a1-06eb-4413-9432-ae132ed32583",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        }
    ]
    ```

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，请从文件系统中删除项目文件夹。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用的最佳实践](../luis-concept-best-practices.md)

