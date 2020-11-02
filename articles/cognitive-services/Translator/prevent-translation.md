---
title: 防止翻译内容 - 翻译器
titleSuffix: Azure Cognitive Services
description: 使用翻译器防止翻译内容。 使用翻译器可以标记内容，以便不对其进行翻译。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
origin.date: 05/26/2020
ms.date: 10/22/2020
ms.author: v-johya
ms.openlocfilehash: de8ecaa70c92e0dc344f1428831ebbb822ad2e4e
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472007"
---
# <a name="how-to-prevent-translation-of-content-with-the-translator"></a>如何使用翻译器防止翻译内容

使用翻译器可以标记内容，以便不对其进行翻译。 例如，你可能想要标记本地化后没有意义的代码、品牌名称或单词/短语。

## <a name="methods-for-preventing-translation"></a>阻止翻译的方法

1. 使用 `notranslate` 标记内容。 根据设计，仅当输入 textType 设置为 HTML 时，这才起作用

   示例：

   ```html
   <span class="notranslate">This will not be translated.</span>
   <span>This will be translated. </span>
   ```
   
   ```html
   <div class="notranslate">This will not be translated.</div>
   <div>This will be translated. </div>
   ```

2. 使用 `translate="no"` 标记内容。 这仅适用于以 HTML 形式设置输入 textType 的情况

   示例：

   ```html
   <span translate="no">This will not be translated.</span>
   <span>This will be translated. </span>
   ```
   
   ```html
   <div translate="no">This will not be translated.</div>
   <div>This will be translated. </div>

3. Use the [dynamic dictionary](dynamic-dictionary.md) to prescribe a specific translation.

4. Don't pass the string to the Translator for translation.


## Next steps
> [!div class="nextstepaction"]
> [Use the Translate operation to translate text](reference/v3-0-translate.md)

<!-- Update_Description: update metedata properties -->