---
title: 快速入门：使用 Azure REST API 和 Java 检测图像中的人脸
titleSuffix: Azure Cognitive Services
description: 在本快速入门中，请使用 Azure 人脸 REST API 和 Java 检测图像中的人脸。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: quickstart
origin.date: 07/03/2019
ms.date: 10/27/2020
ms.custom: devx-track-java
ms.author: v-johya
ms.openlocfilehash: 0e489da00dc4af5003f45d80bedb83cac45d7ac9
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104087"
---
# <a name="quickstart-detect-faces-in-an-image-using-the-rest-api-and-java"></a>快速入门：使用 REST API 和 Java 检测图像中的人脸

本快速入门将通过 Java 使用 Azure 人脸 REST API 来检测图像中的人脸。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/details/cognitive-services/)。 

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [免费创建订阅](https://www.azure.cn/pricing/details/cognitive-services/)
* 拥有 Azure 订阅后，在 Azure 门户中<a href="https://portal.azure.cn/#create/Microsoft.CognitiveServicesFace"  title="创建人脸资源"  target="_blank">创建人脸资源 <span class="docon docon-navigate-external x-hidden-focus"></span></a>，获取密钥和终结点。 部署后，单击“转到资源”。
    * 需要从创建的资源获取密钥和终结点，以便将应用程序连接到人脸 API。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    * 可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。
* 任何所选的 Java IDE。

## <a name="create-the-java-project"></a>创建 Java 项目

1. 在 IDE 中创建新的命令行 Java 应用，并添加包含 **main** 方法的 **Main** 类。
1. 将以下库导入到你的 Java 项目中。 如果使用 Maven，则为每个库提供 Maven 坐标。
   - [Apache HTTP 客户端](https://hc.apache.org/downloads.cgi) (org.apache.httpcomponents:httpclient:4.5.6)
   - [Apache HTTP 核心](https://hc.apache.org/downloads.cgi) (org.apache.httpcomponents:httpcore:4.4.10)
   - [JSON 库](https://github.com/stleary/JSON-java) (org.json:json:20180130)
   - [Apache Commons 日志记录](https://commons.apache.org/proper/commons-logging/download_logging.cgi) (commons-logging:commons-logging:1.1.2)

## <a name="add-face-detection-code"></a>添加人脸检测代码

打开项目的 main 类。 在这里，请添加加载图像和检测人脸所需的代码。

### <a name="import-packages"></a>导入包

将下列 `import` 语句添加到文件顶部。

```java
// <dependencies>
// This sample uses Apache HttpComponents:
// http://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/
// https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/

import java.net.URI;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.util.EntityUtils;
import org.json.JSONArray;
import org.json.JSONObject;
// </dependencies>

// <environment>
/*
 * To compile and run, enter the following at a command prompt:
 *   javac Detect.java -cp .;lib\*
 *   java -cp .;lib\* Detect
 */
public class Detect {
    private static final String subscriptionKey = System.getenv("FACE_SUBSCRIPTION_KEY");
    private static final String endpoint = System.getenv("FACE_ENDPOINT");

    private static final String imageWithFaces =
        "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}";
// </environment>

// <main>
    public static void main(String[] args) {
        HttpClient httpclient = HttpClientBuilder.create().build();

        try
        {
            URIBuilder builder = new URIBuilder(endpoint + "/face/v1.0/detect");

            // Request parameters. All of them are optional.
            builder.setParameter("detectionModel", "detection_02");
            builder.setParameter("returnFaceId", "true");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

            // Request body.
            StringEntity reqEntity = new StringEntity(imageWithFaces);
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();
// </main>

// <print>
            if (entity != null)
            {
                // Format and display the JSON response.
                System.out.println("REST Response:\n");

                String jsonString = EntityUtils.toString(entity).trim();
                if (jsonString.charAt(0) == '[') {
                    JSONArray jsonArray = new JSONArray(jsonString);
                    System.out.println(jsonArray.toString(2));
                }
                else if (jsonString.charAt(0) == '{') {
                    JSONObject jsonObject = new JSONObject(jsonString);
                    System.out.println(jsonObject.toString(2));
                } else {
                    System.out.println(jsonString);
                }
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
        }
    }
}
// </print>
```

### <a name="add-essential-fields"></a>添加必要的字段

将 **Main** 类替换为以下代码。 该数据指定如何连接到人脸服务，以及在何处获取输入数据。 需使用订阅密钥的值更新 `subscriptionKey` 字段，并更改 `uriBase` 字符串，使之包含正确的终结点字符串。 可能还需要将 `imageWithFaces` 值设置为一个路径，使之指向另一图像文件。

[!INCLUDE [subdomains-note](../../../../includes/cognitive-services-custom-subdomains-note.md)]

`faceAttributes` 字段只是一个列表，包含特定类型的属性。 它将指定要检索的有关已检测人脸的信息。

```java
// <dependencies>
// This sample uses Apache HttpComponents:
// http://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/
// https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/

import java.net.URI;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.util.EntityUtils;
import org.json.JSONArray;
import org.json.JSONObject;
// </dependencies>

// <environment>
/*
 * To compile and run, enter the following at a command prompt:
 *   javac Detect.java -cp .;lib\*
 *   java -cp .;lib\* Detect
 */
public class Detect {
    private static final String subscriptionKey = System.getenv("FACE_SUBSCRIPTION_KEY");
    private static final String endpoint = System.getenv("FACE_ENDPOINT");

    private static final String imageWithFaces =
        "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}";
// </environment>

// <main>
    public static void main(String[] args) {
        HttpClient httpclient = HttpClientBuilder.create().build();

        try
        {
            URIBuilder builder = new URIBuilder(endpoint + "/face/v1.0/detect");

            // Request parameters. All of them are optional.
            builder.setParameter("detectionModel", "detection_02");
            builder.setParameter("returnFaceId", "true");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

            // Request body.
            StringEntity reqEntity = new StringEntity(imageWithFaces);
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();
// </main>

// <print>
            if (entity != null)
            {
                // Format and display the JSON response.
                System.out.println("REST Response:\n");

                String jsonString = EntityUtils.toString(entity).trim();
                if (jsonString.charAt(0) == '[') {
                    JSONArray jsonArray = new JSONArray(jsonString);
                    System.out.println(jsonArray.toString(2));
                }
                else if (jsonString.charAt(0) == '{') {
                    JSONObject jsonObject = new JSONObject(jsonString);
                    System.out.println(jsonObject.toString(2));
                } else {
                    System.out.println(jsonString);
                }
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
        }
    }
}
// </print>
```

### <a name="call-the-face-detection-rest-api"></a>调用人脸检测 REST API

通过以下代码添加 **main** 方法。 它构造一个针对人脸 API 的 REST 调用，以便检测远程图像（`faceAttributes` 字符串指定要检索的人脸属性）中的人脸信息。 然后，它将输出数据写入到 JSON 字符串。

```java
// <dependencies>
// This sample uses Apache HttpComponents:
// http://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/
// https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/

import java.net.URI;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.util.EntityUtils;
import org.json.JSONArray;
import org.json.JSONObject;
// </dependencies>

// <environment>
/*
 * To compile and run, enter the following at a command prompt:
 *   javac Detect.java -cp .;lib\*
 *   java -cp .;lib\* Detect
 */
public class Detect {
    private static final String subscriptionKey = System.getenv("FACE_SUBSCRIPTION_KEY");
    private static final String endpoint = System.getenv("FACE_ENDPOINT");

    private static final String imageWithFaces =
        "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}";
// </environment>

// <main>
    public static void main(String[] args) {
        HttpClient httpclient = HttpClientBuilder.create().build();

        try
        {
            URIBuilder builder = new URIBuilder(endpoint + "/face/v1.0/detect");

            // Request parameters. All of them are optional.
            builder.setParameter("detectionModel", "detection_02");
            builder.setParameter("returnFaceId", "true");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

            // Request body.
            StringEntity reqEntity = new StringEntity(imageWithFaces);
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();
// </main>

// <print>
            if (entity != null)
            {
                // Format and display the JSON response.
                System.out.println("REST Response:\n");

                String jsonString = EntityUtils.toString(entity).trim();
                if (jsonString.charAt(0) == '[') {
                    JSONArray jsonArray = new JSONArray(jsonString);
                    System.out.println(jsonArray.toString(2));
                }
                else if (jsonString.charAt(0) == '{') {
                    JSONObject jsonObject = new JSONObject(jsonString);
                    System.out.println(jsonObject.toString(2));
                } else {
                    System.out.println(jsonString);
                }
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
        }
    }
}
// </print>
```

### <a name="parse-the-json-response"></a>分析 JSON 响应

直接在以前的代码下添加以下块，以便将返回的 JSON 数据转换成更加易于读取的格式，然后再将其输出到控制台。 最后，关闭 try-catch 块、 **main** 方法和 **Main** 类。

```java
// <dependencies>
// This sample uses Apache HttpComponents:
// http://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/
// https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/

import java.net.URI;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.util.EntityUtils;
import org.json.JSONArray;
import org.json.JSONObject;
// </dependencies>

// <environment>
/*
 * To compile and run, enter the following at a command prompt:
 *   javac Detect.java -cp .;lib\*
 *   java -cp .;lib\* Detect
 */
public class Detect {
    private static final String subscriptionKey = System.getenv("FACE_SUBSCRIPTION_KEY");
    private static final String endpoint = System.getenv("FACE_ENDPOINT");

    private static final String imageWithFaces =
        "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/c/c3/RH_Louise_Lillian_Gish.jpg\"}";
// </environment>

// <main>
    public static void main(String[] args) {
        HttpClient httpclient = HttpClientBuilder.create().build();

        try
        {
            URIBuilder builder = new URIBuilder(endpoint + "/face/v1.0/detect");

            // Request parameters. All of them are optional.
            builder.setParameter("detectionModel", "detection_02");
            builder.setParameter("returnFaceId", "true");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

            // Request body.
            StringEntity reqEntity = new StringEntity(imageWithFaces);
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();
// </main>

// <print>
            if (entity != null)
            {
                // Format and display the JSON response.
                System.out.println("REST Response:\n");

                String jsonString = EntityUtils.toString(entity).trim();
                if (jsonString.charAt(0) == '[') {
                    JSONArray jsonArray = new JSONArray(jsonString);
                    System.out.println(jsonArray.toString(2));
                }
                else if (jsonString.charAt(0) == '{') {
                    JSONObject jsonObject = new JSONObject(jsonString);
                    System.out.println(jsonObject.toString(2));
                } else {
                    System.out.println(jsonString);
                }
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
        }
    }
}
// </print>
```

## <a name="run-the-app"></a>运行应用

编译并运行代码。 成功的响应会以易于读取的 JSON 格式在控制台窗口中显示人脸数据。 例如：

```json
[{
  "faceRectangle": {
    "top": 131,
    "left": 177,
    "width": 162,
    "height": 162
  }
}]
```

## <a name="extract-face-attributes"></a>提取人脸属性
 
若要提取人脸属性，请使用检测模型 1 并添加 `returnFaceAttributes` 查询参数。

```java
builder.setParameter("detectionModel", "detection_01");
builder.setParameter("returnFaceAttributes", "age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise");
```

响应现在包含人脸属性。 例如：

```json
[{
  "faceRectangle": {
    "top": 131,
    "left": 177,
    "width": 162,
    "height": 162
  },
  "faceAttributes": {
    "makeup": {
      "eyeMakeup": true,
      "lipMakeup": true
    },
    "facialHair": {
      "sideburns": 0,
      "beard": 0,
      "moustache": 0
    },
    "gender": "female",
    "accessories": [],
    "blur": {
      "blurLevel": "low",
      "value": 0.06
    },
    "headPose": {
      "roll": 0.1,
      "pitch": 0,
      "yaw": -32.9
    },
    "smile": 0,
    "glasses": "NoGlasses",
    "hair": {
      "bald": 0,
      "invisible": false,
      "hairColor": [
        {
          "color": "brown",
          "confidence": 1
        },
        {
          "color": "black",
          "confidence": 0.87
        },
        {
          "color": "other",
          "confidence": 0.51
        },
        {
          "color": "blond",
          "confidence": 0.08
        },
        {
          "color": "red",
          "confidence": 0.08
        },
        {
          "color": "gray",
          "confidence": 0.02
        }
      ]
    },
    "emotion": {
      "contempt": 0,
      "surprise": 0.005,
      "happiness": 0,
      "neutral": 0.986,
      "sadness": 0.009,
      "disgust": 0,
      "anger": 0,
      "fear": 0
    },
    "exposure": {
      "value": 0.67,
      "exposureLevel": "goodExposure"
    },
    "occlusion": {
      "eyeOccluded": false,
      "mouthOccluded": false,
      "foreheadOccluded": false
    },
    "noise": {
      "noiseLevel": "low",
      "value": 0
    },
    "age": 22.9
  },
  "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f"
}]
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你创建了一个简单的 Java 控制台应用程序，该应用程序在 Azure 人脸 API 中使用 REST 调用检测图像中的人脸并返回其属性。 接下来，请浏览人脸 API 参考文档，详细了解支持的方案。

> [!div class="nextstepaction"]
> [人脸 API](https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)

