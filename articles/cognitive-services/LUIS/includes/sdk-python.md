---
title: include 文件
description: include 文件
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 10/23/2020
ms.topic: include
ms.custom: include file, cog-serv-seo-aug-2020
ms.openlocfilehash: e1c796b343d5c38c4f6a584a69c5e5764ed142cb
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472970"
---
使用适用于 Python 的语言理解 (LUIS) 客户端库来执行以下操作：

* 创建应用
* 使用示例言语添加意向（机器学习实体）
* 训练和发布应用
* 查询预测运行时

[参考文档](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/index?view=azure-python) | [创作](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-language-luis/azure/cognitiveservices/language/luis/authoring)和[预测](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-language-luis/azure/cognitiveservices/language/luis/runtime)库源代码 | [包 (Pypi)](https://pypi.org/project/azure-cognitiveservices-language-luis/) | [示例](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/LUIS/sdk-3x/authoring_and_predict.py)

## <a name="prerequisites"></a>先决条件

* 最新版本的 [Python 3.x](https://www.python.org/)。
* Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/details/cognitive-services)
* 有了 Azure 订阅后，在 Azure 门户中[创建语言理解创作资源](https://portal.azure.cn/#create/Microsoft.CognitiveServicesLUISAllInOne)，以获取创作密钥和终结点。 等待其部署并单击“转到资源”按钮。
    * 需要从[创建](../luis-how-to-azure-subscription.md#create-luis-resources-in-azure-portal)的资源获取密钥和终结点，以便将应用程序连接到语言理解创作。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。 可以使用免费定价层 (`F0`) 来试用该服务。

## <a name="setting-up"></a>设置

### <a name="create-a-new-python-application"></a>创建新的 Python 应用程序

1. 在控制台窗口中，为应用程序创建一个新目录，并将其移动到该目录中。

    ```console
    mkdir quickstart-sdk && cd quickstart-sdk
    ```

1. 为 Python 代码创建名为 `authoring_and_predict.py` 的文件。

    ```console
    touch authoring_and_predict.py
    ```

### <a name="install-the-client-library-with-pip"></a>使用 Pip 安装客户端库

在应用程序目录中，使用以下命令安装适用于 python 的语言理解 (LUIS) 客户端库：

```console
pip install azure-cognitiveservices-language-luis
```

## <a name="authoring-object-model"></a>创作对象模型

语言理解 (LUIS) 创作客户端是对 Azure 进行身份验证的 [LUISAuthoringClient](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.luisauthoringclient?view=azure-python) 对象，其中包含创作密钥。

## <a name="code-examples-for-authoring"></a>创作的代码示例

创建客户端后，可以使用此客户端访问如下所述的功能：

* 应用 - [创建](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.appsoperations?view=azure-python#add-application-create-object--custom-headers-none--raw-false----operation-config-)、[删除](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.appsoperations?view=azure-python#delete-app-id--force-false--custom-headers-none--raw-false----operation-config-)、[发布](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.appsoperations?view=azure-python#publish-app-id--version-id-none--is-staging-false--custom-headers-none--raw-false----operation-config-)
* 示例言语 - [按批添加](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.examplesoperations?view=azure-python#batch-app-id--version-id--example-label-object-array--custom-headers-none--raw-false----operation-config-)、[按 ID 删除](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.examplesoperations?view=azure-python#delete-app-id--version-id--example-id--custom-headers-none--raw-false----operation-config-)
* 特征 - 管理[短语列表](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.featuresoperations?view=azure-python#add-phrase-list-app-id--version-id--phraselist-create-object--custom-headers-none--raw-false----operation-config-)
* 模型 - 管理[意向](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.modeloperations?view=azure-python#add-intent-app-id--version-id--name-none--custom-headers-none--raw-false----operation-config-)和实体
* 模式 - 管理[模式](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.patternoperations?view=azure-python#add-pattern-app-id--version-id--pattern-none--intent-none--custom-headers-none--raw-false----operation-config-)
* 训练 - [训练](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.trainoperations?view=azure-python#train-version-app-id--version-id--custom-headers-none--raw-false----operation-config-)应用和轮询[训练状态](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.trainoperations?view=azure-python#get-status-app-id--version-id--custom-headers-none--raw-false----operation-config-)
* [版本](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.versionsoperations?view=azure-python) - 使用克隆、导出和删除操作进行管理


## <a name="prediction-object-model"></a>预测对象模型

语言理解 (LUIS) 预测运行时客户端是对 Azure 进行身份验证的 [LUISRuntimeClient](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.runtime.luisruntimeclient?view=azure-python) 对象，其中包含资源密钥。

## <a name="code-examples-for-prediction-runtime"></a>预测运行时的代码示例

创建客户端后，可以使用此客户端访问如下所述的功能：

* 按[过渡槽或生产槽](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.runtime.operations.predictionoperations?view=azure-python#get-slot-prediction-app-id--slot-name--prediction-request--verbose-none--show-all-intents-none--log-none--custom-headers-none--raw-false----operation-config-)进行预测
* 按[版本](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.runtime.operations.predictionoperations?view=azure-python#get-version-prediction-app-id--version-id--prediction-request--verbose-none--show-all-intents-none--log-none--custom-headers-none--raw-false----operation-config-)进行预测

[!INCLUDE [Bookmark links to same article](sdk-code-examples.md)]

## <a name="add-the-dependencies"></a>添加依赖项

将客户端库添加到 python 文件中。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```


## <a name="add-boilerplate-code"></a>添加样板代码

1. 添加 `quickstart` 方法及其调用。 此方法保存大部分剩余代码。 在文件末尾调用此方法。

    ```python
    def quickstart():

        # add calls here, remember to indent properly

    quickstart()
    ```

1. 在 quickstart 方法中添加剩余代码（除非另有指定）。

## <a name="create-variables-for-the-app"></a>为应用创建变量

创建两组变量：第一组为你更改的变量，第二组保留在代码示例中显示的状态。 

1. 创建用于保存创作密钥和资源名称的变量。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

1. 创建用于保存终结点、应用名称、版本和意向名称的变量。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

## <a name="authenticate-the-client"></a>验证客户端

使用密钥创建 [CognitiveServicesCredentials](https://docs.microsoft.com/python/api/msrest/msrest.authentication.cognitiveservicescredentials?view=azure-python) 对象，并在终结点中使用该对象创建一个 [LUISAuthoringClient](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.luisauthoringclient?view=azure-python) 对象。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

## <a name="create-a-luis-app"></a>创建 LUIS 应用

LUIS 应用包含自然语言处理 (NLP) 模型，包括意向、实体和示例言语。

创建 [AppsOperation](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.appsoperations?view=azure-python) 对象的 [add](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.appsoperations?view=azure-python#add-application-create-object--custom-headers-none--raw-false----operation-config-) 方法，用于创建应用。 名称和语言区域性是必需的属性。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```


## <a name="create-intent-for-the-app"></a>为应用创建意向
LUIS 应用模型中的主要对象是意向。 意向与用户言语意向的分组相符。 用户可以提问，或者做出表述，指出希望机器人（或其他客户端应用程序）提供特定的有针对性响应。 意向的示例包括预订航班、询问目的地城市的天气，以及询问客户服务的联系信息。

将 [model.add_intent](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.modeloperations?view=azure-python#add-intent-app-id--version-id--name-none--custom-headers-none--raw-false----operation-config-) 方法与唯一意向的名称配合使用，然后传递应用 ID、版本 ID 和新的意向名称。

`intentName` 值硬编码为 `OrderPizzaIntent`，作为[为应用创建变量](#create-variables-for-the-app)中变量的一部分。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

## <a name="create-entities-for-the-app"></a>为应用创建实体

尽管实体不是必需的，但在大多数应用中都可以看到实体。 实体从用户言语中提取信息，只有使用这些信息才能实现用户的意向。 有多种类型的[预生成](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.modeloperations?view=azure-python#add-prebuilt-app-id--version-id--prebuilt-extractor-names--custom-headers-none--raw-false----operation-config-)实体和自定义实体，每种实体具有自身的数据转换对象 (DTO) 模型。  在应用中添加的常见预生成实体包括 [number](../luis-reference-prebuilt-number.md)、[datetimeV2](../luis-reference-prebuilt-datetimev2.md)、[geographyV2](../luis-reference-prebuilt-geographyv2.md) 和 [ordinal](../luis-reference-prebuilt-ordinal.md)。

必须知道，实体不会使用意向进行标记。 它们可以并且通常应用到多个意向。 只会为特定的单个意向标记示例用户言语。

实体的创建方法属于 [ModelOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.modeloperations?view=azure-python) 类的一部分。 每个实体类型有自身的数据转换对象 (DTO) 模型。

实体创建代码会创建机器学习实体，其中包含子实体以及应用于 `Quantity` 子实体的功能。

:::image type="content" source="../media/quickstart-sdk/machine-learned-entity.png" alt-text="显示创建的实体的门户的部分屏幕截图，其中为包含子实体以及应用于 `Quantity` 子实体的功能的机器学习实体。":::

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

将以下方法置于 `quickstart` 方法之上，查找 Quantity 子实体的 ID，以便将功能分配给该子实体。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

## <a name="add-example-utterance-to-intent"></a>将示例言语添加到意向

为了确定言语的意向并提取实体，应用需要言语示例。 这些示例需要针对特定的单个意向，并且应该标记所有自定义实体。 无需标记预生成实体。

通过创建 [ExampleLabelObject](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.models.examplelabelobject?view=azure-python) 对象的列表来添加示例言语（每个示例言语对应于一个对象）。 每个示例应使用实体名称和实体值的名称/值对字典来标记所有实体。 实体值应与示例言语文本中显示的值完全相同。

:::image type="content" source="../media/quickstart-sdk/labeled-example-machine-learned-entity.png" alt-text="显示创建的实体的门户的部分屏幕截图，其中为包含子实体以及应用于 `Quantity` 子实体的功能的机器学习实体。":::

结合应用 ID、版本 ID 和示例调用 [examples.add](https://docs.microsoft.com//python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.examplesoperations?view=azure-python#add-app-id--version-id--example-label-object--enable-nested-children-false--custom-headers-none--raw-false----operation-config-)。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

## <a name="train-the-app"></a>训练应用

创建模型后，需要为此模型版本训练 LUIS 应用。 训练后的模型可在[容器](../luis-container-howto.md)中使用，或者将其[发布](../luis-how-to-publish-app.md)到过渡槽或生产槽。

[train.train_version](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.trainoperations?view=azure-python#train-version-app-id--version-id--custom-headers-none--raw-false----operation-config-) 方法需要应用 ID 和版本 ID。

极小的模型（如本快速入门中所示的模型）很快就能完成训练。 对于生产级应用程序，应用的训练应该包括轮询调用 [get_status](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.trainoperations?view=azure-python#get-status-app-id--version-id--custom-headers-none--raw-false----operation-config-) 方法以确定训练何时成功或者是否成功。 响应是一个 [ModelTrainingInfo](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.models.modeltraininginfo?view=azure-python) 对象列表，其中分别列出了每个对象的状态。 所有对象必须成功，才能将训练视为完成。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

## <a name="publish-app-to-production-slot"></a>将应用发布到生产槽

使用 [app.publish](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.authoring.operations.appsoperations?view=azure-python#publish-app-id--version-id-none--is-staging-false--custom-headers-none--raw-false----operation-config-) 方法发布 LUIS 应用。 这会将当前已训练的版本发布到终结点上的指定槽。 客户端应用程序使用此终结点发送用户言语，以预测意向和提取实体。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

## <a name="authenticate-the-prediction-runtime-client"></a>对预测运行时客户端进行身份验证

将凭据对象与密钥一起使用，并在终结点中使用该对象创建一个 [LUISRuntimeClientConfiguration](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.runtime.luisruntimeclientconfiguration?view=azure-python) 对象。

[!INCLUDE [Caution about using authoring key](caution-authoring-key.md)]

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

## <a name="get-prediction-from-runtime"></a>从运行时获取预测

添加以下代码以创建对预测运行时的请求。

用户言语是 [prediction_request](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.runtime.models.predictionrequest?view=azure-python) 对象的一部分。

**[get_slot_prediction](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.runtime.operations.predictionoperations?view=azure-python#get-slot-prediction-app-id--slot-name--prediction-request--verbose-none--show-all-intents-none--log-none--custom-headers-none--raw-false----operation-config-)** 方法需要多个参数，如应用 ID、槽名称，以及用于履行请求的预测请求对象。 其他选项（如详细、显示所有意向和日志）都是可选的。 该请求返回 [PredictionResponse](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-luis/azure.cognitiveservices.language.luis.runtime.models.predictionresponse?view=azure-python) 对象。

```python
# To run this sample, install the following modules.
# pip install azure-cognitiveservices-language-luis

# <Dependencies>
from azure.cognitiveservices.language.luis.authoring import LUISAuthoringClient
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
from functools import reduce

import json, time
# </Dependencies>

def quickstart(): 

    # <VariablesYouChange>
    authoringKey = 'REPLACE-WITH-YOUR-ASSIGNED-AUTHORING-KEY'
    authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME"
    predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME"
    # </VariablesYouChange>

    # <VariablesYouDontNeedToChangeChange>
    authoringEndpoint = f'https://{authoringResourceName}.cognitiveservices.azure.cn/'
    predictionEndpoint = f'https://{predictionResourceName}.cognitiveservices.azure.cn/'

    appName = "Contoso Pizza Company"
    versionId = "0.1"
    intentName = "OrderPizzaIntent"
    # </VariablesYouDontNeedToChangeChange>

    # <AuthoringCreateClient>
    client = LUISAuthoringClient(authoringEndpoint, CognitiveServicesCredentials(authoringKey))
    # </AuthoringCreateClient>

    # Create app
    app_id = create_app(client, appName, versionId)

    # <AddIntent>
    client.model.add_intent(app_id, versionId, intentName)
    # </AddIntent>
    
    # Add Entities
    add_entities(client, app_id, versionId)
    
    # Add labeled examples
    add_labeled_examples(client,app_id, versionId, intentName)

    # <TrainAppVersion>
    client.train.train_version(app_id, versionId)
    waiting = True
    while waiting:
        info = client.train.get_status(app_id, versionId)

        # get_status returns a list of training statuses, one for each model. Loop through them and make sure all are done.
        waiting = any(map(lambda x: 'Queued' == x.details.status or 'InProgress' == x.details.status, info))
        if waiting:
            print ("Waiting 10 seconds for training to complete...")
            time.sleep(10)
        else: 
            print ("trained")
            waiting = False
    # </TrainAppVersion>
    
    # <PublishVersion>
    responseEndpointInfo = client.apps.publish(app_id, versionId, is_staging=False)
    # </PublishVersion>
    
    # <PredictionCreateClient>
    runtimeCredentials = CognitiveServicesCredentials(authoringKey)
    clientRuntime = LUISRuntimeClient(endpoint=predictionEndpoint, credentials=runtimeCredentials)
    # </PredictionCreateClient>

    # <QueryPredictionEndpoint>
    # Production == slot name
    predictionRequest = { "query" : "I want two small pepperoni pizzas with more salsa" }
    
    predictionResponse = clientRuntime.prediction.get_slot_prediction(app_id, "Production", predictionRequest)
    print("Top intent: {}".format(predictionResponse.prediction.top_intent))
    print("Sentiment: {}".format (predictionResponse.prediction.sentiment))
    print("Intents: ")

    for intent in predictionResponse.prediction.intents:
        print("\t{}".format (json.dumps (intent)))
    print("Entities: {}".format (predictionResponse.prediction.entities))
    # </QueryPredictionEndpoint>

def create_app(client, appName, versionId):

    # <AuthoringCreateApplication>
    # define app basics
    appDefinition = {
        "name": appName,
        "initial_version_id": versionId,
        "culture": "en-us"
    }

    # create app
    app_id = client.apps.add(appDefinition)

    # get app id - necessary for all other changes
    print("Created LUIS app with ID {}".format(app_id))
    # </AuthoringCreateApplication>
    
    return app_id
    
# </createApp>

def add_entities(client, app_id, versionId):

    # <AuthoringAddEntities>
    # Add Prebuilt entity
    client.model.add_prebuilt(app_id, versionId, prebuilt_extractor_names=["number"])

    # define machine-learned entity
    mlEntityDefinition = [
    {
        "name": "Pizza",
        "children": [
            { "name": "Quantity" },
            { "name": "Type" },
            { "name": "Size" }
        ]
    },
    {
        "name": "Toppings",
        "children": [
            { "name": "Type" },
            { "name": "Quantity" }
        ]
    }]

    # add entity to app
    modelId = client.model.add_entity(app_id, versionId, name="Pizza order", children=mlEntityDefinition)
    
    # define phraselist - add phrases as significant vocabulary to app
    phraseList = {
        "enabledForAllModels": False,
        "isExchangeable": True,
        "name": "QuantityPhraselist",
        "phrases": "few,more,extra"
    }
    
    # add phrase list to app
    phraseListId = client.features.add_phrase_list(app_id, versionId, phraseList)
    
    # Get entity and subentities
    modelObject = client.model.get_entity(app_id, versionId, modelId)
    toppingQuantityId = get_grandchild_id(modelObject, "Toppings", "Quantity")
    pizzaQuantityId = get_grandchild_id(modelObject, "Pizza", "Quantity")

    # add model as feature to subentity model
    prebuiltFeatureRequiredDefinition = { "model_name": "number", "is_required": True }
    client.features.add_entity_feature(app_id, versionId, pizzaQuantityId, prebuiltFeatureRequiredDefinition)
    
    # add model as feature to subentity model
    prebuiltFeatureNotRequiredDefinition = { "model_name": "number" }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, prebuiltFeatureNotRequiredDefinition)

    # add phrase list as feature to subentity model
    phraseListFeatureDefinition = { "feature_name": "QuantityPhraselist", "model_name": None }
    client.features.add_entity_feature(app_id, versionId, toppingQuantityId, phraseListFeatureDefinition)
    # </AuthoringAddEntities>
    

def add_labeled_examples(client, app_id, versionId, intentName):

    # <AuthoringAddLabeledExamples>
    # Define labeled example
    labeledExampleUtteranceWithMLEntity = {
        "text": "I want two small seafood pizzas with extra cheese.",
        "intentName": intentName,
        "entityLabels": [
            {
                "startCharIndex": 7,
                "endCharIndex": 48,
                "entityName": "Pizza order",
                "children": [
                    {
                        "startCharIndex": 7,
                        "endCharIndex": 30,
                        "entityName": "Pizza",
                        "children": [
                            {
                                "startCharIndex": 7,
                                "endCharIndex": 9,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 11,
                                "endCharIndex": 15,
                                "entityName": "Size"
                            },
                            {
                                "startCharIndex": 17,
                                "endCharIndex": 23,
                                "entityName": "Type"
                            }]
                    },
                    {
                        "startCharIndex": 37,
                        "endCharIndex": 48,
                        "entityName": "Toppings",
                        "children": [
                            {
                                "startCharIndex": 37,
                                "endCharIndex": 41,
                                "entityName": "Quantity"
                            },
                            {
                                "startCharIndex": 43,
                                "endCharIndex": 48,
                                "entityName": "Type"
                            }]
                    }
                ]
            }
        ]
    }

    print("Labeled Example Utterance:", labeledExampleUtteranceWithMLEntity)

    # Add an example for the entity.
    # Enable nested children to allow using multiple models with the same name.
    # The quantity subentity and the phraselist could have the same exact name if this is set to True
    client.examples.add(app_id, versionId, labeledExampleUtteranceWithMLEntity, { "enableNestedChildren": True })
    # </AuthoringAddLabeledExamples>
    
# <AuthoringSortModelObject>
def get_grandchild_id(model, childName, grandChildName):
    
    theseChildren = next(filter((lambda child: child.name == childName), model.children))
    theseGrandchildren = next(filter((lambda child: child.name == grandChildName), theseChildren.children))
    
    grandChildId = theseGrandchildren.id
    
    return grandChildId
# </AuthoringSortModelObject>

quickstart()
```

[!INCLUDE [Prediction JSON response](sdk-json.md)]

## <a name="run-the-application"></a>运行应用程序

在快速入门文件中使用 `python` 命令运行应用程序。

```console
python authoring_and_predict.py
```

