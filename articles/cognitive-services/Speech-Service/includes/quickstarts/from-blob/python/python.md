---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
origin.date: 04/04/2020
ms.date: 10/16/2020
ms.author: v-tawe
ms.openlocfilehash: 547d0acb203898999249dbbf866623b09bff8ba5
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127605"
---
## <a name="prerequisites"></a>先决条件

在开始之前，请务必：

> [!div class="checklist"]
> * [设置开发环境并创建空项目](../../../../quickstarts/setup-platform.md?pivots=programming-language-python)
> * [创建 Azure 语音资源](../../../../overview.md#try-the-speech-service-for-free)
> * [将源文件上传到 Azure blob](https://docs.azure.cn/storage/blobs/storage-quickstart-blobs-portal)
> * 需要语音 SDK 版本 1.11.0 或更高版本。

## <a name="download-and-install-the-api-client-library"></a>下载并安装 API 客户端库

若要执行该示例，需要为通过 [Swagger](https://swagger.io) 生成的 REST API 生成 Python 库。

请按照以下步骤进行安装：

1. 转到  https://editor.swagger.io 。
1. 单击“文件”，然后单击“导入 URL” 。
1. 输入 Swagger URL，包括语音服务订阅的区域：`https://<your-region>.cris.azure.cn/docs/v2.0/swagger`。
1. 单击“生成客户端”，然后选择“Python” 。
1. 保存客户端库。
1. 将下载的 python-client-generated.zip 提取到文件系统中的某个位置。
1. 使用 pip：`pip install path/to/package/python-client` 在 Python 环境中安装提取的 python-client 模块。
1. 已安装的名为 `swagger_client` 的包。 可以使用命令 `python -c "import swagger_client"` 来检查安装是否正常工作。

> [!NOTE]
> 由于 [Swagger 自动生成中的已知 bug](https://github.com/swagger-api/swagger-codegen/issues/7541)，导入 `swagger_client` 包时可能会遇到错误。
> 这些错误可以通过从已安装的包中删除
> ```py
> from swagger_client.models.model import Model  # noqa: F401,E501
> ```
> 文件 `swagger_client/models/model.py` 中包含以下内容的行和
> ```py
> from swagger_client.models.inner_error import InnerError  # noqa: F401,E501
> ```
> 文件 `swagger_client/models/inner_error.py` 中包含以下内容的行来进行修复。 错误消息将告知安装这些文件的位置。

## <a name="install-other-dependencies"></a>安装其他依赖项

该示例使用 `requests` 库。 可以通过以下命令进行安装

```bash
pip install requests
```

## <a name="start-with-some-boilerplate-code"></a>从一些样本代码入手

添加一些代码作为项目的框架。

```python
#!/usr/bin/env python
# coding: utf-8
from typing import List

import logging
import sys
import requests
import time
import swagger_client as cris_client


logging.basicConfig(stream=sys.stdout, level=logging.DEBUG, format="%(message)s")

# Your subscription key and region for the Speech service
SUBSCRIPTION_KEY = "YourSubscriptionKey"
SERVICE_REGION = "YourServiceRegion"

NAME = "Simple transcription"
DESCRIPTION = "Simple transcription description"

LOCALE = "en-US"
RECORDINGS_BLOB_URI = "<Your SAS Uri to the recording>"

# Set subscription information when doing transcription with custom models
ADAPTED_ACOUSTIC_ID = None  # guid of a custom acoustic model
ADAPTED_LANGUAGE_ID = None  # guid of a custom language model


def transcribe():
    logging.info("Starting transcription client...")


if __name__ == "__main__":
    transcribe()
```

[!INCLUDE [placeholder-replacements](../placeholder-replacement.md)]

## <a name="create-and-configure-an-http-client"></a>创建和配置 Http 客户端
首先，我们需要具有正确的基本 URL 和身份验证集的 Http 客户端。
将此代码插入 `transcribe`
```python
configuration = cris_client.Configuration()
configuration.api_key['Ocp-Apim-Subscription-Key'] = SUBSCRIPTION_KEY
configuration.host = "https://{}.cris.azure.cn".format(SERVICE_REGION)

# create the client object and authenticate
client = cris_client.ApiClient(configuration)

# create an instance of the transcription api class
transcription_api = cris_client.CustomSpeechTranscriptionsApi(api_client=client)
```

## <a name="generate-a-transcription-request"></a>生成听录请求
接下来，我们将生成听录请求。 将此代码添加到 `transcribe`
```python
transcription_definition = cris_client.TranscriptionDefinition(
    name=NAME, description=DESCRIPTION, locale=LOCALE, recordings_url=RECORDINGS_BLOB_URI
)
```

## <a name="send-the-request-and-check-its-status"></a>发送请求并查看其状态
现在我们将请求发布到语音服务并检查初始响应代码。 此响应代码将仅指示服务是否已收到请求。 该服务将在响应标头中返回一个 URL，这是它将存储听录状态的位置。
```python
data, status, headers = transcription_api.create_transcription_with_http_info(transcription_definition)

# extract transcription location from the headers
transcription_location: str = headers["location"]

# get the transcription Id from the location URI
created_transcription: str = transcription_location.split('/')[-1]

logging.info("Created new transcription with id {}".format(created_transcription))
```

## <a name="wait-for-the-transcription-to-complete"></a>等待听录完成
由于服务以异步方式处理听录，因此需要时常轮询其状态。 我们每 5 秒查看一次。

我们将枚举此语音服务资源正在处理的所有听录，并查找我们创建的听录。

下面是一个轮询代码，其中显示了除成功完成之外的所有状态，我们会在下一步完成该操作。
```python
logging.info("Checking status.")

completed = False

while not completed:
    running, not_started = 0, 0

    # get all transcriptions for the user
    transcriptions: List[cris_client.Transcription] = transcription_api.get_transcriptions()

    # for each transcription in the list we check the status
    for transcription in transcriptions:
        if transcription.status in ("Failed", "Succeeded"):
            # we check to see if it was the transcription we created from this client
            if created_transcription != transcription.id:
                continue

            completed = True

            if transcription.status == "Succeeded":
            else:
                logging.info("Transcription failed :{}.".format(transcription.status_message))
                break
        elif transcription.status == "Running":
            running += 1
        elif transcription.status == "NotStarted":
            not_started += 1

    logging.info("Transcriptions status: "
            "completed (this transcription): {}, {} running, {} not started yet".format(
                completed, running, not_started))

    # wait for 5 seconds
    time.sleep(5)
```

## <a name="display-the-transcription-results"></a>显示听录结果
服务成功完成听录后，结果将存储在可从状态响应中获取的其他 URL 中。

此处我们获取并显示了 JSON 结果。
```python
results_uri = transcription.results_urls["channel_0"]
results = requests.get(results_uri)
logging.info("Transcription succeeded. Results: ")
logging.info(results.content.decode("utf-8"))
```

## <a name="check-your-code"></a>查看代码
此时，代码应如下所示：（我们已向此版本添加了一些注释）
```python
#!/usr/bin/env python
# coding: utf-8

# Copyright (c) Microsoft. All rights reserved.
# Licensed under the MIT license. See LICENSE.md file in the project root for full license information.

from typing import List

import logging
import sys
import requests
import time
import swagger_client as cris_client


logging.basicConfig(stream=sys.stdout, level=logging.DEBUG, format="%(message)s")

# Your subscription key and region for the Speech service
SUBSCRIPTION_KEY = "YourSubscriptionKey"
SERVICE_REGION = "YourServiceRegion"

NAME = "Simple transcription"
DESCRIPTION = "Simple transcription description"

LOCALE = "en-US"
RECORDINGS_BLOB_URI = "<Your SAS Uri to the recording>"

# Set subscription information when doing transcription with custom models
ADAPTED_ACOUSTIC_ID = None  # guid of a custom acoustic model
ADAPTED_LANGUAGE_ID = None  # guid of a custom language model


def transcribe():
    logging.info("Starting transcription client...")

    # configure API key authorization: subscription_key
    configuration = cris_client.Configuration()
    configuration.api_key['Ocp-Apim-Subscription-Key'] = SUBSCRIPTION_KEY
    configuration.host = "https://{}.cris.azure.cn".format(SERVICE_REGION)

    # create the client object and authenticate
    client = cris_client.ApiClient(configuration)

    # create an instance of the transcription api class
    transcription_api = cris_client.CustomSpeechTranscriptionsApi(api_client=client)

    # Use base models for transcription. Comment this block if you are using a custom model.
    # Note: you can specify additional transcription properties by passing a
    # dictionary in the properties parameter. See
    # https://docs.azure.cn/cognitive-services/speech-service/batch-transcription
    # for supported parameters.
    transcription_definition = cris_client.TranscriptionDefinition(
        name=NAME, description=DESCRIPTION, locale=LOCALE, recordings_url=RECORDINGS_BLOB_URI
    )

    # Uncomment this block to use custom models for transcription.
    # Model information (ADAPTED_ACOUSTIC_ID and ADAPTED_LANGUAGE_ID) must be set above.
    # if ADAPTED_ACOUSTIC_ID is None or ADAPTED_LANGUAGE_ID is None:
    #     logging.info("Custom model ids must be set to when using custom models")
    # transcription_definition = cris_client.TranscriptionDefinition(
    #     name=NAME, description=DESCRIPTION, locale=LOCALE, recordings_url=RECORDINGS_BLOB_URI,
    #     models=[cris_client.ModelIdentity(ADAPTED_ACOUSTIC_ID), cris_client.ModelIdentity(ADAPTED_LANGUAGE_ID)]
    # )

    data, status, headers = transcription_api.create_transcription_with_http_info(transcription_definition)

    # extract transcription location from the headers
    transcription_location: str = headers["location"]

    # get the transcription Id from the location URI
    created_transcription: str = transcription_location.split('/')[-1]

    logging.info("Created new transcription with id {}".format(created_transcription))

    logging.info("Checking status.")

    completed = False

    while not completed:
        running, not_started = 0, 0

        # get all transcriptions for the user
        transcriptions: List[cris_client.Transcription] = transcription_api.get_transcriptions()

        # for each transcription in the list we check the status
        for transcription in transcriptions:
            if transcription.status in ("Failed", "Succeeded"):
                # we check to see if it was the transcription we created from this client
                if created_transcription != transcription.id:
                    continue

                completed = True

                if transcription.status == "Succeeded":
                    results_uri = transcription.results_urls["channel_0"]
                    results = requests.get(results_uri)
                    logging.info("Transcription succeeded. Results: ")
                    logging.info(results.content.decode("utf-8"))
                else:
                    logging.info("Transcription failed :{}.".format(transcription.status_message))
                    break
            elif transcription.status == "Running":
                running += 1
            elif transcription.status == "NotStarted":
                not_started += 1

        logging.info("Transcriptions status: "
                "completed (this transcription): {}, {} running, {} not started yet".format(
                    completed, running, not_started))

        # wait for 5 seconds
        time.sleep(5)

    input("Press any key...")


if __name__ == "__main__":
    transcribe()
```

## <a name="build-and-run-your-app"></a>生成并运行应用

现在，可以使用语音服务构建应用并测试语音识别。

## <a name="next-steps"></a>后续步骤

[!INCLUDE [footer](./footer.md)]
