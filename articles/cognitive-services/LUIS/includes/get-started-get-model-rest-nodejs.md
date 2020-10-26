---
title: 使用 Node.js 通过 REST 调用获取模型
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 10/19/2020
ms.author: v-johya
ms.custom: devx-track-js
ms.openlocfilehash: 36c6b8ce4203722204a2cd00544cf43a8a40e4e8
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472515"
---
[参考文档](https://dev.cognitive.azure.cn/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c45) | [示例](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/LUIS/node-model-with-rest/model.js)

## <a name="prerequisites"></a>先决条件

* [Node.js](https://nodejs.org/) 编程语言
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="example-utterances-json-file"></a>示例话语 JSON 文件

[!INCLUDE [Quickstart explanation of example utterance JSON file](get-started-get-model-json-example-utterances.md)]

## <a name="create-the-nodejs-project"></a>创建 Node.js 项目

1. 创建一个新文件夹以保存 Node.js 项目，例如 `node-model-with-rest`。

1. 打开新的命令提示符，导航到你创建的文件夹，并执行以下命令：

    ```console
    npm init
    ```

    在每个提示符下按 Enter 以接受默认设置。

1. 输入以下命令安装“请求-承诺”模块：

    ```console
    npm install --save request
    npm install --save request-promise
    npm install --save querystring
    ```

## <a name="change-model-programmatically"></a>以编程方式更改模型

1. 创建名为 `model.js` 的新文件。 添加以下代码：

```javascript
//
// This quickstart shows how to add utterances to a LUIS model using the REST APIs.
//

var request = require('request-promise');

//////////
// Values to modify.

// YOUR-APP-ID: The App ID GUID found on the luis.azure.cn Application Settings page.
const LUIS_appId = "YOUR-APP-ID";

// YOUR-AUTHORING-KEY: Your LUIS authoring key, 32 character value.
const LUIS_authoringKey = "YOUR-AUTHORING-KEY";

// YOUR-AUTHORING-ENDPOINT: Replace this with your authoring key endpoint.
// For example, "https://your-resource-name.cognitiveservices.azure.cn/"
const LUIS_endpoint = "https://YOUR-AUTHORING-ENDPOINT/";

// NOTE: Replace this your version number. The Pizza app uses a version number of "0.1".
const LUIS_versionId = "0.1";
//////////

const addUtterancesURI = `${LUIS_endpoint}luis/authoring/v3.0-preview/apps/${LUIS_appId}/versions/${LUIS_versionId}/examples`;
const addTrainURI = `${LUIS_endpoint}luis/authoring/v3.0-preview/apps/${LUIS_appId}/versions/${LUIS_versionId}/train`;

const utterances = [
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
];

// Main function.
const main = async() =>{

    await addUtterances(utterances);
    await train("POST");
    await train("GET");

}

// Adds the utterances to the model.
const addUtterances = async (utterances) => {

    const options = {
        uri: addUtterancesURI,
        method: 'POST',
        headers: {
            'Ocp-Apim-Subscription-Key': LUIS_authoringKey
        },
        json: true,
        body: utterances
    };

    const response = await request(options)
    console.log("addUtterance:\n" + JSON.stringify(response, null, 2));
}

// With verb === "POST", sends a training request.
// With verb === "GET", obtains the training status.
const train = async (verb) => {

    const options = {
        uri: addTrainURI,
        method: verb,
        headers: {
            'Ocp-Apim-Subscription-Key': LUIS_authoringKey
        },
        json: true,
        body: null // The body can be empty for a training request
    };

    const response = await request(options)
    console.log("train " + verb + ":\n" + JSON.stringify(response, null, 2));
}

// MAIN
main().then(() => console.log("done")).catch((err)=> console.log(err));
```

1. 将以 `YOUR-` 开头的值替换为你自己的值。

    |信息|目的|
    |--|--|
    |`YOUR-APP-ID`| LUIS 应用 ID。 |
    |`YOUR-AUTHORING-KEY`|32 字符创作密钥。|
    |`YOUR-AUTHORING-ENDPOINT`| 创作 URL 终结点。 例如，`https://replace-with-your-resource-name.api.cognitive.azure.cn/`。 在创建资源时设置资源名称。|

    分配的密钥和资源可以在 LUIS 门户的“Azure 资源”页上的“管理”部分中看到。 应用 ID 可以在“应用程序设置”页的同一“管理”部分中找到。

1. 在命令提示符处，输入下列命令以运行项目：

    ```console
    node model.js
    ```

1. 查看创作响应：

    ```json
    addUtterance:
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
    train POST:
    {
      "statusId": 9,
      "status": "Queued"
    }
    train GET:
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
    done
    ```

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，请从文件系统中删除项目文件夹。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [应用的最佳实践](../luis-concept-best-practices.md)

