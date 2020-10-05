---
title: 使用 Azure 媒体服务 v3 .NET 从策略获取签名密钥
description: 本主题说明如何使用媒体服务 v3 .NET SDK 从现有策略中获取签名密钥。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
origin.date: 08/31/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.custom: seodec18
ms.openlocfilehash: d154f0c6ffc625036ee3b998fb477f4f7ba3f636
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245647"
---
# <a name="get-a-signing-key-from-the-existing-policy"></a>从现有策略获取签名密钥

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Google Widevine 内容保护服务目前在 Azure 中国区域不可用。

V3 API 的主要设计原则之一是使 API 更安全。 v3 API 不会在 **Get** 或 **List** 操作中返回机密或凭据。 请参阅此处的详细说明：有关详细信息，请参阅 [RBAC 和媒体服务帐户](rbac-overview.md)

本文中的示例演示如何使用 .NET 从现有策略中获取签名密钥。 
 
## <a name="download"></a>下载 

使用以下命令将包含完整 .NET 示例的 GitHub 存储库克隆到计算机：  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
带有机密的 ContentKeyPolicy 示例位于 [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM) 文件夹中。

## <a name="get-contentkeypolicy-with-secrets"></a>获取带有机密的 ContentKeyPolicy 

若要访问密钥，请使用 **GetPolicyPropertiesWithSecretsAsync**，如下例所示。

```c#
private static async Task<ContentKeyPolicy> GetOrCreateContentKeyPolicyAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string contentKeyPolicyName,
    byte[] tokenSigningKey)
{
    ContentKeyPolicy policy = await client.ContentKeyPolicies.GetAsync(resourceGroupName, accountName, contentKeyPolicyName);

    if (policy == null)
    {
        ContentKeyPolicySymmetricTokenKey primaryKey = new ContentKeyPolicySymmetricTokenKey(tokenSigningKey);
        List<ContentKeyPolicyTokenClaim> requiredClaims = new List<ContentKeyPolicyTokenClaim>()
        {
            ContentKeyPolicyTokenClaim.ContentKeyIdentifierClaim
        };
        List<ContentKeyPolicyRestrictionTokenKey> alternateKeys = null;
        ContentKeyPolicyTokenRestriction restriction 
            = new ContentKeyPolicyTokenRestriction(Issuer, Audience, primaryKey, ContentKeyPolicyRestrictionTokenType.Jwt, alternateKeys, requiredClaims);

        ContentKeyPolicyPlayReadyConfiguration playReadyConfig = ConfigurePlayReadyLicenseTemplate();
        ContentKeyPolicyWidevineConfiguration widevineConfig = ConfigureWidevineLicenseTempate();
        // ContentKeyPolicyFairPlayConfiguration fairplayConfig = ConfigureFairPlayPolicyOptions();

        List<ContentKeyPolicyOption> options = new List<ContentKeyPolicyOption>();

        options.Add(
            new ContentKeyPolicyOption()
            {
                Configuration = playReadyConfig,
                // If you want to set an open restriction, use
                // Restriction = new ContentKeyPolicyOpenRestriction()
                Restriction = restriction
            });

     // add CBCS ContentKeyPolicyOption into the list
     //   options.Add(
     //       new ContentKeyPolicyOption()
     //       {
     //           Configuration = fairplayConfig,
     //           Restriction = restriction,
     //           Name = "ContentKeyPolicyOption_CBCS"
     //       });

        policy = await client.ContentKeyPolicies.CreateOrUpdateAsync(resourceGroupName, accountName, contentKeyPolicyName, options);
    }
    else
    {
        // Get the signing key from the existing policy.
        var policyProperties = await client.ContentKeyPolicies.GetPolicyPropertiesWithSecretsAsync(resourceGroupName, accountName, contentKeyPolicyName);
        var restriction = policyProperties.Options[0].Restriction as ContentKeyPolicyTokenRestriction;
        if (restriction != null)
        {
            var signingKey = restriction.PrimaryVerificationKey as ContentKeyPolicySymmetricTokenKey;
            if (signingKey != null)
            {
                TokenSigningKey = signingKey.KeyValue;
            }
        }
    }
    return policy;
}
```

## <a name="next-steps"></a>后续步骤

[设计带访问控制的多 DRM 内容保护系统](design-multi-drm-system-with-access-control.md) 
