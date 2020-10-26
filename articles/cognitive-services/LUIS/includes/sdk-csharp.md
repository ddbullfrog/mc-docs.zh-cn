---
title: include 文件
description: include 文件
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 10/23/2020
ms.topic: include
ms.custom: include file, devx-track-dotnet, cog-serv-seo-aug-2020
ms.openlocfilehash: bb804ddb8a30ba65c16e6de22a75f44e1183e49a
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472966"
---
使用适用于 .NET 的语言理解 (LUIS) 客户端库执行以下操作：
* 创建应用
* 使用示例言语添加意向（机器学习实体）
* 训练和发布应用
* 查询预测运行时

[参考文档](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/languageunderstanding?view=azure-dotnet) | [创作](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Language.LUIS.Authoring)和[预测](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Language.LUIS.Runtime)库源代码 | [创作](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring/)和[预测](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime/) NuGet | [C# 示例](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/LanguageUnderstanding/sdk-3x//Program.cs)

## <a name="prerequisites"></a>先决条件

* [.NET Core](https://dotnet.microsoft.com/download/dotnet-core) 和 [.NET Core CLI](https://docs.microsoft.com/dotnet/core/tools/) 的当前版本。
* Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/details/cognitive-services)
* 有了 Azure 订阅后，在 Azure 门户中[创建语言理解创作资源](https://portal.azure.cn/#create/Microsoft.CognitiveServicesLUISAllInOne)，以获取创作密钥和终结点。 等待其部署并单击“转到资源”按钮。
    * 需要从[创建](../luis-how-to-azure-subscription.md#create-luis-resources-in-azure-portal)的资源获取密钥和终结点，以便将应用程序连接到语言理解创作。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。 可以使用免费定价层 (`F0`) 来试用该服务。

## <a name="setting-up"></a>设置

### <a name="create-a-new-c-application"></a>新建 C# 应用程序

在首选编辑器或 IDE 中创建新的 .NET Core 应用程序。

1. 在控制台窗口（例如 CMD、PowerShell 或 Bash）中，使用 dotnet `new` 命令创建名为 `language-understanding-quickstart` 的新控制台应用。 此命令将创建包含单个源文件的简单“Hello World”C# 项目：`Program.cs`。

    ```dotnetcli
    dotnet new console -n language-understanding-quickstart
    ```

1. 将目录更改为新创建的应用文件夹。

    ```dotnetcli
    cd language-understanding-quickstart
    ```

1. 可使用以下代码生成应用程序：

    ```dotnetcli
    dotnet build
    ```

    生成输出不应包含警告或错误。

    ```console
    ...
    Build succeeded.
     0 Warning(s)
     0 Error(s)
    ...
    ```

### <a name="install-the-nuget-libraries"></a>安装 NuGet 库

在应用程序目录中，使用以下命令安装适用于 .NET 的语言理解 (LUIS) 客户端库：

```dotnetcli
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.2.0-preview.3
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.1.0-preview.1
```

## <a name="authoring-object-model"></a>创作对象模型

语言理解 (LUIS) 创作客户端是对 Azure 进行身份验证的 [LUISAuthoringClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.luisauthoringclient?view=azure-dotnet) 对象，其中包含创作密钥。

## <a name="code-examples-for-authoring"></a>创作的代码示例

创建客户端后，可以使用此客户端访问如下所述的功能：

* 应用 - [创建](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.addasync?view=azure-dotnet)、[删除](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.deleteasync?view=azure-dotnet)、[发布](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.publishasync?view=azure-dotnet)
* 示例言语 - [添加](https://docs.microsoft.com//dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.addasync?view=azure-dotnet)、[按 ID 删除](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.deleteasync?view=azure-dotnet)
* 特征 - 管理[短语列表](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.featuresextensions.addphraselistasync?view=azure-dotnet)
* 模型 - 管理[意向](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions?view=azure-dotnet)和实体
* 模式 - 管理[模式](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.patternextensions?view=azure-dotnet)
* 训练 - [训练](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.trainversionasync?view=azure-dotnet)应用和轮询[训练状态](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.getstatusasync?view=azure-dotnet)
* [版本](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.versionsextensions?view=azure-dotnet) - 使用克隆、导出和删除操作进行管理

## <a name="prediction-object-model"></a>预测对象模型

语言理解 (LUIS) 预测运行时客户端是对 Azure 进行身份验证的 [LUISRuntimeClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.luisruntimeclient?view=azure-dotnet) 对象，其中包含资源密钥。

## <a name="code-examples-for-prediction-runtime"></a>预测运行时的代码示例

创建客户端后，可以使用此客户端访问如下所述的功能：

* 按[过渡槽或生产槽](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.predictionoperationsextensions.getslotpredictionasync?view=azure-dotnet)进行预测
* 按[版本](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.predictionoperationsextensions.getversionpredictionasync?view=azure-dotnet)进行预测


[!INCLUDE [Bookmark links to same article](sdk-code-examples.md)]

## <a name="add-the-dependencies"></a>添加依赖项

在首选的编辑器或 IDE 中，从项目目录打开 *Program.cs* 文件。 将现有 `using` 代码替换为以下 `using` 指令：

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="add-boilerplate-code"></a>添加样板代码

1. 更改 `Main` 方法的签名以允许异步调用：

    ```csharp
    public static async Task Main()
    ```

1. 将其余代码添加到 `Program` 类的 `Main` 方法中（除非另有指定）。

## <a name="create-variables-for-the-app"></a>为应用创建变量

创建两组变量：第一组为你更改的变量，第二组保留在代码示例中显示的状态。 

1. 创建用于保存创作密钥和资源名称的变量。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

1. 创建用于保存终结点、应用名称、版本和意向名称的变量。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="authenticate-the-client"></a>验证客户端

使用密钥创建 [ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.apikeyserviceclientcredentials?view=azure-dotnet) 对象，并在终结点中使用该对象创建一个 [LUISAuthoringClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.luisauthoringclient?view=azure-dotnet) 对象。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="create-a-luis-app"></a>创建 LUIS 应用

LUIS 应用包含自然语言处理 (NLP) 模型，包括意向、实体和示例言语。

创建 [ApplicationCreateObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.applicationcreateobject?view=azure-dotnet)。 名称和语言区域性是必需的属性。 调用 [Apps.AddAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.addasync?view=azure-dotnet) 方法。 响应为应用 ID。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="create-intent-for-the-app"></a>为应用创建意向
LUIS 应用模型中的主要对象是意向。 意向与用户言语意向的分组相符。 用户可以提问，或者做出表述，指出希望机器人（或其他客户端应用程序）提供特定的有针对性响应。 意向的示例包括预订航班、询问目的地城市的天气，以及询问客户服务的联系信息。

使用唯一意向的名称创建 [ModelCreateObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.modelcreateobject?view=azure-dotnet)，然后将应用 ID、版本 ID 和 ModelCreateObject 传递给 [Model.AddIntentAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions.addintentasync?view=azure-dotnet) 方法。 响应为意向 ID。

`intentName` 值硬编码为 `OrderPizzaIntent`，作为[为应用创建变量](#create-variables-for-the-app)中变量的一部分。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="create-entities-for-the-app"></a>为应用创建实体

尽管实体不是必需的，但在大多数应用中都可以看到实体。 实体从用户言语中提取信息，只有使用这些信息才能实现用户的意向。 有多种类型的[预生成](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions.addprebuiltasync?view=azure-dotnet)实体和自定义实体，每种实体具有自身的数据转换对象 (DTO) 模型。  在应用中添加的常见预生成实体包括 [number](../luis-reference-prebuilt-number.md)、[datetimeV2](../luis-reference-prebuilt-datetimev2.md)、[geographyV2](../luis-reference-prebuilt-geographyv2.md) 和 [ordinal](../luis-reference-prebuilt-ordinal.md)。

必须知道，实体不会使用意向进行标记。 它们可以并且通常应用到多个意向。 只会为特定的单个意向标记示例用户言语。

实体的创建方法属于 [Model](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions?view=azure-dotnet) 类的一部分。 每个实体类型有自身的数据转换对象 (DTO) 模型，该模型通常在 [Models](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models?view=azure-dotnet) 命名空间中包含单词 `model`。

实体创建代码会创建机器学习实体，其中包含子实体以及应用于 `Quantity` 子实体的功能。

:::image type="content" source="../media/quickstart-sdk/machine-learned-entity.png" alt-text="显示创建的实体的门户的部分屏幕截图，其中为包含子实体以及应用于 `Quantity` 子实体的功能的机器学习实体。":::

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

对该类使用以下方法，查找 Quantity 子实体的 ID，以便将功能分配给该子实体。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="add-example-utterance-to-intent"></a>将示例言语添加到意向

为了确定言语的意向并提取实体，应用需要言语示例。 这些示例需要针对特定的单个意向，并且应该标记所有自定义实体。 无需标记预生成实体。

通过创建 [ExampleLabelObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.examplelabelobject?view=azure-dotnet) 对象的列表来添加示例言语（每个示例言语对应于一个对象）。 每个示例应使用实体名称和实体值的名称/值对字典来标记所有实体。 实体值应与示例言语文本中显示的值完全相同。

:::image type="content" source="../media/quickstart-sdk/labeled-example-machine-learned-entity.png" alt-text="显示创建的实体的门户的部分屏幕截图，其中为包含子实体以及应用于 `Quantity` 子实体的功能的机器学习实体。":::

结合应用 ID、版本 ID 和示例调用 [Examples.AddAsync](https://docs.microsoft.com//dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.addasync?view=azure-dotnet)。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="train-the-app"></a>训练应用

创建模型后，需要为此模型版本训练 LUIS 应用。 训练后的模型可在[容器](../luis-container-howto.md)中使用，或者将其[发布](../luis-how-to-publish-app.md)到过渡槽或生产槽。

[Train.TrainVersionAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions?view=azure-dotnet) 方法需要应用 ID 和版本 ID。

极小的模型（如本快速入门中所示的模型）很快就能完成训练。 对于生产级应用程序，应用的训练应该包括轮询调用 [GetStatusAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.getstatusasync?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_TrainExtensions_GetStatusAsync_Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_ITrain_System_Guid_System_String_System_Threading_CancellationToken_) 方法以确定训练何时或者是否成功。 响应是一个 [ModelTrainingInfo](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.modeltraininginfo?view=azure-dotnet) 对象列表，其中分别列出了每个对象的状态。 所有对象必须成功，才能将训练视为完成。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="publish-app-to-production-slot"></a>将应用发布到生产槽

使用 [PublishAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.publishasync?view=azure-dotnet) 方法发布 LUIS 应用。 这会将当前已训练的版本发布到终结点上的指定槽。 客户端应用程序使用此终结点发送用户言语，以预测意向和提取实体。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

## <a name="authenticate-the-prediction-runtime-client"></a>对预测运行时客户端进行身份验证

将 [ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.apikeyserviceclientcredentials?view=azure-dotnet) 对象和密钥一起使用，并在终结点中使用该对象创建一个 [LUISRuntimeClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.luisruntimeclient?view=azure-dotnet) 对象。

[!INCLUDE [Caution about using authoring key](caution-authoring-key.md)]

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```


## <a name="get-prediction-from-runtime"></a>从运行时获取预测

添加以下代码以创建对预测运行时的请求。

用户言语是 [PredictionRequest](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime.models.predictionrequest?view=azure-dotnet) 对象的一部分。

GetSlotPredictionAsync 方法需要多个参数，如应用 ID、槽名称、用于满足请求的预测请求对象。 其他选项（如详细、显示所有意向和日志）都是可选的。

```csharp
/* To run this sample, install the following modules.
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
 * dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
 */

// <Dependencies>
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.Models;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
using Newtonsoft.Json;
// </Dependencies>

namespace MlEntitySample
{
    public static class MlEntitySample
    {
        public static async Task Main()
        {
            // <VariablesYouChange>
            var key = "REPLACE-WITH-YOUR-AUTHORING-KEY";

            var authoringResourceName = "REPLACE-WITH-YOUR-AUTHORING-RESOURCE-NAME";
            var predictionResourceName = "REPLACE-WITH-YOUR-PREDICTION-RESOURCE-NAME";
            // </VariablesYouChange>

            // <VariablesYouDontNeedToChangeChange>
            var authoringEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", authoringResourceName);
            var predictionEndpoint = String.Format("https://{0}.cognitiveservices.azure.cn/", predictionResourceName);

            var appName = "Contoso Pizza Company";
            var versionId = "0.1";
            var intentName = "OrderPizzaIntent";
            // </VariablesYouDontNeedToChangeChange>

            // <AuthoringCreateClient> 
            var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring.ApiKeyServiceClientCredentials(key);
            var client = new LUISAuthoringClient(credentials) { Endpoint = authoringEndpoint };
            // </AuthoringCreateClient>

            // Create app
            var appId = await CreateApplication(client, appName, versionId);

            // <AddIntent>
            await client.Model.AddIntentAsync(appId, versionId, new ModelCreateObject()
            {
                Name = intentName
            });
            // </AddIntent>

            // Add Entities
            await AddEntities(client, appId, versionId);

            // Add Labeled example utterance
            await AddLabeledExample(client, appId, versionId, intentName);

            // <TrainAppVersion>
            await client.Train.TrainVersionAsync(appId, versionId);
            while (true)
            {
                var status = await client.Train.GetStatusAsync(appId, versionId);
                if (status.All(m => m.Details.Status == "Success"))
                {
                    // Assumes that we never fail, and that eventually we'll always succeed.
                    break;
                }
            }
            // </TrainAppVersion>

            // <PublishVersion>
            await client.Apps.PublishAsync(appId, new ApplicationPublishObject { VersionId = versionId, IsStaging=false});
            // </PublishVersion>

            // <PredictionCreateClient>
            var runtimeClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
            // </PredictionCreateClient>
            
            // <QueryPredictionEndpoint>
            // Production == slot name
            var request = new PredictionRequest { Query = "I want two small pepperoni pizzas with more salsa" };
            var prediction = await runtimeClient.Prediction.GetSlotPredictionAsync(appId, "Production", request);
            Console.Write(JsonConvert.SerializeObject(prediction, Formatting.Indented));
            // </QueryPredictionEndpoint>
        }

        async static Task<Guid> CreateApplication(LUISAuthoringClient client, string appName, string versionId)
        {
            // <AuthoringCreateApplication>
            var newApp = new ApplicationCreateObject
            {
                Culture = "en-us",
                Name = appName,
                InitialVersionId = versionId
            };

            var appId = await client.Apps.AddAsync(newApp);
            // </AuthoringCreateApplication>

            Console.WriteLine("New app ID {0}.", appId);
            return appId;
        }

        async static Task AddEntities(LUISAuthoringClient client, Guid appId, string versionId)
        {

            // <AuthoringAddEntities>
            // Add Prebuilt entity
            await client.Model.AddPrebuiltAsync(appId, versionId, new[] { "number" });
            // </AuthoringCreatePrebuiltEntity>

            // Define ml entity with children and grandchildren
            var mlEntityDefinition = new EntityModelCreateObject
            {
                Name = "Pizza order",
                Children = new[]
                {
                    new ChildEntityModelCreateObject
                    {
                        Name = "Pizza",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Quantity" },
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Size" }
                        }
                    },
                    new ChildEntityModelCreateObject
                    {
                        Name = "Toppings",
                        Children = new[]
                        {
                            new ChildEntityModelCreateObject { Name = "Type" },
                            new ChildEntityModelCreateObject { Name = "Quantity" }
                        }
                    }
                }
            };

            // Add ML entity 
            var mlEntityId = await client.Model.AddEntityAsync(appId, versionId, mlEntityDefinition); ;

            // Add phraselist feature
            var phraselistId = await client.Features.AddPhraseListAsync(appId, versionId, new PhraselistCreateObject
            {
                EnabledForAllModels = false,
                IsExchangeable = true,
                Name = "QuantityPhraselist",
                Phrases = "few,more,extra"
            });

            // Get entity and subentities
            var model = await client.Model.GetEntityAsync(appId, versionId, mlEntityId);
            var toppingQuantityId = GetModelGrandchild(model, "Toppings", "Quantity");
            var pizzaQuantityId = GetModelGrandchild(model, "Pizza", "Quantity");

            // add model as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, pizzaQuantityId, new ModelFeatureInformation { ModelName = "number", IsRequired = true });
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { ModelName = "number"});
            
            // add phrase list as feature to subentity model
            await client.Features.AddEntityFeatureAsync(appId, versionId, toppingQuantityId, new ModelFeatureInformation { FeatureName = "QuantityPhraselist" });
            // </AuthoringAddEntities>
        }
        

        
        async static Task AddLabeledExample(LUISAuthoringClient client, Guid appId, string versionId, string intentName)
        {
            // <AuthoringAddLabeledExamples>
            // Define labeled example
            var labeledExampleUtteranceWithMLEntity = new ExampleLabelObject
            {
                Text = "I want two small seafood pizzas with extra cheese.",
                IntentName = intentName,
                EntityLabels = new[]
                {
                    new EntityLabelObject
                    {
                        StartCharIndex = 7,
                        EndCharIndex = 48,
                        EntityName = "Pizza order",
                        Children = new[]
                        {
                            new EntityLabelObject
                            {
                                StartCharIndex = 7,
                                EndCharIndex = 30,
                                EntityName = "Pizza",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 7, EndCharIndex = 9, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 11, EndCharIndex = 15, EntityName = "Size" },
                                    new EntityLabelObject { StartCharIndex = 17, EndCharIndex = 23, EntityName = "Type" }
                                }
                            },
                            new EntityLabelObject
                            {
                                StartCharIndex = 37,
                                EndCharIndex = 48,
                                EntityName = "Toppings",
                                Children = new[]
                                {
                                    new EntityLabelObject { StartCharIndex = 37, EndCharIndex = 41, EntityName = "Quantity" },
                                    new EntityLabelObject { StartCharIndex = 43, EndCharIndex = 48, EntityName = "Type" }
                                }
                            }
                        }
                    },
                }
            };

            // Add an example for the entity.
            // Enable nested children to allow using multiple models with the same name.
            // The quantity subentity and the phraselist could have the same exact name if this is set to True
            await client.Examples.AddAsync(appId, versionId, labeledExampleUtteranceWithMLEntity, enableNestedChildren: true); 
            // </AuthoringAddLabeledExamples>
        }

        // <AuthoringSortModelObject>
        static Guid GetModelGrandchild(NDepthEntityExtractor model, string childName, string grandchildName)
        {
            return model.Children.
                Single(c => c.Name == childName).
                Children.
                Single(c => c.Name == grandchildName).Id;
        }
        // </AuthoringSortModelObject>
    }
}
```

[!INCLUDE [Prediction JSON response](sdk-json.md)]

## <a name="run-the-application"></a>运行应用程序

从应用程序目录使用 `dotnet run` 命令运行应用程序。

```dotnetcli
dotnet run
```

