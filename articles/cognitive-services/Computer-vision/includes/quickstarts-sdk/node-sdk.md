---
title: 快速入门：适用于 Node.js 的计算机视觉客户端库
description: 通过本快速入门开始使用适用于 Node.js 的计算机视觉客户端库
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 10/16/2020
ms.author: v-johya
ms.custom: devx-track-js
ms.openlocfilehash: 9f883ee971e81ac3cb49afbdcf32c8908894704b
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127640"
---
<a name="HOLTop"></a>

[参考文档](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/?view=azure-node-latest) | [库源代码](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-computervision) | [包 (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-computervision) | [示例](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/details/cognitive-services/)
* 最新版本的 [Node.js](https://nodejs.org/)
* 拥有 Azure 订阅后，在 Azure 门户中<a href="https://portal.azure.cn/#create/Microsoft.CognitiveServicesComputerVision"  title="创建计算机视觉资源"  target="_blank">创建计算机视觉资源 <span class="docon docon-navigate-external x-hidden-focus"></span></a>，获取密钥和终结点。 部署后，单击“转到资源”。
    * 需要从创建的资源获取密钥和终结点，以便将应用程序连接到计算机视觉服务。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    * 可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。
* 为密钥和终结点 URL [创建环境变量](/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication)，分别将其命名为 `COMPUTER_VISION_SUBSCRIPTION_KEY` 和 `COMPUTER_VISION_ENDPOINT`。

## <a name="setting-up"></a>设置

### <a name="create-a-new-nodejs-application"></a>创建新的 Node.js 应用程序

在控制台窗口（例如 cmd、PowerShell 或 Bash）中，为应用创建一个新目录并导航到该目录。

```console
mkdir myapp && cd myapp
```

运行 `npm init` 命令以使用 `package.json` 文件创建一个 node 应用程序。

```console
npm init
```

### <a name="install-the-client-library"></a>安装客户端库

安装 `ms-rest-azure` 和 `@azure/cognitiveservices-computervision` NPM 包:

```console
npm install @azure/cognitiveservices-computervision
```

应用的 `package.json` 文件将使用依赖项进行更新。

### <a name="prepare-the-nodejs-script"></a>准备 Node.js 脚本

创建新文件 *index.js*，将其在文本编辑器中打开。 添加以下 import 语句。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

然后定义函数 `computerVision`，并声明一个包含主函数和回调函数的异步系列。 我们会将快速入门代码添加到主函数中，并调用脚本底部的 `computerVision`。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

## <a name="object-model"></a>对象模型

以下类和接口将处理计算机视觉 Node.js SDK 的某些主要功能。

|名称|说明|
|---|---|
| [ComputerVisionClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/computervisionclient?view=azure-node-latest) | 所有计算机视觉功能都需要此类。 可以使用订阅信息实例化此类，然后使用它来执行大多数图像操作。|
|[VisualFeatureTypes](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/visualfeaturetypes?view=azure-node-latest)| 此枚举定义可在标准分析操作中执行的不同类型的图像分析。 请根据需求指定一组 **VisualFeatureTypes** 值。 |

## <a name="code-examples"></a>代码示例

这些代码片段演示如何使用适用于 Node.js 的计算机视觉客户端库执行以下任务：

* [对客户端进行身份验证](#authenticate-the-client)
* [分析图像](#analyze-an-image)
* [读取印刷体文本和手写文本](#read-printed-and-handwritten-text)

## <a name="authenticate-the-client"></a>验证客户端

为资源的 Azure 终结点和密钥创建变量。 如果在启动应用程序后创建了环境变量，则需要关闭再重新打开运行该应用程序的编辑器、IDE 或 shell 才能访问该变量。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

使用终结点和密钥实例化某个客户端。 使用密钥和终结点创建 [ApiKeyCredentials](https://docs.microsoft.com/python/api/msrest/msrest.authentication.apikeycredentials?view=azure-python) 对象，然后使用它创建 [ComputerVisionClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/computervisionclient?view=azure-node-latest) 对象。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

## <a name="analyze-an-image"></a>分析图像

此部分的代码通过分析远程图像来提取各种视觉特征。 可以在客户端对象的 **analyzeImage** 方法中执行这些操作，也可以使用单个方法来调用它们。 有关详细信息，请参阅[参考文档](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/?view=azure-node-latest)。

> [!NOTE]
> 还可以分析本地图像。 请参阅 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js) 上的示例代码以了解涉及本地图像的方案。

### <a name="get-image-description"></a>获取图像说明

下面的代码获取为图像生成的描述文字列表。 有关更多详细信息，请参阅[描述图像](../../concept-describing-images.md)。

首先，定义要分析的图像的 URL：

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

然后添加以下代码，用于获取图像详细信息并将其输出到控制台。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="get-image-category"></a>获取图像类别

下面的代码获取所检测到的图像类别。 有关更多详细信息，请参阅[对图像进行分类](../../concept-categorizing-images.md)。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

定义帮助程序函数 `formatCategories`：

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="get-image-tags"></a>获取图像标记

以下代码获取图像中检测到的标记集。 有关更多详细信息，请参阅[内容标记](../../concept-tagging-images.md)。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

定义帮助程序函数 `formatTags`：

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="detect-objects"></a>检测物体

以下代码检测图像中的常见物体并将其输出到控制台。 有关更多详细信息，请参阅[物体检测](../../concept-object-detection.md)。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

定义帮助程序函数 `formatRectObjects` 以返回上、左、下、右坐标以及宽度和高度。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="detect-brands"></a>检测品牌

以下代码检测图像中的公司品牌和徽标，并将其输出到控制台。 有关更多详细信息，请参阅[品牌检测](../../concept-brand-detection.md)。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="detect-faces"></a>检测人脸

下面的代码返回图像中检测到的人脸及其矩形坐标，以及选择面属性。 有关更多详细信息，请参阅[人脸检测](../../concept-detecting-faces.md)。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

定义帮助程序函数 `formatRectFaces`：

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="detect-adult-racy-or-gory-content"></a>检测成人、色情或血腥内容

以下代码输出图像中检测到的成人内容。 有关更多详细信息，请参阅[成人、色情或血腥内容](../../concept-detecting-adult-content.md)。

定义要使用的图像的 URL：

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

然后添加以下代码来检测成人内容，并将结果输出到控制台。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="get-image-color-scheme"></a>获取图像配色方案

以下代码输出图像中检测到的颜色属性，如主色和主题色。 有关更多详细信息，请参阅[配色方案](../../concept-detecting-color-schemes.md)。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

定义帮助程序函数 `printColorScheme`，将颜色方案的详细信息输出到控制台。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="get-domain-specific-content"></a>获取特定于域的内容

计算机视觉可以使用专用模型对图像进行进一步分析。 有关更多详细信息，请参阅[特定于域的内容](../../concept-detecting-domain-content.md)。

首先，定义要分析的图像的 URL：

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

以下代码分析了图像中检测到的地标的相关数据。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

定义帮助程序函数 `formatRectDomain`，分析有关检测到的地标的位置数据。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="get-the-image-type"></a>获取图像类型

以下代码输出有关图像类型的信息&mdash;无论它是剪贴画还是线条图。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

定义帮助程序函数 `describeType`：

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

## <a name="extract-text-ocr-with-read"></a>通过 Read 提取文本 (OCR)

计算机视觉可以提取图像中的可见文本，并将其转换为字符流。 此示例使用读取操作。

> [!NOTE]
> 还可以从本地图像读取文本。 请参阅 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js) 上的示例代码以了解涉及本地图像的方案。

### <a name="set-up-test-images"></a>设置测试图像

保存要从中提取文本的图像的 URL 的引用。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

### <a name="call-the-read-api"></a>调用读取 API

添加以下代码，该代码针对给定图像调用 `readTextFromURL` 和 `readTextFromFile` 函数。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

定义 `readTextFromURL` 和 `readTextFromFile` 函数。 它们调用客户端对象上的 read 和 readInStream 方法，这些方法返回操作 ID 并启动异步进程来读取图像的内容 。 然后它们使用操作 ID 来检查操作状态，直到返回结果。 然后它们会返回提取的结果。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

然后定义帮助程序函数 `printRecText`，该函数将读取操作的结果输出到控制台。

```javascript
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License. 
 */
// <snippet_imports>
'use strict';

const async = require('async');
const fs = require('fs');
const https = require('https');
const path = require("path");
const createReadStream = require('fs').createReadStream
const sleep = require('util').promisify(setTimeout);
const ComputerVisionClient = require('@azure/cognitiveservices-computervision').ComputerVisionClient;
const ApiKeyCredentials = require('@azure/ms-rest-js').ApiKeyCredentials;
// </snippet_imports>

/**
 * Computer Vision example
 * 
 * Prerequisites: 
 *  - Node.js 8.0+
 *  - Install the Computer Vision SDK: @azure/cognitiveservices-computervision (See https://www.npmjs.com/package/@azure/cognitiveservices-computervision) by running
 *    the following command in this directory:
 *       npm install
 *  - Set your subscription key and endpoint into your environment variables COMPUTER_VISION_ENDPOINT and COMPUTER_VISION_SUBSCRIPTION_KEY.
 *       An example of COMPUTER_VISION_ENDPOINT will look like:         https://api.cognitive.azure.cn
 *       An example of COMPUTER_VISION_SUBSCRIPTION_KEY will look like: 0123456789abcdef0123456789abcdef
 *  - The DESCRIBE IMAGE example uses a local image celebrities.jpg, which will be downloaded on demand.
 *  - The READ (the API for performing Optical Character Recognition or doing text retrieval from PDF) example uses local images and a PDF files, which will be downloaded on demand.
 * 
 * How to run:
 *  - This quickstart can be run all at once (node ComputerVisionQuickstart.js from the command line) or used to copy/paste sections as needed. 
 *    If sections are extracted, make sure to copy/paste the authenticate section too, as each example relies on it.
 *
 * Resources:
 *  - Node SDK: https://docs.microsoft.com/javascript/api/azure-cognitiveservices-computervision/?view=azure-node-latest
 *  - Documentation: /cognitive-services/computer-vision/
 *  - API v3.0: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
 * 
 * Examples included in this quickstart:
 * Authenticate, Describe Image, Detect Faces, Detect Objects, Detect Tags, Detect Type, 
 * Detect Category, Detect Brand, Detect Color Scheme, Detect Domain-specific Content, Detect Adult Content
 * Generate Thumbnail, Recognize Printed & Handwritten Text using Read API.
 */

// <snippet_vars>
/**
 * AUTHENTICATE
 * This single client is used for all examples.
 */
const key = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
const endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!key) { throw new Error('Set your environment variables for your subscription key in COMPUTER_VISION_SUBSCRIPTION_KEY and endpoint in COMPUTER_VISION_ENDPOINT.'); }
// </snippet_vars>

// <snippet_client>
const computerVisionClient = new ComputerVisionClient(
  new ApiKeyCredentials({ inHeader: { 'Ocp-Apim-Subscription-Key': key } }), endpoint);
// </snippet_client>
/**
 * END - Authenticate
 */

// <snippet_functiondef_begin>
function computerVision() {
  async.series([
    async function () {
      // </snippet_functiondef_begin>

      /**
       * DESCRIBE IMAGE
       * Describes what the main objects or themes are in an image.
       * Describes both a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('DESCRIBE IMAGE');
      console.log();

      // <snippet_describe_image>
      const describeURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_describe_image>

      const describeImagePath = __dirname + '\\celebrities.jpg';
      try {
        await downloadFilesToLocal(describeURL, describeImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      // <snippet_describe>
      // Analyze URL image
      console.log('Analyzing URL image to describe...', describeURL.split('/').pop());
      const caption = (await computerVisionClient.describeImage(describeURL)).captions[0];
      console.log(`This may be ${caption.text} (${caption.confidence.toFixed(2)} confidence)`);
      // </snippet_describe>

      // Analyze local image
      console.log('\nAnalyzing local image to describe...', path.basename(describeImagePath));
      // DescribeImageInStream takes a function that returns a ReadableStream, NOT just a ReadableStream instance.
      const captionLocal = (await computerVisionClient.describeImageInStream(
        () => createReadStream(describeImagePath))).captions[0];
      console.log(`This may be ${caption.text} (${captionLocal.confidence.toFixed(2)} confidence)`);
      /**
       * END - Describe Image
       */
      console.log();

      /**
       * DETECT FACES
       * This example detects faces and returns its:
       *     gender, age, location of face (bounding box), confidence score, and size of face.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT FACES');
      console.log();

      // <snippet_faces>
      const facesImageURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/faces.jpg';

      // Analyze URL image.
      console.log('Analyzing faces in image...', facesImageURL.split('/').pop());
      // Get the visual feature for 'Faces' only.
      const faces = (await computerVisionClient.analyzeImage(facesImageURL, { visualFeatures: ['Faces'] })).faces;

      // Print the bounding box, gender, and age from the faces.
      if (faces.length) {
        console.log(`${faces.length} face${faces.length == 1 ? '' : 's'} found:`);
        for (const face of faces) {
          console.log(`    Gender: ${face.gender}`.padEnd(20)
            + ` Age: ${face.age}`.padEnd(10) + `at ${formatRectFaces(face.faceRectangle)}`);
        }
      } else { console.log('No faces found.'); }
      // </snippet_faces>

      // <snippet_formatfaces>
      // Formats the bounding box
      function formatRectFaces(rect) {
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12)
          + `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_formatfaces>

      /**
       * END - Detect Faces
       */
      console.log();

      /**
       * DETECT OBJECTS
       * Detects objects in URL image:
       *     gives confidence score, shows location of object in image (bounding box), and object size. 
       */
      console.log('-------------------------------------------------');
      console.log('DETECT OBJECTS');
      console.log();

      // <snippet_objects>
      // Image of a dog
      const objectURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-node-sdk-samples/master/Data/image.jpg';

      // Analyze a URL image
      console.log('Analyzing objects in image...', objectURL.split('/').pop());
      const objects = (await computerVisionClient.analyzeImage(objectURL, { visualFeatures: ['Objects'] })).objects;
      console.log();

      // Print objects bounding box and confidence
      if (objects.length) {
        console.log(`${objects.length} object${objects.length == 1 ? '' : 's'} found:`);
        for (const obj of objects) { console.log(`    ${obj.object} (${obj.confidence.toFixed(2)}) at ${formatRectObjects(obj.rectangle)}`); }
      } else { console.log('No objects found.'); }
      // </snippet_objects>

      // <snippet_objectformat>
      // Formats the bounding box
      function formatRectObjects(rect) {
        return `top=${rect.y}`.padEnd(10) + `left=${rect.x}`.padEnd(10) + `bottom=${rect.y + rect.h}`.padEnd(12)
          + `right=${rect.x + rect.w}`.padEnd(10) + `(${rect.w}x${rect.h})`;
      }
      // </snippet_objectformat>
      /**
       * END - Detect Objects
       */
      console.log();

      /**
       * DETECT TAGS  
       * Detects tags for an image, which returns:
       *     all objects in image and confidence score.
       */
      // <snippet_tags>
      console.log('-------------------------------------------------');
      console.log('DETECT TAGS');
      console.log();

      // Image of different kind of dog.
      const tagsURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing tags in image...', tagsURL.split('/').pop());
      const tags = (await computerVisionClient.analyzeImage(tagsURL, { visualFeatures: ['Tags'] })).tags;
      console.log(`Tags: ${formatTags(tags)}`);
      // </snippet_tags>

      // <snippet_tagsformat>
      // Format tags for display
      function formatTags(tags) {
        return tags.map(tag => (`${tag.name} (${tag.confidence.toFixed(2)})`)).join(', ');
      }
      // </snippet_tagsformat>
      /**
       * END - Detect Tags
       */
      console.log();

      /**
       * DETECT TYPE
       * Detects the type of image, says whether it is clip art, a line drawing, or photograph).
       */
      console.log('-------------------------------------------------');
      console.log('DETECT TYPE');
      console.log();

      // <snippet_imagetype>
      const typeURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-python-sdk-samples/master/samples/vision/images/make_things_happen.jpg';

      // Analyze URL image
      console.log('Analyzing type in image...', typeURLImage.split('/').pop());
      const types = (await computerVisionClient.analyzeImage(typeURLImage, { visualFeatures: ['ImageType'] })).imageType;
      console.log(`Image appears to be ${describeType(types)}`);
      // </snippet_imagetype>

      // <snippet_imagetype_describe>
      function describeType(imageType) {
        if (imageType.clipArtType && imageType.clipArtType > imageType.lineDrawingType) return 'clip art';
        if (imageType.lineDrawingType && imageType.clipArtType < imageType.lineDrawingType) return 'a line drawing';
        return 'a photograph';
      }
      // </snippet_imagetype_describe>
      /**
       * END - Detect Type
       */
      console.log();

      /**
       * DETECT CATEGORY
       * Detects the categories of an image. Two different images are used to show the scope of the features.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT CATEGORY');
      console.log();

      // <snippet_categories>
      const categoryURLImage = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';

      // Analyze URL image
      console.log('Analyzing category in image...', categoryURLImage.split('/').pop());
      const categories = (await computerVisionClient.analyzeImage(categoryURLImage)).categories;
      console.log(`Categories: ${formatCategories(categories)}`);
      // </snippet_categories>

      // <snippet_categories_format>
      // Formats the image categories
      function formatCategories(categories) {
        categories.sort((a, b) => b.score - a.score);
        return categories.map(cat => `${cat.name} (${cat.score.toFixed(2)})`).join(', ');
      }
      // </snippet_categories_format>
      /**
       * END - Detect Categories
       */
      console.log();

      /**
       * DETECT BRAND
       * Detects brands and logos that appear in an image.
       */
      console.log('-------------------------------------------------');
      console.log('DETECT BRAND');
      console.log();

      // <snippet_brands>
      const brandURLImage = '/cognitive-services/computer-vision/images/red-shirt-logo.jpg';

      // Analyze URL image
      console.log('Analyzing brands in image...', brandURLImage.split('/').pop());
      const brands = (await computerVisionClient.analyzeImage(brandURLImage, { visualFeatures: ['Brands'] })).brands;

      // Print the brands found
      if (brands.length) {
        console.log(`${brands.length} brand${brands.length != 1 ? 's' : ''} found:`);
        for (const brand of brands) {
          console.log(`    ${brand.name} (${brand.confidence.toFixed(2)} confidence)`);
        }
      } else { console.log(`No brands found.`); }
      // </snippet_brands>
      console.log();

      /**
       * DETECT COLOR SCHEME
       * Detects the color scheme of an image, including foreground, background, dominant, and accent colors.  
       */
      console.log('-------------------------------------------------');
      console.log('DETECT COLOR SCHEME');
      console.log();

      // <snippet_colors>
      const colorURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';

      // Analyze URL image
      console.log('Analyzing image for color scheme...', colorURLImage.split('/').pop());
      console.log();
      const color = (await computerVisionClient.analyzeImage(colorURLImage, { visualFeatures: ['Color'] })).color;
      printColorScheme(color);
      // </snippet_colors>

      // <snippet_colors_print>
      // Print a detected color scheme
      function printColorScheme(colors) {
        console.log(`Image is in ${colors.isBwImg ? 'black and white' : 'color'}`);
        console.log(`Dominant colors: ${colors.dominantColors.join(', ')}`);
        console.log(`Dominant foreground color: ${colors.dominantColorForeground}`);
        console.log(`Dominant background color: ${colors.dominantColorBackground}`);
        console.log(`Suggested accent color: #${colors.accentColor}`);
      }
      // </snippet_colors_print>
      /**
       * END - Detect Color Scheme
       */
      console.log();

      /**
       * GENERATE THUMBNAIL
       * This example generates a thumbnail image of a specified size, from a URL and a local image.
       */
      console.log('-------------------------------------------------');
      console.log('GENERATE THUMBNAIL');
      console.log();
      // Image of a dog.
      const dogURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample16.png';
      console.log('Generating thumbnail...')
      await computerVisionClient.generateThumbnail(100, 100, dogURL, { smartCropping: true })
        .then((thumbResponse) => {
          const destination = fs.createWriteStream("thumb.png")
          thumbResponse.readableStreamBody.pipe(destination)
          console.log('Thumbnail saved.') // Saves into root folder
        })
      console.log();
      /**
       * END - Generate Thumbnail
       */

      /**
      * DETECT DOMAIN-SPECIFIC CONTENT
      * Detects landmarks or celebrities.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT DOMAIN-SPECIFIC CONTENT');
      console.log();

      // <snippet_domain_image>
      const domainURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg';
      // </snippet_domain_image>

      // <snippet_landmarks>
      // Analyze URL image
      console.log('Analyzing image for landmarks...', domainURLImage.split('/').pop());
      const domain = (await computerVisionClient.analyzeImageByDomain('landmarks', domainURLImage)).result.landmarks;

      // Prints domain-specific, recognized objects
      if (domain.length) {
        console.log(`${domain.length} ${domain.length == 1 ? 'landmark' : 'landmarks'} found:`);
        for (const obj of domain) {
          console.log(`    ${obj.name}`.padEnd(20) + `(${obj.confidence.toFixed(2)} confidence)`.padEnd(20) + `${formatRectDomain(obj.faceRectangle)}`);
        }
      } else {
        console.log('No landmarks found.');
      }
      // </snippet_landmarks>

      // <snippet_landmarks_rect>
      // Formats bounding box
      function formatRectDomain(rect) {
        if (!rect) return '';
        return `top=${rect.top}`.padEnd(10) + `left=${rect.left}`.padEnd(10) + `bottom=${rect.top + rect.height}`.padEnd(12) +
          `right=${rect.left + rect.width}`.padEnd(10) + `(${rect.width}x${rect.height})`;
      }
      // </snippet_landmarks_rect>

      console.log();

      /**
      * DETECT ADULT CONTENT
      * Detects "adult" or "racy" content that may be found in images. 
      * The score closer to 1.0 indicates racy/adult content.
      * Detection for both local and URL images.
      */
      console.log('-------------------------------------------------');
      console.log('DETECT ADULT CONTENT');
      console.log();

      // <snippet_adult_image>
      // The URL image and local images are not racy/adult. 
      // Try your own racy/adult images for a more effective result.
      const adultURLImage = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/celebrities.jpg';
      // </snippet_adult_image>

      // <snippet_adult>
      // Function to confirm racy or not
      const isIt = flag => flag ? 'is' : "isn't";

      // Analyze URL image
      console.log('Analyzing image for racy/adult content...', adultURLImage.split('/').pop());
      const adult = (await computerVisionClient.analyzeImage(adultURLImage, {
        visualFeatures: ['Adult']
      })).adult;
      console.log(`This probably ${isIt(adult.isAdultContent)} adult content (${adult.adultScore.toFixed(4)} score)`);
      console.log(`This probably ${isIt(adult.isRacyContent)} racy content (${adult.racyScore.toFixed(4)} score)`);
      // </snippet_adult>
      console.log();
      /**
      * END - Detect Adult Content
      */

      /**
        *READ API
        *
        * This example recognizes both handwritten and printed text, and can handle image files (.jpg/.png/.bmp) and multi-page files (.pdf and .tiff)
        * Please see REST API reference for more information:
        * Read: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005
        * Get Result Result: https://dev.cognitive.azure.cn/docs/services/computer-vision-v3-ga/operations/5d9869604be85dee480c8750
        * 
        */

      // Status strings returned from Read API. NOTE: CASING IS SIGNIFICANT.
      // Before Read 3.0, these are "Succeeded" and "Failed"
      const STATUS_SUCCEEDED = "succeeded"; 
      const STATUS_FAILED = "failed"

      console.log('-------------------------------------------------');
      console.log('READ');
      console.log();
      const printedTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/printed_text.jpg';
      const handwrittenTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/handwritten_text.jpg';

      const handwrittenImagePath = __dirname + '\\handwritten_text.jpg';
      try {
        await downloadFilesToLocal(handwrittenTextURL, handwrittenImagePath);
      } catch {
        console.log('>>> Download sample file failed. Sample cannot continue');
        process.exit(1);
      }

      console.log('\nReading URL image for text in ...', printedTextURL.split('/').pop());
      // API call returns a ReadResponse, grab the operation location (ID) from the response.
      const operationLocationUrl = await computerVisionClient.read(printedTextURL)
        .then((response) => {
          return response.operationLocation;
        });

      console.log();
      // From the operation location URL, grab the last element, the operation ID.
      const operationIdUrl = operationLocationUrl.substring(operationLocationUrl.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdUrl)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File URL image result:');
          // Print the text captured

          // Looping through: TextRecognitionResult[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();

      // With a local image, get the text.
      console.log('\Reading local image for text in ...', path.basename(handwrittenImagePath));

      // Call API, returns a Promise<Models.readInStreamResponse>
      const streamResponse = await computerVisionClient.readInStream(() => createReadStream(handwrittenImagePath))
        .then((response) => {
          return response;
        });

      console.log();
      // Get operation location from response, so you can get the operation ID.
      const operationLocationLocal = streamResponse.operationLocation
      // Get the operation ID at the end of the URL
      const operationIdLocal = operationLocationLocal.substring(operationLocationLocal.lastIndexOf('/') + 1);

      // Wait for the read operation to finish, use the operationId to get the result.
      while (true) {
        const readOpResult = await computerVisionClient.getReadResult(operationIdLocal)
          .then((result) => {
            return result;
          })
        console.log('Read status: ' + readOpResult.status)
        if (readOpResult.status === STATUS_FAILED) {
          console.log('The Read File operation has failed.')
          break;
        }
        if (readOpResult.status === STATUS_SUCCEEDED) {
          console.log('The Read File operation was a success.');
          console.log();
          console.log('Read File local image result:');
          // Print the text captured

          // Looping through: pages of result from readResults[], then Line[]
          for (const textRecResult of readOpResult.analyzeResult.readResults) {
            for (const line of textRecResult.lines) {
              console.log(line.text)
            }
          }
          break;
        }
        await sleep(1000);
      }
      console.log();
      /**
      * END - READ API
      */

      /**
       * READ PRINTED & HANDWRITTEN TEXT
       * Recognizes text from images using OCR (optical character recognition).
       * Recognition is shown for both printed and handwritten text.
       * Read 3.0 supports the following language: en (English), de (German), es (Spanish), fr (French), it (Italian), nl (Dutch) and pt (Portuguese).
       */
      console.log('-------------------------------------------------');
      console.log('READ PRINTED, HANDWRITTEN TEXT AND PDF');
      console.log();

      // <snippet_read_images>
      // URL images containing printed and/or handwritten text. 
      // The URL can point to image files (.jpg/.png/.bmp) or multi-page files (.pdf, .tiff).
      const printedTextSampleURL = 'https://moderatorsampleimages.blob.core.chinacloudapi.cn/samples/sample2.jpg';
      const multiLingualTextURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiLingual.png';
      const mixedMultiPagePDFURL = 'https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/MultiPageHandwrittenForm.pdf';
      // </snippet_read_images>

      // <snippet_read_call>
      // Recognize text in printed image from a URL
      console.log('Read printed text from URL...', printedTextSampleURL.split('/').pop());
      const printedResult = await readTextFromURL(computerVisionClient, printedTextSampleURL);
      printRecText(printedResult);

      // Recognize text in handwritten image from a local file
      
      const handwrittenImageLocalPath = __dirname + '\\handwritten_text.jpg';
      console.log('\nRead handwritten text from local file...', handwrittenImageLocalPath);
      const handwritingResult = await readTextFromFile(computerVisionClient, handwrittenImageLocalPath);
      printRecText(handwritingResult);

      // Recognize multi-lingual text in a PNG from a URL
      console.log('\nRead printed multi-lingual text in a PNG from URL...', multiLingualTextURL.split('/').pop());
      const multiLingualResult = await readTextFromURL(computerVisionClient, multiLingualTextURL);
      printRecText(multiLingualResult);

      // Recognize printed text and handwritten text in a PDF from a URL
      console.log('\nRead printed and handwritten text from a PDF from URL...', mixedMultiPagePDFURL.split('/').pop());
      const mixedPdfResult = await readTextFromURL(computerVisionClient, mixedMultiPagePDFURL);
      printRecText(mixedPdfResult);
      // </snippet_read_call>

      // <snippet_read_helper>
      // Perform read and await the result from URL
      async function readTextFromURL(client, url) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.read(url);
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }

      // Perform read and await the result from local file
      async function readTextFromFile(client, localImagePath) {
        // To recognize text in a local image, replace client.read() with readTextInStream() as shown:
        let result = await client.readInStream(() => createReadStream(localImagePath));
        // Operation ID is last path segment of operationLocation (a URL)
        let operation = result.operationLocation.split('/').slice(-1)[0];

        // Wait for read recognition to complete
        // result.status is initially undefined, since it's the result of read
        while (result.status !== STATUS_SUCCEEDED) { await sleep(1000); result = await client.getReadResult(operation); }
        return result.analyzeResult.readResults; // Return the first page of result. Replace [0] with the desired page if this is a multi-page file such as .pdf or .tiff.
      }
      // </snippet_read_helper>

      // <snippet_read_print>
      // Prints all text from Read result
      function printRecText(readResults) {
        console.log('Recognized text:');
        for (const page in readResults) {
          if (readResults.length > 1) {
            console.log(`==== Page: ${page}`);
          }
          const result = readResults[page];
          if (result.lines.length) {
            for (const line of result.lines) {
              console.log(line.words.map(w => w.text).join(' '));
            }
          }
          else { console.log('No recognized text.'); }
        }
      }
      // </snippet_read_print>

      // <snippet_read_download>
      /**
       * 
       * Download the specified file in the URL to the current local folder
       * 
       */
      function downloadFilesToLocal(url, localFileName) {
        return new Promise((resolve, reject) => {
          console.log('--- Downloading file to local directory from: ' + url);
          const request = https.request(url, (res) => {
            if (res.statusCode !== 200) {
              console.log(`Download sample file failed. Status code: ${res.statusCode}, Message: ${res.statusMessage}`);
              reject();
            }
            var data = [];
            res.on('data', (chunk) => {
              data.push(chunk);
            });
            res.on('end', () => {
              console.log('   ... Downloaded successfully');
              fs.writeFileSync(localFileName, Buffer.concat(data));
              resolve();
            });
          });
          request.on('error', function (e) {
            console.log(e.message);
            reject();
          });
          request.end();
        });
      }
      // </snippet_read_download>

      /**
       * END - Recognize Printed & Handwritten Text
       */

      console.log();
      console.log('-------------------------------------------------');
      console.log('End of quickstart.');
      // <snippet_functiondef_end>

    },
    function () {
      return new Promise((resolve) => {
        resolve();
      })
    }
  ], (err) => {
    throw (err);
  });
}

computerVision();
// </snippet_functiondef_end>
```

## <a name="run-the-application"></a>运行应用程序

在快速入门文件中使用 `node` 命令运行应用程序。

```console
node index.js
```

## <a name="clean-up-resources"></a>清理资源

如果想要清理并删除认知服务订阅，可以删除资源或资源组。 删除资源组同时也会删除与之相关联的任何其他资源。

* [门户](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
>[计算机视觉 API 参考 (Node.js)](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/?view=azure-node-latest)

* [什么是计算机视觉？](../../overview.md)
* 可以在 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js) 上找到此示例的源代码。

