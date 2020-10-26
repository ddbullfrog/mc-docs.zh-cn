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
ms.openlocfilehash: 655d3643f9dd2ec3f1690019f84748ec1651a76b
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472519"
---
[参考文档](https://dev.cognitive.azure.cn/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c45) | [示例](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/LUIS/python-model-with-rest/model.py)

## <a name="prerequisites"></a>先决条件

* [Python 3.6](https://www.python.org/downloads/) 或更高版本。
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="example-utterances-json-file"></a>示例话语 JSON 文件

[!INCLUDE [Quickstart explanation of example utterance JSON file](get-started-get-model-json-example-utterances.md)]

## <a name="create-pizza-app"></a>创建披萨应用

[!INCLUDE [Create pizza app](get-started-get-model-create-pizza-app.md)]

## <a name="change-model-programmatically"></a>以编程方式更改模型

1. 创建名为 `model.py` 的新文件。 添加以下代码：

```python
########### Python 3.6 #############

#
# This quickstart shows how to add utterances to a LUIS model using the REST APIs.
#

import requests

try:

    ##########
    # Values to modify.

    # YOUR-APP-ID: The App ID GUID found on the luis.azure.cn Application Settings page.
    appId = "YOUR-APP-ID"

    # YOUR-AUTHORING-KEY: Your LUIS authoring key, 32 character value.
    authoring_key = "YOUR-AUTHORING-KEY"

    # YOUR-AUTHORING-ENDPOINT: Replace this with your authoring key endpoint.
    # For example, "https://your-resource-name.cognitiveservices.azure.cn/"
    authoring_endpoint = "https://YOUR-AUTHORING-ENDPOINT/"

    # The version number of your LUIS app
    app_version = "0.1"
    ##########

    # The headers to use in this REST call.
    headers = {'Ocp-Apim-Subscription-Key': authoring_key}

    # The URL parameters to use in this REST call.
    params ={
        #'timezoneOffset': '0',
        #'verbose': 'true',
        #'show-all-intents': 'true',
        #'spellCheck': 'false',
        #'staging': 'false'
    }

    # List of example utterances to send to the LUIS app.
    data = """[
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
"""


    # Make the REST call to POST the list of example utterances.
    response = requests.post(f'{authoring_endpoint}luis/authoring/v3.0-preview/apps/{appId}/versions/{app_version}/examples',
        headers=headers, params=params, data=data)

    # Display the results on the console.
    print('Add the list of utterances:')
    print(response.json())


    # Make the REST call to initiate a training session.
    response = requests.post(f'{authoring_endpoint}luis/authoring/v3.0-preview/apps/{appId}/versions/{app_version}/train',
        headers=headers, params=params, data=None)

    # Display the results on the console.
    print('Request training:')
    print(response.json())


    # Make the REST call to request the status of training.
    response = requests.get(f'{authoring_endpoint}luis/authoring/v3.0-preview/apps/{appId}/versions/{app_version}/train',
        headers=headers, params=params, data=None)

    # Display the results on the console.
    print('Request training status:')
    print(response.json())


except Exception as e:
    # Display the error string.
    print(f'{e}')
```

1. 将以 `YOUR-` 开头的值替换为你自己的值。

    |信息|目的|
    |--|--|
    |`YOUR-APP-ID`| LUIS 应用 ID。 |
    |`YOUR-AUTHORING-KEY`|32 字符创作密钥。|
    |`YOUR-AUTHORING-ENDPOINT`| 创作 URL 终结点。 例如，`https://replace-with-your-resource-name.api.cognitive.azure.cn/`。 在创建资源时设置资源名称。|

    分配的密钥和资源可以在 LUIS 门户的“Azure 资源”页上的“管理”部分中看到。 应用 ID 可以在“应用程序设置”页的同一“管理”部分中找到。

1. 在创建该文件的同一目录中，在命令提示符下输入以下命令来运行文件：

    ```console
    python model.py
    ```

1. 查看创作响应：

    ```console
    Add the list of utterances:
    [{'value': {'ExampleId': 1137150691, 'UtteranceText': 'order a pizza'}, 'hasError': False}, {'value': {'ExampleId': 1137150692, 'UtteranceText': 'order a large pepperoni pizza'}, 'hasError': False}, {'value': {'ExampleId': 1137150693, 'UtteranceText': 'i want two large pepperoni pizzas on thin crust'}, 'hasError': False}]
    Request training:
    {'statusId': 9, 'status': 'Queued'}
    Request training status:
    [{'modelId': 'edb46abf-0000-41ab-beb2-a41a0fe1630f', 'details': {'statusId': 3, 'status': 'InProgress', 'exampleCount': 0, 'progressSubstatus': 'CollectingData'}}, {'modelId': 'a5030be2-616c-4648-bf2f-380fa9417d37', 'details': {'statusId': 3, 'status': 'InProgress', 'exampleCount': 0, 'progressSubstatus': 'CollectingData'}}, {'modelId': '3f2b1f31-a3c3-4fbd-8182-e9d9dbc120b9', 'details': {'statusId': 3, 'status': 'InProgress', 'exampleCount': 0, 'progressSubstatus': 'CollectingData'}}, {'modelId': 'e4b6704b-1636-474c-9459-fe9ccbeba51c', 'details': {'statusId': 3, 'status': 'InProgress', 'exampleCount': 0, 'progressSubstatus': 'CollectingData'}}, {'modelId': '031d3777-2a00-4a7a-9323-9a3280a30000', 'details': {'statusId': 3, 'status': 'InProgress', 'exampleCount': 0, 'progressSubstatus': 'CollectingData'}}, {'modelId': '9250e7a1-06eb-4413-9432-ae132ed32583', 'details': {'statusId': 3, 'status': 'InProgress', 'exampleCount': 0, 'progressSubstatus': 'CollectingData'}}]
    ```

    下面是为提高可读性而进行了格式设置的输出：

    ```json
    Add the list of utterances:
    [
      {
        'value': {
          'ExampleId': 1137150691,
          'UtteranceText': 'order a pizza'
        },
        'hasError': False
      },
      {
        'value': {
          'ExampleId': 1137150692,
          'UtteranceText': 'order a large pepperoni pizza'
        },
        'hasError': False
      },
      {
        'value': {
          'ExampleId': 1137150693,
          'UtteranceText': 'i want two large pepperoni pizzas on thin crust'
        },
        'hasError': False
      }
    ]

    Request training:
    {
      'statusId': 9,
      'status': 'Queued'
    }

    Request training status:
    [
      {
        'modelId': 'edb46abf-0000-41ab-beb2-a41a0fe1630f',
        'details': {
          'statusId': 3,
          'status': 'InProgress',
          'exampleCount': 0,
          'progressSubstatus': 'CollectingData'
        }
      },
      {
        'modelId': 'a5030be2-616c-4648-bf2f-380fa9417d37',
        'details': {
          'statusId': 3,
          'status': 'InProgress',
          'exampleCount': 0,
          'progressSubstatus': 'CollectingData'
        }
      },
      {
        'modelId': '3f2b1f31-a3c3-4fbd-8182-e9d9dbc120b9',
        'details': {
          'statusId': 3,
          'status': 'InProgress',
          'exampleCount': 0,
          'progressSubstatus': 'CollectingData'
        }
      },
      {
        'modelId': 'e4b6704b-1636-474c-9459-fe9ccbeba51c',
        'details': {
          'statusId': 3,
          'status': 'InProgress',
          'exampleCount': 0,
          'progressSubstatus': 'CollectingData'
        }
      },
      {
        'modelId': '031d3777-2a00-4a7a-9323-9a3280a30000',
        'details': {
          'statusId': 3,
          'status': 'InProgress',
          'exampleCount': 0,
          'progressSubstatus': 'CollectingData'
        }
      },
      {
        'modelId': '9250e7a1-06eb-4413-9432-ae132ed32583',
        'details': {
          'statusId': 3,
          'status': 'InProgress',
          'exampleCount': 0,
          'progressSubstatus': 'CollectingData'
        }
      }
    ]
    ```

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，请从文件系统中删除该文件。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用的最佳实践](../luis-concept-best-practices.md)

