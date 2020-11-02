---
title: 教程：审查电子商务产品图像 - 内容审查器
titleSuffix: Azure Cognitive Services
description: 本教程说明如何设置应用程序以使用指定的标签（使用 Azure 计算机视觉和自定义视觉）对产品图像进行分析和分类。 标记令人反感的图像以作进一步的评审（使用 Azure 内容审查器）。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: tutorial
ms.date: 10/27/2020
origin.date: 01/27/2020
ms.author: v-johya
ms.custom: devx-track-csharp
ms.openlocfilehash: 0008a623cc41235c45ecd576f56b83a4da112468
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103967"
---
# <a name="tutorial-moderate-e-commerce-product-images-with-azure-content-moderator"></a>教程：使用 Azure 内容审查器审查电子商务产品图像

本教程介绍如何使用 Azure 认知服务（包括内容审查器）分类和审查电子商务场景中的产品图像。 你将使用计算机视觉和自定义视觉向图像应用标记（标签），然后创建团队评审，以便将内容审查器的基于机器学习的技术与人工评审团队相结合，提供一个智能审查系统。

本教程演示如何：

> [!div class="checklist"]
> * 注册内容审查器并创建评审团队。
> * 使用内容审查器的图像 API 扫描可能出现的成人和不雅内容。
> * 使用计算机视觉服务扫描名人内容（或计算机视觉可检测到的其他标记）。
> * 使用自定义视觉服务扫描国旗、玩具和笔（或其他自定义标记）。
> * 提供组合的扫描结果，供人工评审和最终决策。

GitHub 上的[电子商务目录审查示例](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration)存储库中提供了完整的示例代码。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/details/cognitive-services/)。

## <a name="prerequisites"></a>先决条件

