---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
origin.date: 01/13/2020
ms.date: 10/16/2020
ms.author: v-tawe
ms.openlocfilehash: 8fbcd365a069478dda5b05f37d363518244a9474
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127606"
---
## <a name="prerequisites"></a>先决条件

在开始之前，请务必：

> [!div class="checklist"]
> * [设置开发环境并创建空项目](../../../../quickstarts/setup-platform.md?tabs=jre&pivots=programmming-language-java)
> * [创建 Azure 语音资源](../../../../overview.md#try-the-speech-service-for-free)
> * [将源文件上传到 Azure blob](https://docs.azure.cn/storage/blobs/storage-quickstart-blobs-portal)
> * 需要语音 SDK 版本 1.11.0 或更高版本。

## <a name="open-your-project-in-eclipse"></a>在 Eclipse 中打开项目

第一步是确保项目在 Eclipse 中打开。

1. 启动 Eclipse
2. 加载项目并打开 `Main.java`。

## <a name="add-a-reference-to-gson"></a>添加对 Gson 的引用
本快速入门中将使用外部 JSON 序列化程序/反序列化程序。 对于 Java，我们选择了 [Gson](https://github.com/google/gson)。

打开 pom.xml 并添加以下引用。

```xml
<dependencies>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>
</dependencies>
```

## <a name="start-with-some-boilerplate-code"></a>从一些样本代码入手

添加一些代码作为项目的框架。

```java
package quickstart;

import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Date;
import java.util.Dictionary;
import java.util.Hashtable;
import java.util.UUID;

import com.google.gson.Gson;
public class Main {

    private static String region = "YourServiceRegion";
    private static String subscriptionKey = "YourSubscriptionKey";
    private static String Locale = "en-US";
    private static String RecordingsBlobUri = "YourFileUrl";
    private static String Name = "Simple transcription";
    private static String Description = "Simple transcription description";

    public static void main(String[] args) throws IOException, InterruptedException {
        System.out.println("Starting transcriptions client...");
    }
}
```

[!INCLUDE [placeholder-replacements](../placeholder-replacement.md)]

## <a name="json-wrappers"></a>JSON 包装器

由于 REST API 采用 JSON 格式的请求并且也返回 JSON 结果，因此我们可以仅使用字符串与之进行交互，但不建议这样做。
为了使请求和响应更易于管理，我们将声明一些用于对 JSON 进行序列化/反序列化处理的类。

开始操作并在 `Main` 之前放置声明。
```java
final class Transcription {
    public String name;
    public String description;
    public String locale;
    public URL recordingsUrl;
    public Hashtable<String, String> resultsUrls;
    public UUID id;
    public Date createdDateTime;
    public Date lastActionDateTime;
    public String status;
    public String statusMessage;
}

final class TranscriptionDefinition {
    private TranscriptionDefinition(String name, String description, String locale, URL recordingsUrl,
            ModelIdentity[] models) {
        this.Name = name;
        this.Description = description;
        this.RecordingsUrl = recordingsUrl;
        this.Locale = locale;
        this.Models = models;
        this.properties = new Hashtable<String, String>();
        this.properties.put("PunctuationMode", "DictatedAndAutomatic");
        this.properties.put("ProfanityFilterMode", "Masked");
        this.properties.put("AddWordLevelTimestamps", "True");
    }

    public String Name;
    public String Description;
    public URL RecordingsUrl;
    public String Locale;
    public ModelIdentity[] Models;
    public Dictionary<String, String> properties;

    public static TranscriptionDefinition Create(String name, String description, String locale, URL recordingsUrl) {
        return TranscriptionDefinition.Create(name, description, locale, recordingsUrl, new ModelIdentity[0]);
    }

    public static TranscriptionDefinition Create(String name, String description, String locale, URL recordingsUrl,
            ModelIdentity[] models) {
        return new TranscriptionDefinition(name, description, locale, recordingsUrl, models);
    }
}

final class ModelIdentity {
    private ModelIdentity(UUID id) {
        this.Id = id;
    }

    public UUID Id;

    public static ModelIdentity Create(UUID Id) {
        return new ModelIdentity(Id);
    }
}

class AudioFileResult {
    public String AudioFileName;
    public SegmentResult[] SegmentResults;
}

class RootObject {
    public AudioFileResult[] AudioFileResults;
}

class NBest {
    public double Confidence;
    public String Lexical;
    public String ITN;
    public String MaskedITN;
    public String Display;
}

class SegmentResult {
    public String RecognitionStatus;
    public String Offset;
    public String Duration;
    public NBest[] NBest;
}
```

## <a name="create-and-configure-an-http-client"></a>创建和配置 Http 客户端
首先，我们需要具有正确的基本 URL 和身份验证集的 Http 客户端。
将此代码插入 `Main`
```java
String url = "https://" + region + ".cris.azure.cn/api/speechtotext/v2.0/Transcriptions/";
URL serviceUrl = new URL(url);

HttpURLConnection postConnection = (HttpURLConnection) serviceUrl.openConnection();
postConnection.setDoOutput(true);
postConnection.setRequestMethod("POST");
postConnection.setRequestProperty("Content-Type", "application/json");
postConnection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);

```

## <a name="generate-a-transcription-request"></a>生成听录请求
接下来，我们将生成听录请求。 将此代码添加到 `Main`

```java
TranscriptionDefinition definition = TranscriptionDefinition.Create(Name, Description, Locale,
        new URL(RecordingsBlobUri));
```

## <a name="send-the-request-and-check-its-status"></a>发送请求并查看其状态
现在我们将请求发布到语音服务并检查初始响应代码。 此响应代码将仅指示服务是否已收到请求。 该服务将在响应标头中返回一个 URL，这是它将存储听录状态的位置。

```java
Gson gson = new Gson();

OutputStream stream = postConnection.getOutputStream();
stream.write(gson.toJson(definition).getBytes());
stream.flush();

int statusCode = postConnection.getResponseCode();

if (statusCode != HttpURLConnection.HTTP_ACCEPTED) {
    System.out.println("Unexpected status code " + statusCode);
    return;
}
```

## <a name="wait-for-the-transcription-to-complete"></a>等待听录完成
由于服务以异步方式处理听录，因此需要时常轮询其状态。 我们每 5 秒查看一次。

通过检索在发布请求时收到的 URL 中的内容，可以查看状态。 内容返回后，我们将其反序列化为一个帮助程序类，使其便于交互。

下面是一个轮询代码，其中显示了除成功完成之外的所有状态，我们会在下一步完成该操作。

```java
String transcriptionLocation = postConnection.getHeaderField("location");

System.out.println("Transcription is located at " + transcriptionLocation);

URL transcriptionUrl = new URL(transcriptionLocation);

boolean completed = false;
while (!completed) {
    HttpURLConnection getConnection = (HttpURLConnection) transcriptionUrl.openConnection();
    getConnection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
    getConnection.setRequestMethod("GET");

    int responseCode = getConnection.getResponseCode();

    if (responseCode != HttpURLConnection.HTTP_OK) {
        System.out.println("Fetching the transcription returned unexpected http code " + responseCode);
        return;
    }

    Transcription t = gson.fromJson(new InputStreamReader(getConnection.getInputStream()),
            Transcription.class);

    switch (t.status) {
    case "Failed":
        completed = true;
        System.out.println("Transcription has failed " + t.statusMessage);
        break;
    case "Succeeded":
        break;
    case "Running":
        System.out.println("Transcription is running.");
        break;
    case "NotStarted":
        System.out.println("Transcription has not started.");
        break;
    }

    if (!completed) {
        Thread.sleep(5000);
    }
}
```

## <a name="display-the-transcription-results"></a>显示听录结果
服务成功完成听录后，结果将存储在可从状态响应中获取的其他 URL 中。

我们将下载该 URL 的所有内容，对 JSON 进行反序列化，并循环遍历不断输出显示文本的结果。

```java
package quickstart;

import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Date;
import java.util.Dictionary;
import java.util.Hashtable;
import java.util.UUID;

import com.google.gson.Gson;

final class Transcription {
    public String name;
    public String description;
    public String locale;
    public URL recordingsUrl;
    public Hashtable<String, String> resultsUrls;
    public UUID id;
    public Date createdDateTime;
    public Date lastActionDateTime;
    public String status;
    public String statusMessage;
}

final class TranscriptionDefinition {
    private TranscriptionDefinition(String name, String description, String locale, URL recordingsUrl,
            ModelIdentity[] models) {
        this.Name = name;
        this.Description = description;
        this.RecordingsUrl = recordingsUrl;
        this.Locale = locale;
        this.Models = models;
        this.properties = new Hashtable<String, String>();
        this.properties.put("PunctuationMode", "DictatedAndAutomatic");
        this.properties.put("ProfanityFilterMode", "Masked");
        this.properties.put("AddWordLevelTimestamps", "True");
    }

    public String Name;
    public String Description;
    public URL RecordingsUrl;
    public String Locale;
    public ModelIdentity[] Models;
    public Dictionary<String, String> properties;

    public static TranscriptionDefinition Create(String name, String description, String locale, URL recordingsUrl) {
        return TranscriptionDefinition.Create(name, description, locale, recordingsUrl, new ModelIdentity[0]);
    }

    public static TranscriptionDefinition Create(String name, String description, String locale, URL recordingsUrl,
            ModelIdentity[] models) {
        return new TranscriptionDefinition(name, description, locale, recordingsUrl, models);
    }
}

final class ModelIdentity {
    private ModelIdentity(UUID id) {
        this.Id = id;
    }

    public UUID Id;

    public static ModelIdentity Create(UUID Id) {
        return new ModelIdentity(Id);
    }
}

class AudioFileResult {
    public String AudioFileName;
    public SegmentResult[] SegmentResults;
}

class RootObject {
    public AudioFileResult[] AudioFileResults;
}

class NBest {
    public double Confidence;
    public String Lexical;
    public String ITN;
    public String MaskedITN;
    public String Display;
}

class SegmentResult {
    public String RecognitionStatus;
    public String Offset;
    public String Duration;
    public NBest[] NBest;
}

public class Main {

    private static String region = "YourServiceRegion";
    private static String subscriptionKey = "YourSubscriptionKey";
    private static String Locale = "en-US";
    private static String RecordingsBlobUri = "YourFileUrl";
    private static String Name = "Simple transcription";
    private static String Description = "Simple transcription description";

    public static void main(String[] args) throws IOException, InterruptedException {
        System.out.println("Starting transcriptions client...");
        String url = "https://" + region + ".cris.azure.cn/api/speechtotext/v2.0/Transcriptions/";
        URL serviceUrl = new URL(url);

        HttpURLConnection postConnection = (HttpURLConnection) serviceUrl.openConnection();
        postConnection.setDoOutput(true);
        postConnection.setRequestMethod("POST");
        postConnection.setRequestProperty("Content-Type", "application/json");
        postConnection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);

        TranscriptionDefinition definition = TranscriptionDefinition.Create(Name, Description, Locale,
                new URL(RecordingsBlobUri));

        Gson gson = new Gson();

        OutputStream stream = postConnection.getOutputStream();
        stream.write(gson.toJson(definition).getBytes());
        stream.flush();

        int statusCode = postConnection.getResponseCode();

        if (statusCode != HttpURLConnection.HTTP_ACCEPTED) {
            System.out.println("Unexpected status code " + statusCode);
            return;
        }

        String transcriptionLocation = postConnection.getHeaderField("location");

        System.out.println("Transcription is located at " + transcriptionLocation);

        URL transcriptionUrl = new URL(transcriptionLocation);

        boolean completed = false;
        while (!completed) {
            {
                HttpURLConnection getConnection = (HttpURLConnection) transcriptionUrl.openConnection();
                getConnection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
                getConnection.setRequestMethod("GET");

                int responseCode = getConnection.getResponseCode();

                if (responseCode != HttpURLConnection.HTTP_OK) {
                    System.out.println("Fetching the transcription returned unexpected http code " + responseCode);
                    return;
                }

                Transcription t = gson.fromJson(new InputStreamReader(getConnection.getInputStream()),
                        Transcription.class);

                switch (t.status) {
                case "Failed":
                    completed = true;
                    System.out.println("Transcription has failed " + t.statusMessage);
                    break;
                case "Succeeded":
                    completed = true;
                    String result = t.resultsUrls.get("channel_0");
                    System.out.println("Transcription has completed. Results are at " + result);

                    System.out.println("Fetching results");
                    URL resultUrl = new URL(result);

                    HttpURLConnection resultConnection = (HttpURLConnection) resultUrl.openConnection();
                    resultConnection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
                    resultConnection.setRequestMethod("GET");

                    responseCode = resultConnection.getResponseCode();

                    if (responseCode != HttpURLConnection.HTTP_OK) {
                        System.out.println("Fetching the transcription returned unexpected http code " + responseCode);
                        return;
                    }

                    RootObject root = gson.fromJson(new InputStreamReader(resultConnection.getInputStream()),
                            RootObject.class);

                    for (AudioFileResult af : root.AudioFileResults) {
                        System.out
                                .println("There were " + af.SegmentResults.length + " results in " + af.AudioFileName);
                        for (SegmentResult segResult : af.SegmentResults) {
                            System.out.println("Status: " + segResult.RecognitionStatus);
                            if (segResult.RecognitionStatus.equalsIgnoreCase("success") && segResult.NBest.length > 0) {
                                System.out.println("Best text result was: '" + segResult.NBest[0].Display + "'");
                            }
                        }
                    }

                    break;
                case "Running":
                    System.out.println("Transcription is running.");
                    break;
                case "NotStarted":
                    System.out.println("Transcription has not started.");
                    break;
                }

                if (!completed) {
                    Thread.sleep(5000);
                }
            }
        }
    }
}
```

## <a name="check-your-code"></a>查看代码
此时，代码应如下所示：（我们已向此版本添加了一些注释）

```java
package quickstart;

import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Date;
import java.util.Dictionary;
import java.util.Hashtable;
import java.util.UUID;

import com.google.gson.Gson;

final class Transcription {
    public String name;
    public String description;
    public String locale;
    public URL recordingsUrl;
    public Hashtable<String, String> resultsUrls;
    public UUID id;
    public Date createdDateTime;
    public Date lastActionDateTime;
    public String status;
    public String statusMessage;
}

final class TranscriptionDefinition {
    private TranscriptionDefinition(String name, String description, String locale, URL recordingsUrl,
            ModelIdentity[] models) {
        this.Name = name;
        this.Description = description;
        this.RecordingsUrl = recordingsUrl;
        this.Locale = locale;
        this.Models = models;
        this.properties = new Hashtable<String, String>();
        this.properties.put("PunctuationMode", "DictatedAndAutomatic");
        this.properties.put("ProfanityFilterMode", "Masked");
        this.properties.put("AddWordLevelTimestamps", "True");
    }

    public String Name;
    public String Description;
    public URL RecordingsUrl;
    public String Locale;
    public ModelIdentity[] Models;
    public Dictionary<String, String> properties;

    public static TranscriptionDefinition Create(String name, String description, String locale, URL recordingsUrl) {
        return TranscriptionDefinition.Create(name, description, locale, recordingsUrl, new ModelIdentity[0]);
    }

    public static TranscriptionDefinition Create(String name, String description, String locale, URL recordingsUrl,
            ModelIdentity[] models) {
        return new TranscriptionDefinition(name, description, locale, recordingsUrl, models);
    }
}

final class ModelIdentity {
    private ModelIdentity(UUID id) {
        this.Id = id;
    }

    public UUID Id;

    public static ModelIdentity Create(UUID Id) {
        return new ModelIdentity(Id);
    }
}

class AudioFileResult {
    public String AudioFileName;
    public SegmentResult[] SegmentResults;
}

class RootObject {
    public AudioFileResult[] AudioFileResults;
}

class NBest {
    public double Confidence;
    public String Lexical;
    public String ITN;
    public String MaskedITN;
    public String Display;
}

class SegmentResult {
    public String RecognitionStatus;
    public String Offset;
    public String Duration;
    public NBest[] NBest;
}

public class Main {

    private static String region = "YourServiceRegion";
    private static String subscriptionKey = "YourSubscriptionKey";
    private static String Locale = "en-US";
    private static String RecordingsBlobUri = "YourFileUrl";
    private static String Name = "Simple transcription";
    private static String Description = "Simple transcription description";

    public static void main(String[] args) throws IOException, InterruptedException {
        System.out.println("Starting transcriptions client...");
        String url = "https://" + region + ".cris.azure.cn/api/speechtotext/v2.0/Transcriptions/";
        URL serviceUrl = new URL(url);

        HttpURLConnection postConnection = (HttpURLConnection) serviceUrl.openConnection();
        postConnection.setDoOutput(true);
        postConnection.setRequestMethod("POST");
        postConnection.setRequestProperty("Content-Type", "application/json");
        postConnection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);

        TranscriptionDefinition definition = TranscriptionDefinition.Create(Name, Description, Locale,
                new URL(RecordingsBlobUri));

        Gson gson = new Gson();

        OutputStream stream = postConnection.getOutputStream();
        stream.write(gson.toJson(definition).getBytes());
        stream.flush();

        int statusCode = postConnection.getResponseCode();

        if (statusCode != HttpURLConnection.HTTP_ACCEPTED) {
            System.out.println("Unexpected status code " + statusCode);
            return;
        }

        String transcriptionLocation = postConnection.getHeaderField("location");

        System.out.println("Transcription is located at " + transcriptionLocation);

        URL transcriptionUrl = new URL(transcriptionLocation);

        boolean completed = false;
        while (!completed) {
            {
                HttpURLConnection getConnection = (HttpURLConnection) transcriptionUrl.openConnection();
                getConnection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
                getConnection.setRequestMethod("GET");

                int responseCode = getConnection.getResponseCode();

                if (responseCode != HttpURLConnection.HTTP_OK) {
                    System.out.println("Fetching the transcription returned unexpected http code " + responseCode);
                    return;
                }

                Transcription t = gson.fromJson(new InputStreamReader(getConnection.getInputStream()),
                        Transcription.class);

                switch (t.status) {
                case "Failed":
                    completed = true;
                    System.out.println("Transcription has failed " + t.statusMessage);
                    break;
                case "Succeeded":
                    completed = true;
                    String result = t.resultsUrls.get("channel_0");
                    System.out.println("Transcription has completed. Results are at " + result);

                    System.out.println("Fetching results");
                    URL resultUrl = new URL(result);

                    HttpURLConnection resultConnection = (HttpURLConnection) resultUrl.openConnection();
                    resultConnection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
                    resultConnection.setRequestMethod("GET");

                    responseCode = resultConnection.getResponseCode();

                    if (responseCode != HttpURLConnection.HTTP_OK) {
                        System.out.println("Fetching the transcription returned unexpected http code " + responseCode);
                        return;
                    }

                    RootObject root = gson.fromJson(new InputStreamReader(resultConnection.getInputStream()),
                            RootObject.class);

                    for (AudioFileResult af : root.AudioFileResults) {
                        System.out
                                .println("There were " + af.SegmentResults.length + " results in " + af.AudioFileName);
                        for (SegmentResult segResult : af.SegmentResults) {
                            System.out.println("Status: " + segResult.RecognitionStatus);
                            if (segResult.RecognitionStatus.equalsIgnoreCase("success") && segResult.NBest.length > 0) {
                                System.out.println("Best text result was: '" + segResult.NBest[0].Display + "'");
                            }
                        }
                    }

                    break;
                case "Running":
                    System.out.println("Transcription is running.");
                    break;
                case "NotStarted":
                    System.out.println("Transcription has not started.");
                    break;
                }

                if (!completed) {
                    Thread.sleep(5000);
                }
            }
        }
    }
}
```

## <a name="build-and-run-your-app"></a>生成并运行应用

现在，可以使用语音服务构建应用并测试语音识别。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [footer](./footer.md)]
