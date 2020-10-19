---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
origin.date: 01/13/2020
ms.date: 10/16/2020
ms.author: v-tawe
ms.openlocfilehash: 84c84bf7b00a1177e8b3df4f5a615fdc50a1be4b
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127603"
---
## <a name="prerequisites"></a>先决条件

在开始之前，请务必：

> [!div class="checklist"]
> * [设置开发环境并创建空项目](../../../../quickstarts/setup-platform.md?tabs=linux&pivots=programmming-language-cpp)
> * [创建 Azure 语音资源](../../../../get-started.md)
> * [将源文件上传到 Azure blob](https://docs.azure.cn/storage/blobs/storage-quickstart-blobs-portal)
> * 需要语音 SDK 1.11.0 或更高版本。

## <a name="open-your-project-in-visual-studio"></a>在 Visual Studio 中打开项目

第一步是确保在 Visual Studio 中打开项目。

1. 启动 Visual Studio 2019。
2. 加载项目并打开 `helloworld.cpp`。

## <a name="add-a-references"></a>添加参考

为了加速代码开发，我们将使用几个外部组件：
* [CPP Rest SDK](https://github.com/microsoft/cpprestsdk)，用于对 REST 服务进行 REST 调用的客户端库。
* [nlohmann/json](https://github.com/nlohmann/json)，便利的 JSON 分析/序列化/反序列化库。

两者均可使用 [vcpkg](https://github.com/Microsoft/vcpkg/) 进行安装。

```
vcpkg install cpprestsdk cpprestsdk:x64-windows
vcpkg install nlohmann-json
```

## <a name="start-with-some-boilerplate-code"></a>从一些样本代码入手

添加一些代码作为项目的框架。

```cpp
#include <iostream>
#include <strstream>
#include <Windows.h>
#include <locale>
#include <codecvt>
#include <string>

#include <cpprest/http_client.h>
#include <cpprest/filestream.h>
#include <nlohmann/json.hpp>

using namespace std;
using namespace utility;                    // Common utilities like string conversions
using namespace web;                        // Common features like URIs.
using namespace web::http;                  // Common HTTP functionality
using namespace web::http::client;          // HTTP client features
using namespace concurrency::streams;       // Asynchronous streams
using json = nlohmann::json;

const string_t region = U("YourServiceRegion");
const string_t subscriptionKey = U("YourSubscriptionKey");
const string name = "Simple transcription";
const string description = "Simple transcription description";
const string myLocale = "en-US";
const string recordingsBlobUri = "YourFileUrl";

void recognizeSpeech()
{
    std::wstring_convert<std::codecvt_utf8_utf16<wchar_t >> converter;

}

int wmain()
{
    recognizeSpeech();
    cout << "Please press a key to continue.\n";
    cin.get();
    return 0;
}
```

[!INCLUDE [placeholder-replacements](../placeholder-replacement.md)]

## <a name="json-wrappers"></a>JSON 包装器

由于 REST API 采用 JSON 格式的请求并且也返回 JSON 结果，因此我们可以仅使用字符串与之进行交互，但不建议这样做。
为了使请求和响应更易于管理，我们将声明一些用于对 JSON 进行序列化/反序列化处理的类，以及一些可帮助 nlohmann/json 的方法。

继续操作并将其声明放在 `recognizeSpeech` 之前

```cpp
class TranscriptionDefinition {
private:
    TranscriptionDefinition(string name,
        string description,
        string locale,
        string recordingsUrl,
        std::list<string> models) {

        Name = name;
        Description = description;
        RecordingsUrl = recordingsUrl;
        Locale = locale;
        Models = models;
    }

public:
    string Name;
    string Description;
    string RecordingsUrl;
    string Locale;
    std::list<string> Models;
    std::map<string, string> properties;

    static TranscriptionDefinition Create(string name, string description, string locale, string recordingsUrl) {
        return TranscriptionDefinition(name, description, locale, recordingsUrl, std::list<string>());
    }
    static TranscriptionDefinition Create(string name, string description, string locale, string recordingsUrl,
        std::list<string> models) {
        return TranscriptionDefinition(name, description, locale, recordingsUrl, models);
    }
};

void to_json(nlohmann::json& j, const TranscriptionDefinition& t) {
    j = nlohmann::json{
            { "description", t.Description },
            { "locale", t.Locale },
            { "models", t.Models },
            { "name", t.Name },
            { "properties", t.properties },
            { "recordingsurl",t.RecordingsUrl }
    };
};

void from_json(const nlohmann::json& j, TranscriptionDefinition& t) {
    j.at("locale").get_to(t.Locale);
    j.at("models").get_to(t.Models);
    j.at("name").get_to(t.Name);
    j.at("properties").get_to(t.properties);
    j.at("recordingsurl").get_to(t.RecordingsUrl);
}

class Transcription {
public:
    string name;
    string description;
    string locale;
    string recordingsUrl;
    map<string, string> resultsUrls;
    string id;
    string createdDateTime;
    string lastActionDateTime;
    string status;
    string statusMessage;
};

void to_json(nlohmann::json& j, const Transcription& t) {
    j = nlohmann::json{
            { "description", t.description },
            { "locale", t.locale },
            { "createddatetime", t.createdDateTime },
            { "name", t.name },
            { "id", t.id },
            { "recordingsurl",t.recordingsUrl },
            { "resultUrls", t.resultsUrls},
            { "status", t.status},
            { "statusmessage", t.statusMessage}
    };
};

void from_json(const nlohmann::json& j, Transcription& t) {
    j.at("description").get_to(t.description);
    j.at("locale").get_to(t.locale);
    j.at("createdDateTime").get_to(t.createdDateTime);
    j.at("name").get_to(t.name);
    j.at("recordingsUrl").get_to(t.recordingsUrl);
    t.resultsUrls = j.at("resultsUrls").get<map<string, string>>();
    j.at("status").get_to(t.status);
    j.at("statusMessage").get_to(t.statusMessage);
}
class Result
{
public:
    string Lexical;
    string ITN;
    string MaskedITN;
    string Display;
};
void from_json(const nlohmann::json& j, Result& r) {
    j.at("Lexical").get_to(r.Lexical);
    j.at("ITN").get_to(r.ITN);
    j.at("MaskedITN").get_to(r.MaskedITN);
    j.at("Display").get_to(r.Display);
}

class NBest : public Result
{
public:
    double Confidence;  
};
void from_json(const nlohmann::json& j, NBest& nb) {
    j.at("Confidence").get_to(nb.Confidence);
    j.at("Lexical").get_to(nb.Lexical);
    j.at("ITN").get_to(nb.ITN);
    j.at("MaskedITN").get_to(nb.MaskedITN);
    j.at("Display").get_to(nb.Display);
}

class SegmentResult
{
public:
    string RecognitionStatus;
    ULONG Offset;
    ULONG Duration;
    std::list<NBest> NBest;
};
void from_json(const nlohmann::json& j, SegmentResult& sr) {
    j.at("RecognitionStatus").get_to(sr.RecognitionStatus);
    j.at("Offset").get_to(sr.Offset);
    j.at("Duration").get_to(sr.Duration);
    sr.NBest = j.at("NBest").get<list<NBest>>();
}

class AudioFileResult
{
public:
    string AudioFileName;
    std::list<SegmentResult> SegmentResults;
    std::list<Result> CombinedResults;
};
void from_json(const nlohmann::json& j, AudioFileResult& arf) {
    j.at("AudioFileName").get_to(arf.AudioFileName);
    arf.SegmentResults = j.at("SegmentResults").get<list<SegmentResult>>();
    arf.CombinedResults = j.at("CombinedResults").get<list<Result>>();
}

class RootObject {
public:
    std::list<AudioFileResult> AudioFileResults;
};
void from_json(const nlohmann::json& j, RootObject& r) {
    r.AudioFileResults = j.at("AudioFileResults").get<list<AudioFileResult>>();
}
```

## <a name="create-and-configure-an-http-client"></a>创建和配置 Http 客户端
首先，我们需要具有正确的基本 URL 和身份验证集的 Http 客户端。
将此代码插入 `recognizeSpeech`

```cpp
utility::string_t service_url = U("https://") + region + U(".cris.azure.cn/api/speechtotext/v2.0/Transcriptions/");
uri u(service_url);

http_client c(u);
http_request msg(methods::POST);
msg.headers().add(U("Content-Type"), U("application/json"));
msg.headers().add(U("Ocp-Apim-Subscription-Key"), subscriptionKey);
```

## <a name="generate-a-transcription-request"></a>生成听录请求
接下来，我们将生成听录请求。 将此代码添加到 `recognizeSpeech`

```cpp
auto transportdef = TranscriptionDefinition::Create(name, description, myLocale, recordingsBlobUri);

nlohmann::json transportdefJSON = transportdef;

msg.set_body(transportdefJSON.dump());
```

## <a name="send-the-request-and-check-its-status"></a>发送请求并查看其状态
现在我们将请求发布到语音服务并检查初始响应代码。 此响应代码将仅指示服务是否已收到请求。 该服务将在响应标头中返回一个 URL，这是它将存储听录状态的位置。

```cpp
auto response = c.request(msg).get();
auto statusCode = response.status_code();

if (statusCode != status_codes::Accepted)
{
    cout << "Unexpected status code " << statusCode << endl;
    return;
}

string_t transcriptionLocation = response.headers()[U("location")];

cout << "Transcription status is located at " << converter.to_bytes(transcriptionLocation) << endl;
```

## <a name="wait-for-the-transcription-to-complete"></a>等待听录完成
由于服务以异步方式处理听录，因此需要时常轮询其状态。 我们每 5 秒查看一次。

通过检索在发布请求时收到的 URL 中的内容，可以查看状态。 内容返回后，我们将其反序列化为一个帮助程序类，使其便于交互。

下面是一个轮询代码，其中显示了除成功完成之外的所有状态，我们会在下一步完成该操作。

```cpp
bool completed = false;

while (!completed)
{
    auto statusResponse = statusCheckClient.request(statusCheckMessage).get();
    auto statusResponseCode = statusResponse.status_code();

    if (statusResponseCode != status_codes::OK)
    {
        cout << "Fetching the transcription returned unexpected http code " << statusResponseCode << endl;
        return;
    }

    auto body = statusResponse.extract_string().get();
    nlohmann::json statusJSON = nlohmann::json::parse(body);
    Transcription transcriptonStatus = statusJSON;

    if (!_stricmp(transcriptonStatus.status.c_str(), "Failed"))
    {
        completed = true;
        cout << "Transcription has failed " << transcriptonStatus.statusMessage << endl;
    }
    else if (!_stricmp(transcriptonStatus.status.c_str(), "Succeeded"))
    {
    }
    else if (!_stricmp(transcriptonStatus.status.c_str(), "Running"))
    {
        cout << "Transcription is running." << endl;
    }
    else if (!_stricmp(transcriptonStatus.status.c_str(), "NotStarted"))
    {
        cout << "Transcription has not started." << endl;
    }

    if (!completed) {
        Sleep(5000);
    }

}
```

## <a name="display-the-transcription-results"></a>显示听录结果
服务成功完成听录后，结果将存储在可从状态响应中获取的其他 URL 中。

我们将下载该 URL 的所有内容，对 JSON 进行反序列化，并循环遍历不断输出显示文本的结果。

```cpp
completed = true;
cout << "Success!" << endl;
string result = transcriptonStatus.resultsUrls["channel_0"];
cout << "Transcription has completed. Results are at " << result << endl;
cout << "Fetching results" << endl;

http_client result_client(converter.from_bytes(result));
http_request resultMessage(methods::GET);
resultMessage.headers().add(U("Ocp-Apim-Subscription-Key"), subscriptionKey);

auto resultResponse = result_client.request(resultMessage).get();

auto responseCode = resultResponse.status_code();

if (responseCode != status_codes::OK)
{
    cout << "Fetching the transcription returned unexpected http code " << responseCode << endl;
    return;
}

auto resultBody = resultResponse.extract_string().get();

nlohmann::json resultJSON = nlohmann::json::parse(resultBody);
RootObject root = resultJSON;

for (AudioFileResult af : root.AudioFileResults)
{
    cout << "There were " << af.SegmentResults.size() << " results in " << af.AudioFileName << endl;
    
    for (SegmentResult segResult : af.SegmentResults)
    {
        cout << "Status: " << segResult.RecognitionStatus << endl;

        if (!_stricmp(segResult.RecognitionStatus.c_str(), "success") && segResult.NBest.size() > 0)
        {
            cout << "Best text result was: '" << segResult.NBest.front().Display << "'" << endl;
        }
    }
}
```

## <a name="check-your-code"></a>查看代码
此时，代码应如下所示：（我们已向此版本添加了一些注释）

```cpp
#include <iostream>
#include <strstream>
#include <Windows.h>
#include <locale>
#include <codecvt>
#include <string>

#include <cpprest/http_client.h>
#include <cpprest/filestream.h>
#include <nlohmann/json.hpp>

using namespace std;
using namespace utility;                    // Common utilities like string conversions
using namespace web;                        // Common features like URIs.
using namespace web::http;                  // Common HTTP functionality
using namespace web::http::client;          // HTTP client features
using namespace concurrency::streams;       // Asynchronous streams
using json = nlohmann::json;

const string_t region = U("YourServiceRegion");
const string_t subscriptionKey = U("YourSubscriptionKey");
const string name = "Simple transcription";
const string description = "Simple transcription description";
const string myLocale = "en-US";
const string recordingsBlobUri = "YourFileUrl";

class TranscriptionDefinition {
private:
    TranscriptionDefinition(string name,
        string description,
        string locale,
        string recordingsUrl,
        std::list<string> models) {

        Name = name;
        Description = description;
        RecordingsUrl = recordingsUrl;
        Locale = locale;
        Models = models;
    }

public:
    string Name;
    string Description;
    string RecordingsUrl;
    string Locale;
    std::list<string> Models;
    std::map<string, string> properties;

    static TranscriptionDefinition Create(string name, string description, string locale, string recordingsUrl) {
        return TranscriptionDefinition(name, description, locale, recordingsUrl, std::list<string>());
    }
    static TranscriptionDefinition Create(string name, string description, string locale, string recordingsUrl,
        std::list<string> models) {
        return TranscriptionDefinition(name, description, locale, recordingsUrl, models);
    }
};

void to_json(nlohmann::json& j, const TranscriptionDefinition& t) {
    j = nlohmann::json{
            { "description", t.Description },
            { "locale", t.Locale },
            { "models", t.Models },
            { "name", t.Name },
            { "properties", t.properties },
            { "recordingsurl",t.RecordingsUrl }
    };
};

void from_json(const nlohmann::json& j, TranscriptionDefinition& t) {
    j.at("locale").get_to(t.Locale);
    j.at("models").get_to(t.Models);
    j.at("name").get_to(t.Name);
    j.at("properties").get_to(t.properties);
    j.at("recordingsurl").get_to(t.RecordingsUrl);
}

class Transcription {
public:
    string name;
    string description;
    string locale;
    string recordingsUrl;
    map<string, string> resultsUrls;
    string id;
    string createdDateTime;
    string lastActionDateTime;
    string status;
    string statusMessage;
};

void to_json(nlohmann::json& j, const Transcription& t) {
    j = nlohmann::json{
            { "description", t.description },
            { "locale", t.locale },
            { "createddatetime", t.createdDateTime },
            { "name", t.name },
            { "id", t.id },
            { "recordingsurl",t.recordingsUrl },
            { "resultUrls", t.resultsUrls},
            { "status", t.status},
            { "statusmessage", t.statusMessage}
    };
};

void from_json(const nlohmann::json& j, Transcription& t) {
    j.at("description").get_to(t.description);
    j.at("locale").get_to(t.locale);
    j.at("createdDateTime").get_to(t.createdDateTime);
    j.at("name").get_to(t.name);
    j.at("recordingsUrl").get_to(t.recordingsUrl);
    t.resultsUrls = j.at("resultsUrls").get<map<string, string>>();
    j.at("status").get_to(t.status);
    j.at("statusMessage").get_to(t.statusMessage);
}
class Result
{
public:
    string Lexical;
    string ITN;
    string MaskedITN;
    string Display;
};
void from_json(const nlohmann::json& j, Result& r) {
    j.at("Lexical").get_to(r.Lexical);
    j.at("ITN").get_to(r.ITN);
    j.at("MaskedITN").get_to(r.MaskedITN);
    j.at("Display").get_to(r.Display);
}

class NBest : public Result
{
public:
    double Confidence;  
};
void from_json(const nlohmann::json& j, NBest& nb) {
    j.at("Confidence").get_to(nb.Confidence);
    j.at("Lexical").get_to(nb.Lexical);
    j.at("ITN").get_to(nb.ITN);
    j.at("MaskedITN").get_to(nb.MaskedITN);
    j.at("Display").get_to(nb.Display);
}

class SegmentResult
{
public:
    string RecognitionStatus;
    ULONG Offset;
    ULONG Duration;
    std::list<NBest> NBest;
};
void from_json(const nlohmann::json& j, SegmentResult& sr) {
    j.at("RecognitionStatus").get_to(sr.RecognitionStatus);
    j.at("Offset").get_to(sr.Offset);
    j.at("Duration").get_to(sr.Duration);
    sr.NBest = j.at("NBest").get<list<NBest>>();
}

class AudioFileResult
{
public:
    string AudioFileName;
    std::list<SegmentResult> SegmentResults;
    std::list<Result> CombinedResults;
};
void from_json(const nlohmann::json& j, AudioFileResult& arf) {
    j.at("AudioFileName").get_to(arf.AudioFileName);
    arf.SegmentResults = j.at("SegmentResults").get<list<SegmentResult>>();
    arf.CombinedResults = j.at("CombinedResults").get<list<Result>>();
}

class RootObject {
public:
    std::list<AudioFileResult> AudioFileResults;
};
void from_json(const nlohmann::json& j, RootObject& r) {
    r.AudioFileResults = j.at("AudioFileResults").get<list<AudioFileResult>>();
}


void recognizeSpeech()
{
    std::wstring_convert<std::codecvt_utf8_utf16<wchar_t >> converter;

    utility::string_t service_url = U("https://") + region + U(".cris.azure.cn/api/speechtotext/v2.0/Transcriptions/");
    uri u(service_url);

    http_client c(u);
    http_request msg(methods::POST);
    msg.headers().add(U("Content-Type"), U("application/json"));
    msg.headers().add(U("Ocp-Apim-Subscription-Key"), subscriptionKey);

    auto transportdef = TranscriptionDefinition::Create(name, description, myLocale, recordingsBlobUri);

    nlohmann::json transportdefJSON = transportdef;

    msg.set_body(transportdefJSON.dump());

    auto response = c.request(msg).get();
    auto statusCode = response.status_code();

    if (statusCode != status_codes::Accepted)
    {
        cout << "Unexpected status code " << statusCode << endl;
        return;
    }

    string_t transcriptionLocation = response.headers()[U("location")];

    cout << "Transcription status is located at " << converter.to_bytes(transcriptionLocation) << endl;

    http_client statusCheckClient(transcriptionLocation);
    http_request statusCheckMessage(methods::GET);
    statusCheckMessage.headers().add(U("Ocp-Apim-Subscription-Key"), subscriptionKey);

    bool completed = false;

    while (!completed)
    {
        auto statusResponse = statusCheckClient.request(statusCheckMessage).get();
        auto statusResponseCode = statusResponse.status_code();

        if (statusResponseCode != status_codes::OK)
        {
            cout << "Fetching the transcription returned unexpected http code " << statusResponseCode << endl;
            return;
        }

        auto body = statusResponse.extract_string().get();
        nlohmann::json statusJSON = nlohmann::json::parse(body);
        Transcription transcriptonStatus = statusJSON;

        if (!_stricmp(transcriptonStatus.status.c_str(), "Failed"))
        {
            completed = true;
            cout << "Transcription has failed " << transcriptonStatus.statusMessage << endl;
        }
        else if (!_stricmp(transcriptonStatus.status.c_str(), "Succeeded"))
        {
            completed = true;
            cout << "Success!" << endl;
            string result = transcriptonStatus.resultsUrls["channel_0"];
            cout << "Transcription has completed. Results are at " << result << endl;
            cout << "Fetching results" << endl;

            http_client result_client(converter.from_bytes(result));
            http_request resultMessage(methods::GET);
            resultMessage.headers().add(U("Ocp-Apim-Subscription-Key"), subscriptionKey);

            auto resultResponse = result_client.request(resultMessage).get();

            auto responseCode = resultResponse.status_code();

            if (responseCode != status_codes::OK)
            {
                cout << "Fetching the transcription returned unexpected http code " << responseCode << endl;
                return;
            }

            auto resultBody = resultResponse.extract_string().get();
            
            nlohmann::json resultJSON = nlohmann::json::parse(resultBody);
            RootObject root = resultJSON;

            for (AudioFileResult af : root.AudioFileResults)
            {
                cout << "There were " << af.SegmentResults.size() << " results in " << af.AudioFileName << endl;
                
                for (SegmentResult segResult : af.SegmentResults)
                {
                    cout << "Status: " << segResult.RecognitionStatus << endl;

                    if (!_stricmp(segResult.RecognitionStatus.c_str(), "success") && segResult.NBest.size() > 0)
                    {
                        cout << "Best text result was: '" << segResult.NBest.front().Display << "'" << endl;
                    }
                }
            }
        }
        else if (!_stricmp(transcriptonStatus.status.c_str(), "Running"))
        {
            cout << "Transcription is running." << endl;
        }
        else if (!_stricmp(transcriptonStatus.status.c_str(), "NotStarted"))
        {
            cout << "Transcription has not started." << endl;
        }

        if (!completed) {
            Sleep(5000);
        }

    }
}

int wmain()
{
    recognizeSpeech();
    cout << "Please press a key to continue.\n";
    cin.get();
    return 0;
}
```

## <a name="build-and-run-your-app"></a>生成并运行应用

现在，可以使用语音服务构建应用并测试语音识别。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [footer](./footer.md)]