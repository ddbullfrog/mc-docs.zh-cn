---
title: 人脸 .NET 客户端库快速入门
description: 使用适用于 .NET 的人脸客户端库来检测人脸、查找相似的人脸（按图像进行人脸搜索）、识别人脸（人脸识别搜索）并迁移人脸数据。
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 10/28/2020
ms.author: v-johya
ms.openlocfilehash: 5facd6591a65f72d8436cda44a2661bb3d8cb0e1
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105460"
---
开始使用适用于 .NET 的人脸客户端库进行人脸识别。 请按照以下步骤安装程序包并试用基本任务的示例代码。 通过人脸服务，可以访问用于检测和识别图像中的人脸的高级算法。

使用适用于 .NET 的人脸客户端库可以：

* [检测图像中的人脸](#detect-faces-in-an-image)
* [查找相似人脸](#find-similar-faces)
* [创建和训练人员组](#create-and-train-a-person-group)
* [识别人脸](#identify-a-face)

[参考文档](https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/faceapi?view=azure-dotnet) | [库源代码](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.Face) | [包 (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.6.0-preview.1) | [示例](https://docs.microsoft.com/samples/browse/?products=azure&term=face)

## <a name="prerequisites"></a>先决条件


* Azure 订阅 - [免费创建订阅](https://www.azure.cn/pricing/details/cognitive-services/)
* [Visual Studio IDE](https://visualstudio.microsoft.com/vs/) 或最新版本的 [.NET Core](https://dotnet.microsoft.com/download/dotnet-core)。
* 拥有 Azure 订阅后，在 Azure 门户中<a href="https://portal.azure.cn/#create/Microsoft.CognitiveServicesFace"  title="创建人脸资源"  target="_blank">创建人脸资源 <span class="docon docon-navigate-external x-hidden-focus"></span></a>，获取密钥和终结点。 部署后，单击“转到资源”。
    * 需要从创建的资源获取密钥和终结点，以便将应用程序连接到人脸 API。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    * 可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。

## <a name="setting-up"></a>设置

### <a name="create-a-new-c-application"></a>新建 C# 应用程序

#### <a name="visual-studio-ide"></a>[Visual Studio IDE](#tab/visual-studio)

使用 Visual Studio 创建新的 .NET Core 应用程序。 

### <a name="install-the-client-library"></a>安装客户端库 

创建新项目后，右键单击“解决方案资源管理器”中的项目解决方案，然后选择“管理 NuGet 包”，以安装客户端库 。 在打开的包管理器中，选择“浏览”，选中“包括预发行版”并搜索 `Microsoft.Azure.CognitiveServices.Vision.Face`。 选择版本 `2.6.0-preview.1`，然后选择“安装”。 

#### <a name="cli"></a>[CLI](#tab/cli)

在控制台窗口（例如 cmd、PowerShell 或 Bash）中，使用 `dotnet new` 命令创建名为 `face-quickstart` 的新控制台应用。 此命令将创建包含单个源文件的简单“Hello World”C# 项目： *program.cs* 。 

```console
dotnet new console -n face-quickstart
```

将目录更改为新创建的应用文件夹。 可使用以下代码生成应用程序：

```console
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

### <a name="install-the-client-library"></a>安装客户端库 

在应用程序目录中，使用以下命令安装适用于 .NET 的人脸客户端库：

```console
dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
```

---

> [!TIP]
> 想要立即查看整个快速入门代码文件？ 可以在 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/Face/FaceQuickstart.cs) 上找到它，其中包含此快速入门中的代码示例。


从项目目录中，打开 Program.cs 文件，并添加以下 `using` 指令：

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

在应用程序的“Program”类中，为资源的密钥和终结点创建变量。


> [!IMPORTANT]
> 转到 Azure 门户。 如果在“先决条件”部分中创建的 [产品名称] 资源已成功部署，请单击“后续步骤”下的“转到资源”按钮  。 在资源的“密钥和终结点”页的“资源管理”下可以找到密钥和终结点 。 
>
> 完成后，请记住将密钥从代码中删除，并且永远不要公开发布该密钥。 对于生产环境，请考虑使用安全的方法来存储和访问凭据。 有关详细信息，请参阅认知服务[安全性](/cognitive-services/cognitive-services-security)文章。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

在应用程序的“Main”方法中，添加对本快速入门中使用的方法的调用。 稍后将实现这些操作。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

## <a name="object-model"></a>对象模型

以下类和接口将处理人脸 .NET 客户端库的某些主要功能：

|名称|说明|
|---|---|
|[FaceClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient?view=azure-dotnet) | 此类代表使用人脸服务的授权，使用所有人脸功能时都需要用到它。 请使用你的订阅信息实例化此类，然后使用它来生成其他类的实例。 |
|[FaceOperations](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperations?view=azure-dotnet)|此类处理可对人脸执行的基本检测和识别任务。 |
|[DetectedFace](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface?view=azure-dotnet)|此类代表已从图像中的单个人脸检测到的所有数据。 可以使用它来检索有关人脸的详细信息。|
|[FaceListOperations](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.facelistoperations?view=azure-dotnet)|此类管理云中存储的 **FaceList** 构造，这些构造存储各种不同的人脸。 |
|[PersonGroupPersonExtensions](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongrouppersonextensions?view=azure-dotnet)| 此类管理云中存储的 **Person** 构造，这些构造存储属于单个人员的一组人脸。|
|[PersonGroupOperations](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperations?view=azure-dotnet)| 此类管理云中存储的 **PersonGroup** 构造，这些构造存储各种不同的 **Person** 对象。 |

## <a name="code-examples"></a>代码示例

以下代码片段演示如何使用适用于 .NET 的人脸客户端库执行以下任务：

* [对客户端进行身份验证](#authenticate-the-client)
* [检测图像中的人脸](#detect-faces-in-an-image)
* [查找相似人脸](#find-similar-faces)
* [创建和训练人员组](#create-and-train-a-person-group)
* [识别人脸](#identify-a-face)

## <a name="authenticate-the-client"></a>验证客户端

在新方法中，使用终结点和密钥实例化客户端。 使用密钥创建一个 **[ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.apikeyserviceclientcredentials?view=azure-dotnet)** 对象，并在终结点中使用该对象创建一个 **[FaceClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient?view=azure-dotnet)** 对象。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

### <a name="declare-helper-fields"></a>声明帮助程序字段

你稍后将添加的几个人脸操作需要以下字段。 在“Program”类的根目录中定义以下 URL 字符串。 此 URL 指向示例图像的文件夹。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

在“Main”方法中，定义字符串以指向不同的识别模型类型。 稍后，你将能够指定要用于人脸检测的识别模型。 有关这些选项的信息，请参阅[指定识别模型](../../Face-API-How-to-Topics/specify-recognition-model.md)。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

## <a name="detect-faces-in-an-image"></a>在图像中检测人脸

### <a name="get-detected-face-objects"></a>获取检测到的人脸对象

创建新方法以检测人脸。 `DetectFaceExtract` 方法处理给定 URL 处的三个图像，并在程序内存中创建 [DetectedFace](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface?view=azure-dotnet) 对象的列表。 **[FaceAttributeType](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype?view=azure-dotnet)** 值列表指定要提取的特征。 

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

### <a name="display-detected-face-data"></a>显示检测到的人脸数据

`DetectFaceExtract` 方法的其余部分将分析和打印每个检测到的人脸的属性数据。 每个属性必须在原始人脸检测 API 调用中单独指定（在 **[FaceAttributeType](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype?view=azure-dotnet)** 列表中）。 下面的代码处理每个属性，但你可能只需要使用一个或一些属性。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

## <a name="find-similar-faces"></a>查找相似人脸

以下代码采用检测到的单个人脸（源），并搜索其他一组人脸（目标），以找到匹配项（按图像进行人脸搜索）。 找到匹配项后，它会将匹配的人脸的 ID 输出到控制台。

### <a name="detect-faces-for-comparison"></a>检测人脸以进行比较

首先定义另一个人脸检测方法。 需要先检测图像中的人脸，然后才能对其进行比较；此检测方法已针对比较操作进行优化。 它不会提取以上部分所示的详细人脸属性，而是使用另一个识别模型。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

### <a name="find-matches"></a>查找匹配项

以下方法检测一组目标图像和单个源图像中的人脸。 然后，它将比较这些人脸，并查找与源图像类似的所有目标图像。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

### <a name="print-matches"></a>输出匹配项

以下代码将匹配详细信息输出到控制台：

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

## <a name="identify-a-face"></a>识别人脸

识别操作采用一个（或多个）人员的图像，并在图像中查找每个人脸的标识（人脸识别搜索）。 它将每个检测到的人脸与某个 **PersonGroup** （面部特征已知的不同 **Person** 对象的数据库）进行比较。 为了执行“识别”操作，你首先需要创建并训练 PersonGroup

### <a name="create-a-person-group"></a>创建人员组

以下代码创建包含六个不同 **Person** 对象的 **PersonGroup** 。 它将每个 **Person** 与一组示例图像相关联，然后进行训练以按面部特征识别每个人。 **Person** 和 **PersonGroup** 对象在验证、识别和分组操作中使用。

在类的根目录中声明一个字符串变量，用于表示要创建的 **PersonGroup** 的 ID。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

在新方法中添加以下代码。 此方法将执行“识别”操作。 第一个代码块将人员的姓名与其示例图像相关联。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

请注意，此代码定义 `sourceImageFileName` 变量。 此变量对应于源图像 &mdash; 包含要识别的人员的图像。

接下来添加以下代码，以便为字典中的每个人员创建一个 **Person** 对象，并从相应的图像添加人脸数据。 每个 **Person** 对象通过其唯一 ID 字符串来与同一个 **PersonGroup** 相关联。 请记得将变量 `client`、 `url` 和 `RECOGNITION_MODEL1` 传入此方法。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

### <a name="train-the-persongroup"></a>训练 PersonGroup

从图像中提取人脸数据并将其分类成不同的 **Person** 对象后，必须训练 **PersonGroup** 才能识别与其每个 **Person** 对象关联的视觉特征。 以下代码调用异步 **train** 方法并轮询结果，然后将状态输出到控制台。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

现已准备好在验证、识别或分组操作中使用此 **Person** 组及其关联的 **Person** 对象。

### <a name="identify-faces"></a>标识人脸

以下代码采用源映像，创建在图像中检测到的所有人脸的列表。 将会根据 **PersonGroup** 识别这些人脸。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

下一代码片段将调用 IdentifyAsync 操作，并将结果输出到控制台。 此处，服务会尝试将源图像中的每个人脸与给定 **PersonGroup** 中的某个 **Person** 进行匹配。 Identify 方法就此结束。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

## <a name="run-the-application"></a>运行应用程序

#### <a name="visual-studio-ide"></a>[Visual Studio IDE](#tab/visual-studio)

单击 IDE 窗口顶部的“调试”按钮，运行应用程序。

#### <a name="cli"></a>[CLI](#tab/cli)

从应用程序目录使用 `dotnet run` 命令运行应用程序。

```dotnet
dotnet run
```

---

## <a name="clean-up-resources"></a>清理资源

如果想要清理并删除认知服务订阅，可以删除资源或资源组。 删除资源组同时也会删除与之相关联的任何其他资源。

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

如果你在本快速入门中创建了 **PersonGroup** 并想要删除它，请在程序中运行以下代码：

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

使用以下代码定义删除方法：

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
// </snippet_using>

/**
 * FACE QUICKSTART
 * 
 * This quickstart includes the following examples for Face:
 *  - Detect Faces
 *  - Find Similar
 *  - Identify faces (and person group operations)
 *  - Large Person Group 
 *  - Group Faces
 *  - FaceList
 *  - Large FaceList
 * 
 * Prerequisites:
 *  - Visual Studio 2019 (or 2017, but this is app uses .NETCore, not .NET Framework)
 *  - NuGet libraries:
 *    Microsoft.Azure.CognitiveServices.Vision.Face
 *    
 * How to run:
 *  - Create a new C# Console app in Visual Studio 2019.
 *  - Copy/paste the Program.cs file in the Github quickstart into your own Program.cs file. 
 *  
 * Dependencies within the samples: 
 *  - Authenticate produces a client that's used by all samples.
 *  - Detect Faces is a helper function that is used by several other samples. 
 *   
 * References:
 *  - Face Documentation: /cognitive-services/face/
 *  - .NET SDK: https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/face?view=azure-dotnet
 *  - API Reference: /cognitive-services/face/apireference
 */

namespace FaceQuickstart
{
    class Program
    {
        // Used for the Identify and Delete examples.
        // <snippet_persongroup_declare>
        static string personGroupId = Guid.NewGuid().ToString();
        // </snippet_persongroup_declare>

        // <snippet_image_url>
        // Used for all examples.
        // URL for the images.
        const string IMAGE_BASE_URL = "https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/";
        // </snippet_image_url>

        // <snippet_creds>
        // From your Face subscription in the Azure portal, get your subscription key and endpoint.
        const string SUBSCRIPTION_KEY = "<your subscription key>";
        const string ENDPOINT = "<your api endpoint>";
        // </snippet_creds>

        static void Main(string[] args)
        {
           
            // <snippet_detect_models>
            // Recognition model 3 was released in 2020 May.
            // It is recommended since its overall accuracy is improved
            // compared with models 1 and 2.
            const string RECOGNITION_MODEL3 = RecognitionModel.Recognition03;
            // </snippet_detect_models>

            // Large FaceList variables
            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, "_" or "-" characters
            const string LargeFaceListName = "MyLargeFaceListName";

            // <snippet_maincalls>
            // Authenticate.
            IFaceClient client = Authenticate(ENDPOINT, SUBSCRIPTION_KEY);
            // </snippet_client>

            // Detect - get features from faces.
            DetectFaceExtract(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Find Similar - find a similar face from a list of faces.
            FindSimilar(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Verify - compare two images if the same person or not.
            Verify(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();

            // Identify - recognize a face(s) in a person group (a person group is created in this example).
            IdentifyInPersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // LargePersonGroup - create, then get data.
            LargePersonGroup(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // Group faces - automatically group similar faces.
            Group(client, IMAGE_BASE_URL, RECOGNITION_MODEL3).Wait();
            // FaceList - create a face list, then get data
            // </snippet_maincalls>

            FaceListOperations(client, IMAGE_BASE_URL).Wait();
            // Large FaceList - create a large face list, then get data
            LargeFaceListOperations(client, IMAGE_BASE_URL).Wait();

            // <snippet_persongroup_delete>
            // At end, delete person groups in both regions (since testing only)
            Console.WriteLine("========DELETE PERSON GROUP========");
            Console.WriteLine();
            DeletePersonGroup(client, personGroupId).Wait();
            // </snippet_persongroup_delete>

            Console.WriteLine("End of quickstart.");
        }

        // <snippet_auth>
        /*
         *  AUTHENTICATE
         *  Uses subscription key and region to create a client.
         */
        public static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
        // </snippet_auth>
        /*
         * END - Authenticate
         */

        // <snippet_detect>
        /* 
         * DETECT FACES
         * Detects features from faces and IDs them.
         */
        public static async Task DetectFaceExtract(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========DETECT FACES========");
            Console.WriteLine();

            // Create a list of images
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                // "detection2.jpg", // (optional: single man)
                                // "detection3.jpg", // (optional: single male construction worker)
                                // "detection4.jpg", // (optional: 3 people at cafe, 1 is blurred)
                                "detection5.jpg",    // family, woman child man
                                "detection6.jpg"     // elderly couple, male female
                            };

            foreach (var imageFileName in imageFileNames)
            {
                IList<DetectedFace> detectedFaces;

                // Detect faces with all attributes from image url.
                detectedFaces = await client.Face.DetectWithUrlAsync($"{url}{imageFileName}",
                        returnFaceAttributes: new List<FaceAttributeType?> { FaceAttributeType.Accessories, FaceAttributeType.Age,
                        FaceAttributeType.Blur, FaceAttributeType.Emotion, FaceAttributeType.Exposure, FaceAttributeType.FacialHair,
                        FaceAttributeType.Gender, FaceAttributeType.Glasses, FaceAttributeType.Hair, FaceAttributeType.HeadPose,
                        FaceAttributeType.Makeup, FaceAttributeType.Noise, FaceAttributeType.Occlusion, FaceAttributeType.Smile },
                        // We specify detection model 1 because we are retrieving attributes.
                        detectionModel: DetectionModel.Detection01,
                        recognitionModel: recognitionModel);

                Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{imageFileName}`.");
                // </snippet_detect>
                // <snippet_detect_parse>
                // Parse and print all attributes of each detected face.
                foreach (var face in detectedFaces)
                {
                    Console.WriteLine($"Face attributes for {imageFileName}:");

                    // Get bounding box of the faces
                    Console.WriteLine($"Rectangle(Left/Top/Width/Height) : {face.FaceRectangle.Left} {face.FaceRectangle.Top} {face.FaceRectangle.Width} {face.FaceRectangle.Height}");

                    // Get accessories of the faces
                    List<Accessory> accessoriesList = (List<Accessory>)face.FaceAttributes.Accessories;
                    int count = face.FaceAttributes.Accessories.Count;
                    string accessory; string[] accessoryArray = new string[count];
                    if (count == 0) { accessory = "NoAccessories"; }
                    else
                    {
                        for (int i = 0; i < count; ++i) { accessoryArray[i] = accessoriesList[i].Type.ToString(); }
                        accessory = string.Join(",", accessoryArray);
                    }
                    Console.WriteLine($"Accessories : {accessory}");

                    // Get face other attributes
                    Console.WriteLine($"Age : {face.FaceAttributes.Age}");
                    Console.WriteLine($"Blur : {face.FaceAttributes.Blur.BlurLevel}");

                    // Get emotion on the face
                    string emotionType = string.Empty;
                    double emotionValue = 0.0;
                    Emotion emotion = face.FaceAttributes.Emotion;
                    if (emotion.Anger > emotionValue) { emotionValue = emotion.Anger; emotionType = "Anger"; }
                    if (emotion.Contempt > emotionValue) { emotionValue = emotion.Contempt; emotionType = "Contempt"; }
                    if (emotion.Disgust > emotionValue) { emotionValue = emotion.Disgust; emotionType = "Disgust"; }
                    if (emotion.Fear > emotionValue) { emotionValue = emotion.Fear; emotionType = "Fear"; }
                    if (emotion.Happiness > emotionValue) { emotionValue = emotion.Happiness; emotionType = "Happiness"; }
                    if (emotion.Neutral > emotionValue) { emotionValue = emotion.Neutral; emotionType = "Neutral"; }
                    if (emotion.Sadness > emotionValue) { emotionValue = emotion.Sadness; emotionType = "Sadness"; }
                    if (emotion.Surprise > emotionValue) { emotionType = "Surprise"; }
                    Console.WriteLine($"Emotion : {emotionType}");

                    // Get more face attributes
                    Console.WriteLine($"Exposure : {face.FaceAttributes.Exposure.ExposureLevel}");
                    Console.WriteLine($"FacialHair : {string.Format("{0}", face.FaceAttributes.FacialHair.Moustache + face.FaceAttributes.FacialHair.Beard + face.FaceAttributes.FacialHair.Sideburns > 0 ? "Yes" : "No")}");
                    Console.WriteLine($"Gender : {face.FaceAttributes.Gender}");
                    Console.WriteLine($"Glasses : {face.FaceAttributes.Glasses}");

                    // Get hair color
                    Hair hair = face.FaceAttributes.Hair;
                    string color = null;
                    if (hair.HairColor.Count == 0) { if (hair.Invisible) { color = "Invisible"; } else { color = "Bald"; } }
                    HairColorType returnColor = HairColorType.Unknown;
                    double maxConfidence = 0.0f;
                    foreach (HairColor hairColor in hair.HairColor)
                    {
                        if (hairColor.Confidence <= maxConfidence) { continue; }
                        maxConfidence = hairColor.Confidence; returnColor = hairColor.Color; color = returnColor.ToString();
                    }
                    Console.WriteLine($"Hair : {color}");

                    // Get more attributes
                    Console.WriteLine($"HeadPose : {string.Format("Pitch: {0}, Roll: {1}, Yaw: {2}", Math.Round(face.FaceAttributes.HeadPose.Pitch, 2), Math.Round(face.FaceAttributes.HeadPose.Roll, 2), Math.Round(face.FaceAttributes.HeadPose.Yaw, 2))}");
                    Console.WriteLine($"Makeup : {string.Format("{0}", (face.FaceAttributes.Makeup.EyeMakeup || face.FaceAttributes.Makeup.LipMakeup) ? "Yes" : "No")}");
                    Console.WriteLine($"Noise : {face.FaceAttributes.Noise.NoiseLevel}");
                    Console.WriteLine($"Occlusion : {string.Format("EyeOccluded: {0}", face.FaceAttributes.Occlusion.EyeOccluded ? "Yes" : "No")} " +
                        $" {string.Format("ForeheadOccluded: {0}", face.FaceAttributes.Occlusion.ForeheadOccluded ? "Yes" : "No")}   {string.Format("MouthOccluded: {0}", face.FaceAttributes.Occlusion.MouthOccluded ? "Yes" : "No")}");
                    Console.WriteLine($"Smile : {face.FaceAttributes.Smile}");
                    Console.WriteLine();
                }
            }
        }
        // </snippet_detect_parse>

        // Detect faces from image url for recognition purpose. This is a helper method for other functions in this quickstart.
        // Parameter `returnFaceId` of `DetectWithUrlAsync` must be set to `true` (by default) for recognition purpose.
        // The field `faceId` in returned `DetectedFace`s will be used in Face - Find Similar, Face - Verify. and Face - Identify.
        // It will expire 24 hours after the detection call.
        // <snippet_face_detect_recognize>
        private static async Task<List<DetectedFace>> DetectFaceRecognize(IFaceClient faceClient, string url, string recognition_model)
        {
            // Detect faces from image URL. Since only recognizing, use the recognition model 1.
            // We use detection model 2 because we are not retrieving attributes.
            IList<DetectedFace> detectedFaces = await faceClient.Face.DetectWithUrlAsync(url, recognitionModel: recognition_model, detectionModel: DetectionModel.Detection02);
            Console.WriteLine($"{detectedFaces.Count} face(s) detected from image `{Path.GetFileName(url)}`");
            return detectedFaces.ToList();
        }
        // </snippet_face_detect_recognize>
        /*
         * END - DETECT FACES 
         */

        // <snippet_find_similar>
        /*
         * FIND SIMILAR
         * This example will take an image and find a similar one to it in another image.
         */
        public static async Task FindSimilar(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========FIND SIMILAR========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            string sourceImageFileName = "findsimilar.jpg";
            IList<Guid?> targetFaceIds = new List<Guid?>();
            foreach (var targetImageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                var faces = await DetectFaceRecognize(client, $"{url}{targetImageFileName}", recognition_model);
                // Add detected faceId to list of GUIDs.
                targetFaceIds.Add(faces[0].FaceId.Value);
            }

            // Detect faces from source image url.
            IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognition_model);
            Console.WriteLine();

            // Find a similar face(s) in the list of IDs. Comapring only the first in list for testing purposes.
            IList<SimilarFace> similarResults = await client.Face.FindSimilarAsync(detectedFaces[0].FaceId.Value, null, null, targetFaceIds);
            // </snippet_find_similar>
            // <snippet_find_similar_print>
            foreach (var similarResult in similarResults)
            {
                Console.WriteLine($"Faces from {sourceImageFileName} & ID:{similarResult.FaceId} are similar with confidence: {similarResult.Confidence}.");
            }
            Console.WriteLine();
            // </snippet_find_similar_print>
        }
        /*
         * END - FIND SIMILAR 
         */

        /*
         * VERIFY
         * The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID 
         * or a Person object and determines whether they belong to the same person. If you pass in a Person object, 
         * you can optionally pass in a PersonGroup to which that Person belongs to improve performance.
         */
        public static async Task Verify(IFaceClient client, string url, string recognitionModel03)
        {
            Console.WriteLine("========VERIFY========");
            Console.WriteLine();

            List<string> targetImageFileNames = new List<string> { "Family1-Dad1.jpg", "Family1-Dad2.jpg" };
            string sourceImageFileName1 = "Family1-Dad3.jpg";
            string sourceImageFileName2 = "Family1-Son1.jpg";


            List<Guid> targetFaceIds = new List<Guid>();
            foreach (var imageFileName in targetImageFileNames)
            {
                // Detect faces from target image url.
                List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName} ", recognitionModel03);
                targetFaceIds.Add(detectedFaces[0].FaceId.Value);
                Console.WriteLine($"{detectedFaces.Count} faces detected from image `{imageFileName}`.");
            }

            // Detect faces from source image file 1.
            List<DetectedFace> detectedFaces1 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName1} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces1.Count} faces detected from image `{sourceImageFileName1}`.");
            Guid sourceFaceId1 = detectedFaces1[0].FaceId.Value;

            // Detect faces from source image file 2.
            List<DetectedFace> detectedFaces2 = await DetectFaceRecognize(client, $"{url}{sourceImageFileName2} ", recognitionModel03);
            Console.WriteLine($"{detectedFaces2.Count} faces detected from image `{sourceImageFileName2}`.");
            Guid sourceFaceId2 = detectedFaces2[0].FaceId.Value;

            // Verification example for faces of the same person.
            VerifyResult verifyResult1 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId1, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult1.IsIdentical
                    ? $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of the same (Positive) person, similarity confidence: {verifyResult1.Confidence}."
                    : $"Faces from {sourceImageFileName1} & {targetImageFileNames[0]} are of different (Negative) persons, similarity confidence: {verifyResult1.Confidence}.");

            // Verification example for faces of different persons.
            VerifyResult verifyResult2 = await client.Face.VerifyFaceToFaceAsync(sourceFaceId2, targetFaceIds[0]);
            Console.WriteLine(
                verifyResult2.IsIdentical
                    ? $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of the same (Negative) person, similarity confidence: {verifyResult2.Confidence}."
                    : $"Faces from {sourceImageFileName2} & {targetImageFileNames[0]} are of different (Positive) persons, similarity confidence: {verifyResult2.Confidence}.");

            Console.WriteLine();
        }
        /*
         * END - VERIFY 
         */

        /*
         * IDENTIFY FACES
         * To identify faces, you need to create and define a person group.
         * The Identify operation takes one or several face IDs from DetectedFace or PersistedFace and a PersonGroup and returns 
         * a list of Person objects that each face might belong to. Returned Person objects are wrapped as Candidate objects, 
         * which have a prediction confidence value.
         */
        // <snippet_persongroup_files>
        public static async Task IdentifyInPersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========IDENTIFY FACES========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
                new Dictionary<string, string[]>
                    { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                    };
            // A group photo that includes some of the persons you seek to identify from your dictionary.
            string sourceImageFileName = "identification1.jpg";
            // </snippet_persongroup_files>

            // <snippet_persongroup_create>
            // Create a person group. 
            Console.WriteLine($"Create a person group ({personGroupId}).");
            await client.PersonGroup.CreateAsync(personGroupId, personGroupId, recognitionModel: recognitionModel);
            // The similar faces will be grouped into a single person group person.
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);
                Person person = await client.PersonGroupPerson.CreateAsync(personGroupId: personGroupId, name: groupedFace);
                Console.WriteLine($"Create a person group person '{groupedFace}'.");

                // Add face to the person group person.
                foreach (var similarImage in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person({groupedFace}) from image `{similarImage}`");
                    PersistedFace face = await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, person.PersonId,
                        $"{url}{similarImage}", similarImage);
                }
            }
            // </snippet_persongroup_create>

            // <snippet_persongroup_train>
            // Start to train the person group.
            Console.WriteLine();
            Console.WriteLine($"Train person group {personGroupId}.");
            await client.PersonGroup.TrainAsync(personGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.PersonGroup.GetTrainingStatusAsync(personGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // </snippet_persongroup_train>
            // <snippet_identify_sources>
            List<Guid?> sourceFaceIds = new List<Guid?>();
            // Detect faces from source image url.
            List<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{sourceImageFileName}", recognitionModel);

            // Add detected faceId to sourceFaceIds.
            foreach (var detectedFace in detectedFaces) { sourceFaceIds.Add(detectedFace.FaceId.Value); }
            // </snippet_identify_sources>
            
            // <snippet_identify>
            // Identify the faces in a person group. 
            var identifyResults = await client.Face.IdentifyAsync(sourceFaceIds, personGroupId);

            foreach (var identifyResult in identifyResults)
            {
                Person person = await client.PersonGroupPerson.GetAsync(personGroupId, identifyResult.Candidates[0].PersonId);
                Console.WriteLine($"Person '{person.Name}' is identified for face in: {sourceImageFileName} - {identifyResult.FaceId}," +
                    $" confidence: {identifyResult.Candidates[0].Confidence}.");
            }
            Console.WriteLine();
        }
        // </snippet_identify>

        /*
         * END - IDENTIFY FACES
         */

        /*
         * LARGE PERSON GROUP
         * The example will create a large person group, retrieve information from it, 
         * list the Person IDs it contains, and finally delete a large person group.
         * For simplicity, the same images are used for the regular-sized person group in IDENTIFY FACES of this quickstart.
         * A large person group is made up of person group persons. 
         * One person group person is made up of many similar images of that person, which are each PersistedFace objects.
         */
        public static async Task LargePersonGroup(IFaceClient client, string url, string recognitionModel)
        {
            Console.WriteLine("========LARGE PERSON GROUP========");
            Console.WriteLine();

            // Create a dictionary for all your images, grouping similar ones under the same key.
            Dictionary<string, string[]> personDictionary =
            new Dictionary<string, string[]>
                { { "Family1-Dad", new[] { "Family1-Dad1.jpg", "Family1-Dad2.jpg" } },
                      { "Family1-Mom", new[] { "Family1-Mom1.jpg", "Family1-Mom2.jpg" } },
                      { "Family1-Son", new[] { "Family1-Son1.jpg", "Family1-Son2.jpg" } },
                      { "Family1-Daughter", new[] { "Family1-Daughter1.jpg", "Family1-Daughter2.jpg" } },
                      { "Family2-Lady", new[] { "Family2-Lady1.jpg", "Family2-Lady2.jpg" } },
                      { "Family2-Man", new[] { "Family2-Man1.jpg", "Family2-Man2.jpg" } }
                };

            // Create a large person group ID. 
            string largePersonGroupId = Guid.NewGuid().ToString();
            Console.WriteLine($"Create a large person group ({largePersonGroupId}).");

            // Create the large person group
            await client.LargePersonGroup.CreateAsync(largePersonGroupId: largePersonGroupId, name: largePersonGroupId, recognitionModel);

            // Create Person objects from images in our dictionary
            // We'll store their IDs in the process
            List<Guid> personIds = new List<Guid>();
            foreach (var groupedFace in personDictionary.Keys)
            {
                // Limit TPS
                await Task.Delay(250);

                Person personLarge = await client.LargePersonGroupPerson.CreateAsync(largePersonGroupId, groupedFace);
                Console.WriteLine();
                Console.WriteLine($"Create a large person group person '{groupedFace}' ({personLarge.PersonId}).");

                // Store these IDs for later retrieval
                personIds.Add(personLarge.PersonId);

                // Add face to the large person group person.
                foreach (var image in personDictionary[groupedFace])
                {
                    Console.WriteLine($"Add face to the person group person '{groupedFace}' from image `{image}`");
                    PersistedFace face = await client.LargePersonGroupPerson.AddFaceFromUrlAsync(largePersonGroupId, personLarge.PersonId,
                        $"{url}{image}", image);
                }
            }

            // Start to train the large person group.
            Console.WriteLine();
            Console.WriteLine($"Train large person group {largePersonGroupId}.");
            await client.LargePersonGroup.TrainAsync(largePersonGroupId);

            // Wait until the training is completed.
            while (true)
            {
                await Task.Delay(1000);
                var trainingStatus = await client.LargePersonGroup.GetTrainingStatusAsync(largePersonGroupId);
                Console.WriteLine($"Training status: {trainingStatus.Status}.");
                if (trainingStatus.Status == TrainingStatusType.Succeeded) { break; }
            }
            Console.WriteLine();

            // Now that we have created and trained a large person group, we can retrieve data from it.
            // Get list of persons and retrieve data, starting at the first Person ID in previously saved list.
            IList<Person> persons = await client.LargePersonGroupPerson.ListAsync(largePersonGroupId, start: "");

            Console.WriteLine($"Persisted Face IDs (from {persons.Count} large person group persons): ");
            foreach (Person person in persons)
            {
                foreach (Guid pFaceId in person.PersistedFaceIds)
                {
                    Console.WriteLine($"The person '{person.Name}' has an image with ID: {pFaceId}");
                }
            }
            Console.WriteLine();

            // After testing, delete the large person group, PersonGroupPersons also get deleted.
            await client.LargePersonGroup.DeleteAsync(largePersonGroupId);
            Console.WriteLine($"Deleted the large person group {largePersonGroupId}.");
            Console.WriteLine();
        }
        /*
         * END - LARGE PERSON GROUP
         */

        /*
         * GROUP FACES
         * This method of grouping is useful if you don't need to create a person group. It will automatically group similar
         * images, whereas the person group method allows you to define the grouping.
         * A single "messyGroup" array contains face IDs for which no similarities were found.
         */
        public static async Task Group(IFaceClient client, string url, string recognition_model)
        {
            Console.WriteLine("========GROUP FACES========");
            Console.WriteLine();

            // Create list of image names
            List<string> imageFileNames = new List<string>
                              {
                                  "Family1-Dad1.jpg",
                                  "Family1-Dad2.jpg",
                                  "Family3-Lady1.jpg",
                                  "Family1-Daughter1.jpg",
                                  "Family1-Daughter2.jpg",
                                  "Family1-Daughter3.jpg"
                              };
            // Create empty dictionary to store the groups
            Dictionary<string, string> faces = new Dictionary<string, string>();
            List<Guid?> faceIds = new List<Guid?>();

            // First, detect the faces in your images
            foreach (var imageFileName in imageFileNames)
            {
                // Detect faces from image url.
                IList<DetectedFace> detectedFaces = await DetectFaceRecognize(client, $"{url}{imageFileName}", recognition_model);
                // Add detected faceId to faceIds and faces.
                faceIds.Add(detectedFaces[0].FaceId.Value);
                faces.Add(detectedFaces[0].FaceId.ToString(), imageFileName);
            }
            Console.WriteLine();
            // Group the faces. Grouping result is a group collection, each group contains similar faces.
            var groupResult = await client.Face.GroupAsync(faceIds);

            // Face groups contain faces that are similar to all members of its group.
            for (int i = 0; i < groupResult.Groups.Count; i++)
            {
                Console.Write($"Found face group {i + 1}: ");
                foreach (var faceId in groupResult.Groups[i]) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }

            // MessyGroup contains all faces which are not similar to any other faces. The faces that cannot be grouped.
            if (groupResult.MessyGroup.Count > 0)
            {
                Console.Write("Found messy face group: ");
                foreach (var faceId in groupResult.MessyGroup) { Console.Write($"{faces[faceId.ToString()]} "); }
                Console.WriteLine(".");
            }
            Console.WriteLine();
        }
        /*
         * END - GROUP FACES
         */

        /*
         * FACELIST OPERATIONS
         * Create a face list and add single-faced images to it, then retrieve data from the faces.
         * Images are used from URLs.
         */
        public static async Task FaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("========FACELIST OPERATIONS========");
            Console.WriteLine();

            const string FaceListId = "myfacelistid_001";
            const string FaceListName = "MyFaceListName";

            // Create an empty FaceList with user-defined specifications, it gets stored in the client.
            await client.FaceList.CreateAsync(faceListId: FaceListId, name: FaceListName);

            // Create a list of single-faced images to append to base URL. Images with mulitple faces are not accepted.
            List<string> imageFileNames = new List<string>
                            {
                                "detection1.jpg",    // single female with glasses
                                "detection2.jpg",    // single male
                                "detection3.jpg",    // single male construction worker
                            };

            // Add Faces to the FaceList.
            foreach (string image in imageFileNames)
            {
                string urlFull = baseUrl + image;
                // Returns a Task<PersistedFace> which contains a GUID, and is stored in the client.
                await client.FaceList.AddFaceFromUrlAsync(faceListId: FaceListId, url: urlFull);
            }

            // Print the face list
            Console.WriteLine("Face IDs from the face list: ");
            Console.WriteLine();

            // List the IDs of each stored image
            FaceList faceList = await client.FaceList.GetAsync(FaceListId);

            foreach (PersistedFace face in faceList.PersistedFaces)
            {
                Console.WriteLine(face.PersistedFaceId);
            }

            // Delete the face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.FaceList.DeleteAsync(FaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the face list.");
            Console.WriteLine();
        }
        /*
         * END - FACELIST OPERATIONS
         */

        /*
        * LARGE FACELIST OPERATIONS
        * Create a large face list and adds single-faced images to it, then retrieve data from the faces.
        * Images are used from URLs. Large face lists are preferred for scale, up to 1 million images.
        */
        public static async Task LargeFaceListOperations(IFaceClient client, string baseUrl)
        {
            Console.WriteLine("======== LARGE FACELIST OPERATIONS========");
            Console.WriteLine();

            const string LargeFaceListId = "mylargefacelistid_001"; // must be lowercase, 0-9, or "_"
            const string LargeFaceListName = "MyLargeFaceListName";
            const int timeIntervalInMilliseconds = 1000; // waiting time in training

            List<string> singleImages = new List<string>
                                {
                                    "Family1-Dad1.jpg",
                                    "Family1-Daughter1.jpg",
                                    "Family1-Mom1.jpg",
                                    "Family1-Son1.jpg",
                                    "Family2-Lady1.jpg",
                                    "Family2-Man1.jpg",
                                    "Family3-Lady1.jpg",
                                    "Family3-Man1.jpg"
                                };

            // Create a large face list
            Console.WriteLine("Creating a large face list...");
            await client.LargeFaceList.CreateAsync(largeFaceListId: LargeFaceListId, name: LargeFaceListName);

            // Add Faces to the LargeFaceList.
            Console.WriteLine("Adding faces to a large face list...");
            foreach (string image in singleImages)
            {
                // Returns a PersistedFace which contains a GUID.
                await client.LargeFaceList.AddFaceFromUrlAsync(largeFaceListId: LargeFaceListId, url: $"{baseUrl}{image}");
            }

            // Training a LargeFaceList is what sets it apart from a regular FaceList.
            // You must train before using the large face list, for example to use the Find Similar operations.
            Console.WriteLine("Training a large face list...");
            await client.LargeFaceList.TrainAsync(LargeFaceListId);

            // Wait for training finish.
            while (true)
            {
                Task.Delay(timeIntervalInMilliseconds).Wait();
                var status = await client.LargeFaceList.GetTrainingStatusAsync(LargeFaceListId);

                if (status.Status == TrainingStatusType.Running)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    continue;
                }
                else if (status.Status == TrainingStatusType.Succeeded)
                {
                    Console.WriteLine($"Training status: {status.Status}");
                    break;
                }
                else
                {
                    throw new Exception("The train operation has failed!");
                }
            }

            // Print the large face list
            Console.WriteLine();
            Console.WriteLine("Face IDs from the large face list: ");
            Console.WriteLine();
            Parallel.ForEach(
                    await client.LargeFaceList.ListFacesAsync(LargeFaceListId),
                    faceId =>
                    {
                        Console.WriteLine(faceId.PersistedFaceId);
                    }
                );

            // Delete the large face list, for repetitive testing purposes (cannot recreate list with same name).
            await client.LargeFaceList.DeleteAsync(LargeFaceListId);
            Console.WriteLine();
            Console.WriteLine("Deleted the large face list.");
            Console.WriteLine();
        }
        /*
        * END - LARGE FACELIST OPERATIONS
        */

        // <snippet_deletepersongroup>
        /*
         * DELETE PERSON GROUP
         * After this entire example is executed, delete the person group in your Azure account,
         * otherwise you cannot recreate one with the same name (if running example repeatedly).
         */
        public static async Task DeletePersonGroup(IFaceClient client, String personGroupId)
        {
            await client.PersonGroup.DeleteAsync(personGroupId);
            Console.WriteLine($"Deleted the person group {personGroupId}.");
        }
        // </snippet_deletepersongroup>
        /*
         * END - DELETE PERSON GROUP
         */
    }
}
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已了解如何使用适用于 .NET 的人脸客户端库来执行基本人脸识别任务。 接下来，请在参考文档中详细了解该库。

> [!div class="nextstepaction"]
> [人脸 API 参考 (.NET)](https://docs.microsoft.com/dotnet/api/overview/cognitiveservices/client/faceapi?view=azure-dotnet)

* [什么是人脸服务？](../../overview.md)
* 可以在 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/Face/FaceQuickstart.cs) 上找到此示例的源代码。