- 内容审查器的订阅密钥。 遵照[创建认知服务帐户](https://docs.azure.cn/cognitive-services/cognitive-services-apis-create-account)中的说明订阅内容审查器服务并获取密钥。
- 计算机视觉订阅密钥（遵照上面相同的说明获取）。
- 任何版本的 [Visual Studio 2015 或 2017](https://www.visualstudio.com/downloads/)。
- 为每个标签提供一组图像，供自定义视觉分类器使用（在本例中为玩具、笔和美国国旗）。

## <a name="create-a-review-team"></a>创建评审团队

请参阅[在 Web 上试用内容审查器](quick-start.md)快速入门，获取有关如何注册[内容审查器评审工具](https://contentmoderator.cognitive.microsoft.com/)和创建评审团队的说明。 记下“凭据”页上的“团队 ID”值。  

## <a name="create-custom-moderation-tags"></a>创建自定义审查标记

接下来，在评审工具中创建自定义标记（在此过程中如需帮助，请参阅[标记](https://docs.azure.cn/cognitive-services/content-moderator/review-tool-user-guide/tags)一文）。 在本例中，我们将添加以下标记：“名人”、“美国”、“国旗”、“玩具”和“笔”。      并非所有标记都需要是计算机视觉中可检测到的类别（例如“名人”）；可以添加自己的自定义标记，不过，在添加后，必须训练自定义视觉分类器才能检测到这些标记。 

![配置自定义标记](images/tutorial-ecommerce-tags2.PNG)

## <a name="create-visual-studio-project"></a>创建 Visual Studio 项目

1. 在 Visual Studio 中打开“新建项目”对话框。 依次展开“已安装”、“Visual C#”，然后选择“控制台应用(.NET Framework)”。   
1. 将应用程序命名为 **EcommerceModeration** ，然后单击“确定”。 
1. 如果要将此项目添加到现有的解决方案，请将此项目选作单一启动项目。

本教程会突出显示项目的核心代码，但不会介绍每个代码行。 将示例项目（ [电子商务目录审查示例](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration)）中 _Program.cs_ 的整个内容复制到新项目的 _Program.cs_ 文件。 然后，逐步完成以下各个部分，以了解项目的工作原理以及如何自行使用该项目。

## <a name="define-api-keys-and-endpoints"></a>定义 API 密钥和终结点

本教程将使用三个认知服务；因此，需要三个对应的密钥和 API 终结点。 查看 **Program** 类中的以下字段：

```csharp
        public const string ContentModeratorKey = "XXXXXXXXXXXXX";
        public const string ComputerVisionKey = "XXXXXXXXXXXX";
        public const string CustomVisionKey = "XXXXXXXXXXX";

        // All your end points based on the new account and subscriptions
        public const string ImageUri = "https://api.cognitive.azure.cn/contentmoderator/moderate/v1.0/ProcessImage/Evaluate";
        public const string ReviewUri = "https://api.cognitive.azure.cn/contentmoderator/review/v1.0/teams/YOUR-TEAM-ID/reviews";
        public const string ComputerVisionUri = "https://api.cognitive.azure.cn/vision/v1.0/analyze?details=celebrities";
        public const string CustomVisionUri = "https://api.cognitive.azure.cn/customvision/v1.0/Prediction/XXXXXXXXXXXXXXXX/url";
```

需要使用订阅密钥的值更新 `___Key` 字段，并且需要将 `___Uri` 字段更改为正确的终结点 URL（稍后将获得自定义视觉密钥和终结点）。 可以在每个 Azure 资源的“快速入门”选项卡中找到这些值  。 在 `ReviewUri` 字段的 `YOURTEAMID` 部分填写之前创建的评审团队的 ID。 稍后再填充 `CustomVisionUri` 字段的最后一个部分。

[!INCLUDE [subdomains note](../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="primary-method-calls"></a>主要方法调用

查看 **Main** 方法中循环访问图像 URL 列表的以下代码。 该代码使用三个不同的服务分析每个图像，在 **ReviewTags** 数组中记录已应用的标记，然后为审查人员创建一个评审（将图像发送到内容审查器评审工具）。 以下部分将介绍这些方法。 可以根据需要控制要发送哪些图像供评审，只需在条件语句中使用 **ReviewTags** 数组来检查已应用的标记即可。

```csharp
            // for each image URL in the file...
            foreach (var Url in Urls)
            {
                // Initiatize a new review tags array
                ReviewTags = new KeyValuePair[MAXTAGSCOUNT];

                // Evaluate for potential adult and racy content with Content Moderator API
                EvaluateAdultRacy(Url, ref ReviewTags);

                // Evaluate for potential presence of celebrity (ies) in images with Computer Vision API
                EvaluateComputerVisionTags(Url, ComputerVisionUri, ComputerVisionKey, ref ReviewTags);

                // Evaluate for potential presence of custom categories other than Marijuana
                EvaluateCustomVisionTags(Url, CustomVisionUri, CustomVisionKey, ref ReviewTags);

                // Create review in the Content Moderator review tool
                CreateReview(Url, ReviewTags);
            }
```

## <a name="evaluateadultracy-method"></a>EvaluateAdultRacy 方法

查看 **Program** 类中的 **EvaluateAdultRacy** 方法。 此方法将图像 URL 和键值对数组用作参数。 它调用内容审查器的图像 API（使用 REST）获取图像的成人和不雅评分。 如果评分大于 0.4（范围为 0 到 1），则将 **ReviewTags** 数组中的相应值设置为 **True** 。

```csharp
        /// <summary>
        /// Use Content Moderator API to evaluate for potential adult and racy content
        /// </summary>
        /// <param name="ImageUrl"></param>
        /// <param name="ReviewTags"></param>
        /// <returns>API call success or not</returns>
        public static bool EvaluateAdultRacy(string ImageUrl, ref KeyValuePair[] ReviewTags)
        {
            float AdultScore = 0;
            float RacyScore = 0;

            var File = ImageUrl;
            string Body = $"{{\"DataRepresentation\":\"URL\",\"Value\":\"{File}\"}}";

            HttpResponseMessage response = CallAPI(ImageUri, ContentModeratorKey, CallType.POST,
                                                   "Ocp-Apim-Subscription-Key", "application/json", "", Body);

            if (response.IsSuccessStatusCode)
            {
                // {“answers”:[{“answer”:“Hello”,“questions”:[“Hi”],“score”:100.0}]}
                // Parse the response body. Blocking!
                GetAdultRacyScores(response.Content.ReadAsStringAsync().Result, out AdultScore, out RacyScore);
            }

            ReviewTags[0] = new KeyValuePair();
            ReviewTags[0].Key = "a";
            ReviewTags[0].Value = "false";
            if (AdultScore > 0.4)
            {
                ReviewTags[0].Value = "true";
            }

            ReviewTags[1] = new KeyValuePair();
            ReviewTags[1].Key = "r";
            ReviewTags[1].Value = "false";
            if (RacyScore > 0.3)
            {
                ReviewTags[1].Value = "true";
            }
            return response.IsSuccessStatusCode;
        }
```

## <a name="evaluatecomputervisiontags-method"></a>EvaluateComputerVisionTags 方法

下一个方法采用图像 URL 和计算机视觉订阅信息，并分析图像中是否存在名人。 如果找到一个或多个名人，则将 **ReviewTags** 数组中的相应值设置为 **True** 。

```csharp
        /// <summary>
        /// Use Computer Vision API to evaluate for potential celebrity presence in image
        /// </summary>
        /// <param name="ImageUrl"></param>
        /// <param name="ComputerVisionUri"></param>
        /// <param name="ComputerVisionKey"></param>
        /// <param name="ReviewTags"></param>
        /// <returns>API call success or not</returns>
        public static bool EvaluateComputerVisionTags(string ImageUrl, string ComputerVisionUri, string ComputerVisionKey, ref KeyValuePair[] ReviewTags)
        {
            var File = ImageUrl;
            string Body = $"{{\"URL\":\"{File}\"}}";

            HttpResponseMessage Response = CallAPI(ComputerVisionUri, ComputerVisionKey, CallType.POST,
                                                   "Ocp-Apim-Subscription-Key", "application/json", "", Body);

            if (Response.IsSuccessStatusCode)
            {
                ReviewTags[2] = new KeyValuePair();
                ReviewTags[2].Key = "cb";
                ReviewTags[2].Value = "false";

                ComputerVisionPrediction CVObject = JsonConvert.DeserializeObject<ComputerVisionPrediction>(Response.Content.ReadAsStringAsync().Result);

                if ((CVObject.categories[0].detail != null) && (CVObject.categories[0].detail.celebrities.Count() > 0))
                {                 
                    ReviewTags[2].Value = "true";
                }
            }

            return Response.IsSuccessStatusCode;
        }
```

## <a name="evaluatecustomvisiontags-method"></a>EvaluateCustomVisionTags 方法

接下来查看 **EvaluateCustomVisionTags** 方法。该方法分类实际产品 &mdash; 在本例中为国旗、玩具和笔。 遵照[如何生成分类器](https://docs.azure.cn/cognitive-services/custom-vision-service/getting-started-build-a-classifier)指南中的说明生成自己的自定义图像分类器，并检测图像中的国旗、玩具和笔（或选作自定义标记的任何内容）。 可以使用 [GitHub 存储库](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration)的 **sample-images** 文件夹中的图像快速训练此示例中的某些类别。

![包含笔、玩具和国旗训练图像的自定义视觉网页](images/tutorial-ecommerce-custom-vision.PNG)

训练分类器后，获取预测密钥和预测终结点 URL（检索这些信息时如需帮助，请参阅[获取 URL 和预测密钥](https://docs.azure.cn/cognitive-services/custom-vision-service/use-prediction-api#get-the-url-and-prediction-key)），并将这些值分别分配到 `CustomVisionKey` 和 `CustomVisionUri` 字段。 该方法使用这些值来查询分类器。 如果分类器在图像中找到一个或多个自定义标记，此方法会将 **ReviewTags** 数组中的相应值设置为 **True** 。

```csharp
       /// <summary>
       /// Use Custom Vision API to evaluate for potential content from custom-trained categories 
       /// </summary>
       /// <param name="ImageUrl"></param>
       /// <param name="CustomVisionUri"></param>
       /// <param name="CustomVisionKey"></param>
       /// <param name="ReviewTags"></param>
       /// <returns>API call success or not</returns>
        public static bool EvaluateCustomVisionTags(string ImageUrl, string CustomVisionUri, string CustomVisionKey, ref KeyValuePair[] ReviewTags)
        {
            var File = ImageUrl;
            string Body = $"{{\"URL\":\"{File}\"}}";

            HttpResponseMessage response = CallAPI(CustomVisionUri, CustomVisionKey, CallType.POST,
                                                   "Prediction-Key", "application/json", "", Body);

            if (response.IsSuccessStatusCode)
            {
                // Parse the response body. Blocking!
                SaveCustomVisionTags(response.Content.ReadAsStringAsync().Result, ref ReviewTags);
            }

            return response.IsSuccessStatusCode;
        }
```

## <a name="create-reviews-for-review-tool"></a>为评审工具创建评审

前面的部分介绍了应用如何扫描传入图像中的成人和不雅内容（内容审查器）、名人（计算机视觉）和其他各种对象（自定义视觉）。 接下来了解 CreateReview 方法。该方法将图像及其中应用的所有标记（作为元数据传入）上传到内容审查器审阅工具  。 

```csharp
        /// <summary>
        /// Call Content Moderator's Review API to create a review with the image (URL) and the review tags as inputs
        /// </summary>
        /// <param name="ImageUrl"></param>
        /// <param name="Metadata"></param>
        /// <returns>API call success or not</returns>
        public static bool CreateReview(string ImageUrl, KeyValuePair[] Metadata)
        {

            ReviewCreationRequest Review = new ReviewCreationRequest();
            Review.Item[0] = new ReviewItem();
            Review.Item[0].Content = ImageUrl;
            Review.Item[0].Metadata = new KeyValuePair[MAXTAGSCOUNT];
            Metadata.CopyTo(Review.Item[0].Metadata, 0);

            //SortReviewItems(ref Review);

            string Body = JsonConvert.SerializeObject(Review.Item);

            HttpResponseMessage response = CallAPI(ReviewUri, ContentModeratorKey, CallType.POST,
                                                   "Ocp-Apim-Subscription-Key", "application/json", "", Body);

            return response.IsSuccessStatusCode;
        }
```

图像将显示在[内容审查器评审工具](https://contentmoderator.cognitive.microsoft.com/)的“评审”选项卡中。

![内容审查器评审工具的屏幕截图，其中包含多个图像及其突出显示的标记](images/tutorial-ecommerce-content-moderator.PNG)

## <a name="submit-a-list-of-test-images"></a>提交测试图像列表

在 **Main** 方法中可以看到，此程序将查找“C:Test”目录，该目录中的 _Urls.txt_ 文件包含图像 URL 列表。 请创建此类文件和目录，或者将路径更改为指向你的文本文件。 然后，在此文件中填充要测试的图像的 URL。

```csharp
            // Check for a test directory for a text file with the list of Image URLs to scan
            var topdir = @"C:\test\";
            var Urlsfile = topdir + "Urls.txt";

            if (!Directory.Exists(topdir))
                return;

            if (!File.Exists(Urlsfile))
            {
                return;
            }

            // Read all image URLs in the file
            var Urls = File.ReadLines(Urlsfile);
```

## <a name="run-the-program"></a>运行程序

如果已遵循上述所有步骤，该程序应会处理每个图像（在所有三个服务中查询其相关标记），然后将图像连同标记信息一起上传到内容审查器评审工具。

## <a name="next-steps"></a>后续步骤

在本教程中，你设置一个程序来分析产品图像，按产品类型对其进行标记，使评审团队能够在内容审查方面做出明智的决策。 接下来，请了解有关图像审查的详细信息。

> [!div class="nextstepaction"]
> [评审已审查的图像](./review-tool-user-guide/review-moderated-images.md)

