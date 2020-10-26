---
title: 使用 Node.js 导入话语 - LUIS
titleSuffix: Azure Cognitive Services
description: 了解如何使用 LUIS Authoring API 以编程方式从 CSV 格式的预先存在数据生成 LUIS 应用。
services: cognitive-services
manager: nitinme
ms.custom: seodec18, devx-track-js
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
origin.date: 09/05/2019
ms.date: 10/19/2020
ms.author: v-johya
ms.openlocfilehash: 6e9e3b3cf97763bb56a11f71da06ef25d4af9f67
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472348"
---
# <a name="build-a-luis-app-programmatically-using-nodejs"></a>使用 Node.js 以编程方式生成 LUIS 应用

LUIS 提供与 [LUIS](luis-reference-regions.md) 网站功能相同的编程 API。 如果有预先存在的数据，这样可以节省时间，而且以编程方式创建 LUIS 应用比手动输入信息快。

[!INCLUDE [Waiting for LUIS portal refresh](./includes/wait-v3-upgrade.md)]

## <a name="prerequisites"></a>必备条件

* 登录 [LUIS](luis-reference-regions.md) 网站，并在“帐户设置”中找到[创作密钥](luis-how-to-azure-subscription.md#authoring-key)。 使用此密钥调用 Authoring API。
* 如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/details/cognitive-services/)。
* 本文从用户请求的一家虚拟公司的 CSV 格式日志文件开始。 可从[此处](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/IoT.csv)下载。
* 使用 NPM 安装最新的 Node.js。 从[此处](https://nodejs.org/en/download/)下载它。
* **[建议]** 用于 IntelliSense 和调试的 Visual Studio Code 可从 [此处](https://code.visualstudio.com/)免费下载。

本文中的所有代码都可在 [Azure-Samples 语言理解 GitHub 存储库](https://github.com/Azure-Samples/cognitive-services-language-understanding/tree/master/examples/build-app-programmatically-csv)中找到。

## <a name="map-preexisting-data-to-intents-and-entities"></a>将预先存在的数据映射到意向和实体
即使系统在创建时未考虑使用 LUIS，如果它包含映射到用户不同操作的文本数据，也许能够从现有用户输入类别映射到 LUIS 中的意向。 如果可标识用户所说的重要单词或短语，这些单词可能会映射到实体。

打开 [`IoT.csv`](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/IoT.csv) 文件。 它包含对虚构家庭自动化服务的用户查询日志，包括分类方式、用户所说的内容以及一些包含有用信息的列。

![预先存在数据的 CSV 文件](./media/luis-tutorial-node-import-utterances-csv/csv.png)

可以看到“RequestType”列可能是意向，“Request”列显示了一个示例陈述  。 如果其他字段出现在陈述中，则可能是实体。 由于有意向、实体和示例陈述，因此需要一个简单的示例应用。

## <a name="steps-to-generate-a-luis-app-from-non-luis-data"></a>从非 LUIS 数据生成 LUIS 应用的步骤
从 CSV 文件生成新的 LUIS 应用：

* 分析 CSV 文件中的数据：
    * 转换为可以使用创作 API 上传到 LUIS 的格式。
    * 从已分析的数据收集有关意向和实体的信息。
* 调用创作 API 以执行以下操作：
    * 创建应用。
    * 添加从已分析数据中收集的意向和实体。
    * 创建 LUIS 应用后，可以从已分析的数据中添加示例陈述。

可以在 `index.js` 文件的最后一部分中看到此程序流。 复制或[下载](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/index.js)此代码并将其保存在 `index.js`。

```javascript
var path = require('path');

const parse = require('./_parse');
const createApp = require('./_create');
const addEntities = require('./_entities');
const addIntents = require('./_intents');
const upload = require('./_upload');

// Change these values
const LUIS_authoringKey = "YOUR_AUTHORING_KEY";
const LUIS_appName = "Sample App - build from IoT csv file";
const LUIS_appCulture = "en-us"; 
const LUIS_versionId = "0.1";

// NOTE: final output of add-utterances api named utterances.upload.json
const downloadFile = "./IoT.csv";
const uploadFile = "./utterances.json"

// The app ID is returned from LUIS when your app is created
var LUIS_appId = ""; // default app ID
var intents = [];
var entities = [];


/* add utterances parameters */
var configAddUtterances = {
    LUIS_subscriptionKey: LUIS_authoringKey,
    LUIS_appId: LUIS_appId,
    LUIS_versionId: LUIS_versionId,
    inFile: path.join(__dirname, uploadFile),
    batchSize: 100,
    uri: "https://api.cognitive.azure.cn/luis/api/v2.0/apps/{appId}/versions/{versionId}/examples"
};

/* create app parameters */
var configCreateApp = {
    LUIS_subscriptionKey: LUIS_authoringKey,
    LUIS_versionId: LUIS_versionId,
    appName: LUIS_appName,
    culture: LUIS_appCulture,
    uri: "https://api.cognitive.azure.cn/luis/api/v2.0/apps/"
};

/* add intents parameters */
var configAddIntents = {
    LUIS_subscriptionKey: LUIS_authoringKey,
    LUIS_appId: LUIS_appId,
    LUIS_versionId: LUIS_versionId,
    intentList: intents,
    uri: "https://api.cognitive.azure.cn/luis/api/v2.0/apps/{appId}/versions/{versionId}/intents"
};

/* add entities parameters */
var configAddEntities = {
    LUIS_subscriptionKey: LUIS_authoringKey,
    LUIS_appId: LUIS_appId,
    LUIS_versionId: LUIS_versionId,
    entityList: entities,
    uri: "https://api.cognitive.azure.cn/luis/api/v2.0/apps/{appId}/versions/{versionId}/entities"
};

/* input and output files for parsing CSV */
var configParse = {
    inFile: path.join(__dirname, downloadFile),
    outFile: path.join(__dirname, uploadFile)
};

// Parse CSV
parse(configParse)
    .then((model) => {
        // Save intent and entity names from parse
        intents = model.intents;
        entities = model.entities;
        // Create the LUIS app
        return createApp(configCreateApp);

    }).then((appId) => {
        // Add intents
        LUIS_appId = appId;
        configAddIntents.LUIS_appId = appId;
        configAddIntents.intentList = intents;
        return addIntents(configAddIntents);

    }).then(() => {
        // Add entities
        configAddEntities.LUIS_appId = LUIS_appId;
        configAddEntities.entityList = entities;
        return addEntities(configAddEntities);

    }).then(() => {
        // Add example utterances to the intents in the app
        configAddUtterances.LUIS_appId = LUIS_appId;
        return upload(configAddUtterances);

    }).catch(err => {
        console.log(err.message);
    });
```


## <a name="parse-the-csv"></a>分析 CSV

包含 CSV 中的陈述的列条目必须被分析为 LUIS 可以理解的 JSON 格式。 此 JSON 格式必须包含一个 `intentName` 字段，该字段标识陈述的意向。 它还必须包含一个 `entityLabels` 字段，如果陈述中没有实体，该字段可以是空的。

例如，“打开灯”条目映射到此 JSON：

```json
        {
            "text": "Turn on the lights",
            "intentName": "TurnOn",
            "entityLabels": [
                {
                    "entityName": "Operation",
                    "startCharIndex": 5,
                    "endCharIndex": 6
                },
                {
                    "entityName": "Device",
                    "startCharIndex": 12,
                    "endCharIndex": 17
                }
            ]
        }
```

在此示例中，`intentName` 来自 CSV 文件中“Request”列标题下的用户请求，`entityName` 来自具有密钥信息的其他列  。 例如，如果有“操作”或“设备”条目，并且该字符串也出现在实际请求中，则可将其标记为实体  。 下面的代码演示此分析过程。 可以复制或[下载](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_parse.js)它并将其保存在 `_parse.js`。

```javascript
// node 7.x
// built with streams for larger files

const fse = require('fs-extra');
const path = require('path');
const lineReader = require('line-reader');
const babyparse = require('babyparse');
const Promise = require('bluebird');

const intent_column = 0;
const utterance_column = 1;
var entityNames = [];

var eachLine = Promise.promisify(lineReader.eachLine);

function listOfIntents(intents) {
    return intents.reduce(function (a, d) {
        if (a.indexOf(d.intentName) === -1) {
            a.push(d.intentName);
        }
        return a;
    }, []);

}

function listOfEntities(utterances) {
    return utterances.reduce(function (a, d) {        
        d.entityLabels.forEach(function(entityLabel) {
            if (a.indexOf(entityLabel.entityName) === -1) {
                a.push(entityLabel.entityName);
            }     
        }, this);
        return a;
    }, []);
}

var utterance = function (rowAsString) {

    let json = {
        "text": "",
        "intentName": "",
        "entityLabels": [],
    };

    if (!rowAsString) return json;

    let dataRow = babyparse.parse(rowAsString);
    // Get intent name and utterance text 
    json.intentName = dataRow.data[0][intent_column];
    json.text = dataRow.data[0][utterance_column];
    // For each column heading that may be an entity, search for the element in this column in the utterance.
    entityNames.forEach(function (entityName) {
        entityToFind = dataRow.data[0][entityName.column];
        if (entityToFind != "") {
            strInd = json.text.indexOf(entityToFind);
            if (strInd > -1) {
                let entityLabel = {
                    "entityName": entityName.name,
                    "startCharIndex": strInd,
                    "endCharIndex": strInd + entityToFind.length - 1
                }
                json.entityLabels.push(entityLabel);
            }
        }
    }, this);
    return json;

};


const convert = async (config) => {

    try {

        var i = 0;

        // get inFile stream
        inFileStream = await fse.createReadStream(config.inFile, 'utf-8')

        // create out file
        var myOutFile = await fse.createWriteStream(config.outFile, 'utf-8');
        var utterances = [];

        // read 1 line at a time
        return eachLine(inFileStream, (line) => {

            // skip first line with headers
            if (i++ == 0) {

                // csv to baby parser object
                let dataRow = babyparse.parse(line);

                // populate entityType list
                var index = 0;
                dataRow.data[0].forEach(function (element) {
                    if ((index != intent_column) && (index != utterance_column)) {
                        entityNames.push({ name: element, column: index });
                    }
                    index++;
                }, this);

                return;
            }

            // transform utterance from csv to json
            utterances.push(utterance(line));

        }).then(() => {
            console.log("intents: " + JSON.stringify(listOfIntents(utterances)));
            console.log("entities: " + JSON.stringify(listOfEntities(utterances)));
            myOutFile.write(JSON.stringify({ "converted_date": new Date().toLocaleString(), "utterances": utterances }));
            myOutFile.end();
            console.log("parse done");
            console.log("JSON file should contain utterances. Next step is to create an app with the intents and entities it found.");

            var model = 
            {
                intents: listOfIntents(utterances),
                entities: listOfEntities(utterances)                
            }
            return model;

        });

    } catch (err) {
        throw err;
    }

}

module.exports = convert;
```



## <a name="create-the-luis-app"></a>创建 LUIS 应用
将数据分析到 JSON 后，将其添加到 LUIS 应用。 下面的代码创建 LUIS 应用。 复制或[下载](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_create.js)它，并将其保存到 `_create.js`。

```javascript
// node 7.x
// uses async/await - promises

var rp = require('request-promise');
var fse = require('fs-extra');
var path = require('path');



// main function to call
// Call Apps_Create
var createApp = async (config) => {
    
        try {
    
            // JSON for the request body
            // { "name": MyAppName, "culture": "en-us"}
            var jsonBody = { 
                "name": config.appName, 
                "culture": config.culture
            };
    
            // Create a LUIS app
            var createAppPromise = callCreateApp({
                uri: config.uri,
                method: 'POST',
                headers: {
                    'Ocp-Apim-Subscription-Key': config.LUIS_subscriptionKey
                },
                json: true,
                body: jsonBody
            });
    
            let results = await createAppPromise;

            // Create app returns an app ID
            let appId = results.response;  
            console.log(`Called createApp, created app with ID ${appId}`);
            return appId;

    
        } catch (err) {
            console.log(`Error creating app:  ${err.message} `);
            throw err;
        }
    
    }

// Send JSON as the body of the POST request to the API
var callCreateApp = async (options) => {
    try {

        var response; 
        if (options.method === 'POST') {
            response = await rp.post(options);
        } else if (options.method === 'GET') { // TODO: There's no GET for create app
            response = await rp.get(options);
        }
        // response from successful create should be the new app ID
        return { response };

    } catch (err) {
        throw err;
    }
} 

module.exports = createApp;
```


## <a name="add-intents"></a>添加意向
创建应用后需要向其添加意向。 下面的代码创建 LUIS 应用。 复制或[下载](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_intents.js)它，并将其保存到 `_intents.js`。

```javascript

var rp = require('request-promise');
var fse = require('fs-extra');
var path = require('path');
var request = require('requestretry');

// time delay between requests
const delayMS = 1000;

// retry recount
const maxRetry = 5;

// retry request if error or 429 received
var retryStrategy = function (err, response, body) {
    let shouldRetry = err || (response.statusCode === 429);
    if (shouldRetry) console.log("retrying add intent...");
    return shouldRetry;
}

// Call add-intents
var addIntents = async (config) => {
    var intentPromises = [];
    config.uri = config.uri.replace("{appId}", config.LUIS_appId).replace("{versionId}", config.LUIS_versionId);

    config.intentList.forEach(function (intent) {
        config.intentName = intent;
        try {

            // JSON for the request body
            var jsonBody = {
                "name": config.intentName,
            };

            // Create an intent
            var addIntentPromise = callAddIntent({
                // uri: config.uri,
                url: config.uri,
                fullResponse: false,
                method: 'POST',
                headers: {
                    'Ocp-Apim-Subscription-Key': config.LUIS_subscriptionKey
                },
                json: true,
                body: jsonBody,
                maxAttempts: maxRetry,
                retryDelay: delayMS,
                retryStrategy: retryStrategy
            });
            intentPromises.push(addIntentPromise);

            console.log(`Called addIntents for intent named ${intent}.`);

        } catch (err) {
            console.log(`Error in addIntents:  ${err.message} `);

        }
    }, this);

    let results = await Promise.all(intentPromises);
    console.log(`Results of all promises = ${JSON.stringify(results)}`);
    let response = results;


}

// Send JSON as the body of the POST request to the API
var callAddIntent = async (options) => {
    try {

        var response;        
        response = await request(options);
        return { response: response };

    } catch (err) {
        console.log(`Error in callAddIntent:  ${err.message} `);
    }
}

module.exports = addIntents;
```


## <a name="add-entities"></a>添加实体
以下代码向 LUIS 应用添加实体。 复制或[下载](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_entities.js)它，并将其保存到 `_entities.js`。

```javascript
// node 7.x
// uses async/await - promises

const request = require("requestretry");
var rp = require('request-promise');
var fse = require('fs-extra');
var path = require('path');

// time delay between requests
const delayMS = 1000;

// retry recount
const maxRetry = 5;

// retry request if error or 429 received
var retryStrategy = function (err, response, body) {
    let shouldRetry = err || (response.statusCode === 429);
    if (shouldRetry) console.log("retrying add entity...");
    return shouldRetry;
}

// main function to call
// Call add-entities
var addEntities = async (config) => {
    var entityPromises = [];
    config.uri = config.uri.replace("{appId}", config.LUIS_appId).replace("{versionId}", config.LUIS_versionId);

    config.entityList.forEach(function (entity) {
        try {
            config.entityName = entity;
            // JSON for the request body
            // { "name": MyEntityName}
            var jsonBody = {
                "name": config.entityName,
            };

            // Create an app
            var addEntityPromise = callAddEntity({
                url: config.uri,
                fullResponse: false,
                method: 'POST',
                headers: {
                    'Ocp-Apim-Subscription-Key': config.LUIS_subscriptionKey
                },
                json: true,
                body: jsonBody,
                maxAttempts: maxRetry,
                retryDelay: delayMS,
                retryStrategy: retryStrategy
            });
            entityPromises.push(addEntityPromise);

            console.log(`called addEntity for entity named ${entity}.`);

        } catch (err) {
            console.log(`Error in addEntities:  ${err.message} `);
            //throw err;
        }
    }, this);
    let results = await Promise.all(entityPromises);
    console.log(`Results of all promises = ${JSON.stringify(results)}`);
    let response = results;// await fse.writeJson(createResults.json, results);


}

// Send JSON as the body of the POST request to the API
var callAddEntity = async (options) => {
    try {

        var response;        
        response = await request(options);
        return { response: response };

    } catch (err) {
        console.log(`error in callAddEntity: ${err.message}`);
    }
}

module.exports = addEntities;
```



## <a name="add-utterances"></a>添加表达
LUIS 应用中定义了实体和意向后，可以添加陈述。 下面的代码使用 [Utterances_AddBatch](https://dev.cognitive.azure.cn/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09) API，它允许每次添加最多 100 个陈述。  复制或[下载](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_upload.js)它，并将其保存到 `_upload.js`。

```javascript
// node 7.x
// uses async/await - promises

var rp = require('request-promise');
var fse = require('fs-extra');
var path = require('path');
var request = require('requestretry');

// time delay between requests
const delayMS = 500;

// retry recount
const maxRetry = 5;

// retry request if error or 429 received
var retryStrategy = function (err, response, body) {
    let shouldRetry = err || (response.statusCode === 429);
    if (shouldRetry) console.log("retrying add examples...");
    return shouldRetry;
}

// main function to call
var upload = async (config) => {

    try{
      
        // read in utterances
        var entireBatch = await fse.readJson(config.inFile);

        // break items into pages to fit max batch size
        var pages = getPagesForBatch(entireBatch.utterances, config.batchSize);

        var uploadPromises = [];

        // load up promise array
        pages.forEach(page => {
            config.uri = "https://api.cognitive.azure.cn/luis/api/v2.0/apps/{appId}/versions/{versionId}/examples".replace("{appId}", config.LUIS_appId).replace("{versionId}", config.LUIS_versionId)
            var pagePromise = sendBatchToApi({
                url: config.uri,
                fullResponse: false,
                method: 'POST',
                headers: {
                    'Ocp-Apim-Subscription-Key': config.LUIS_subscriptionKey
                },
                json: true,
                body: page,
                maxAttempts: maxRetry,
                retryDelay: delayMS,
                retryStrategy: retryStrategy
            });

            uploadPromises.push(pagePromise);
        })

        //execute promise array
        
        let results =  await Promise.all(uploadPromises)
        console.log(`\n\nResults of all promises = ${JSON.stringify(results)}`);
        let response = await fse.writeJson(config.inFile.replace('.json','.upload.json'),results);

        console.log("upload done");

    } catch(err){
        throw err;        
    }

}
// turn whole batch into pages batch 
// because API can only deal with N items in batch
var getPagesForBatch = (batch, maxItems) => {

    try{
        var pages = []; 
        var currentPage = 0;

        var pageCount = (batch.length % maxItems == 0) ? Math.round(batch.length / maxItems) : Math.round((batch.length / maxItems) + 1);

        for (let i = 0;i<pageCount;i++){

            var currentStart = currentPage * maxItems;
            var currentEnd = currentStart + maxItems;
            var pagedBatch = batch.slice(currentStart,currentEnd);

            var j = 0;
            pagedBatch.forEach(item=>{
                item.ExampleId = j++;
            });

            pages.push(pagedBatch);

            currentPage++;
        }
        return pages;
    }catch(err){
        throw(err);
    }
}

// send json batch as post.body to API
var sendBatchToApi = async (options) => {
    try {

        var response = await request(options);
        //return {page: options.body, response:response};
        return {response:response};
    }catch(err){
        throw err;
    }   
}   

module.exports = upload;
```


## <a name="run-the-code"></a>运行代码


### <a name="install-nodejs-dependencies"></a>安装 Node.js 依赖项
在终端/命令行中，从 NPM 安装 Node.js 依赖项。

```console
> npm install
```

### <a name="change-configuration-settings"></a>更改配置设置
若要使用此应用程序，需要将 index.js 文件中的值更改为自己的终结点密钥，并提供希望应用拥有的名称。 还可以设置应用的区域性或更改版本号。

打开 index.js 文件，并在文件顶部更改这些值。


```javascript
// Change these values
const LUIS_programmaticKey = "YOUR_AUTHORING_KEY";
const LUIS_appName = "Sample App";
const LUIS_appCulture = "en-us";
const LUIS_versionId = "0.1";
```

### <a name="run-the-script"></a>运行脚本
使用 Node.js 从终端/命令行运行脚本。

```console
> node index.js
```

或

```console
> npm start
```

### <a name="application-progress"></a>应用程序进度
应用程序运行时，命令行显示进度。 命令行输出包括 LUIS 的响应格式。

```console
> node index.js
intents: ["TurnOn","TurnOff","Dim","Other"]
entities: ["Operation","Device","Room"]
parse done
JSON file should contain utterances. Next step is to create an app with the intents and entities it found.
Called createApp, created app with ID 314b306c-0033-4e09-92ab-94fe5ed158a2
Called addIntents for intent named TurnOn.
Called addIntents for intent named TurnOff.
Called addIntents for intent named Dim.
Called addIntents for intent named Other.
Results of all calls to addIntent = [{"response":"e7eaf224-8c61-44ed-a6b0-2ab4dc56f1d0"},{"response":"a8a17efd-f01c-488d-ad44-a31a818cf7d7"},{"response":"bc7c32fc-14a0-4b72-bad4-d345d807f965"},{"response":"727a8d73-cd3b-4096-bc8d-d7cfba12eb44"}]
called addEntity for entity named Operation.
called addEntity for entity named Device.
called addEntity for entity named Room.
Results of all calls to addEntity= [{"response":"6a7e914f-911d-4c6c-a5bc-377afdce4390"},{"response":"56c35237-593d-47f6-9d01-2912fa488760"},{"response":"f1dd440c-2ce3-4a20-a817-a57273f169f3"}]
retrying add examples...

Results of add utterances = [{"response":[{"value":{"UtteranceText":"turn on the lights","ExampleId":-67649},"hasError":false},{"value":{"UtteranceText":"turn the heat on","ExampleId":-69067},"hasError":false},{"value":{"UtteranceText":"switch on the kitchen fan","ExampleId":-3395901},"hasError":false},{"value":{"UtteranceText":"turn off bedroom lights","ExampleId":-85402},"hasError":false},{"value":{"UtteranceText":"turn off air conditioning","ExampleId":-8991572},"hasError":false},{"value":{"UtteranceText":"kill the lights","ExampleId":-70124},"hasError":false},{"value":{"UtteranceText":"dim the lights","ExampleId":-174358},"hasError":false},{"value":{"UtteranceText":"hi how are you","ExampleId":-143722},"hasError":false},{"value":{"UtteranceText":"answer the phone","ExampleId":-69939},"hasError":false},{"value":{"UtteranceText":"are you there","ExampleId":-149588},"hasError":false},{"value":{"UtteranceText":"help","ExampleId":-81949},"hasError":false},{"value":{"UtteranceText":"testing the circuit","ExampleId":-11548708},"hasError":false}]}]
upload done
```




## <a name="open-the-luis-app"></a>打开 LUIS 应用
该脚本完成后，可以登录  。 应该能够看到在 TurnOn、TurnOff 和 None 意向下添加的陈述  。

![TurnOn 意向](./media/luis-tutorial-node-import-utterances-csv/imported-utterances-661.png)


## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [测试和训练 LUIS 网站中的应用](luis-interactive-test.md)

## <a name="additional-resources"></a>其他资源

此示例应用程序使用以下 LUIS API：
- [创建应用](https://dev.cognitive.azure.cn/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c36)
- [添加意向](https://dev.cognitive.azure.cn/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0c)
- [添加实体](https://dev.cognitive.azure.cn/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0e)
- [添加陈述](https://dev.cognitive.azure.cn/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09)

