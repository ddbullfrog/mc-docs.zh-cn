---
title: 使用 iOS SDK
description: 如何使用适用于 Azure 移动应用的 iOS SDK
ms.assetid: 4e8e45df-c36a-4a60-9ad4-393ec10b7eb9
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: article
origin.date: 06/25/2019
md.date: 03/23/2020
ms.author: v-tawe
ms.openlocfilehash: cd9583b1ef529d3dd6aca213f0c5a70816ec0ec2
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211668"
---
# <a name="how-to-use-ios-client-library-for-azure-mobile-apps"></a>如何使用适用于 Azure 移动应用的 iOS 客户端库

[!INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

## <a name="overview"></a>概述
本指南介绍如何使用最新的 [Azure 移动应用 iOS SDK][1] 执行常见任务。 对于 Azure 移动应用的新手，请先完成 [Azure 移动应用快速入门]，创建后端、创建表并下载预先生成的 iOS Xcode 项目。 本指南侧重于客户端 iOS SDK。 若要了解有关用于后端的服务器端 SDK 的详细信息，请参阅 Server SDK 操作方法。

## <a name="reference-documentation"></a>参考文档

iOS 客户端 SDK 的参考文档位于此处：[Azure 移动应用 iOS 客户端参考][2]。

## <a name="supported-platforms"></a>支持的平台

iOS SDK 支持适用于 iOS 8.0 版及更高版本的 Objective-C 项目、Swift 2.2 项目和 Swift 2.3 项目。

“服务器流”身份验证在呈现的 UI 中使用 WebView。  如果设备不能显示 Web 视图 UI，则需使用本产品范围外的另一种身份验证方法。  
因此这个 SDK 不适用于监视类型或同样受限制的设备。

## <a name="setup-and-prerequisites"></a><a name="Setup"></a>安装与先决条件

本指南假设已创建了包含表的后端。 本指南假设该表的架构与这些教程中的表相同。 本指南还假设在代码中引用了 `MicrosoftAzureMobile.framework` 并导入了 `MicrosoftAzureMobile/MicrosoftAzureMobile.h`。

## <a name="how-to-create-client"></a><a name="create-client"></a>如何：创建客户端

若要在项目中访问 Azure 移动应用后端，请创建 `MSClient`。 将 `AppUrl` 替换为应用 URL。 可以将 `gatewayURLString` 和 `applicationKey` 留空。 如果设置了用于身份验证的网关，请使用网关 URL 填充 `gatewayURLString` 。

**Objective-C** ：

```objc
MSClient *client = [MSClient clientWithApplicationURLString:@"AppUrl"];
```

**Swift** ：

```swift
let client = MSClient(applicationURLString: "AppUrl")
```

## <a name="how-to-create-table-reference"></a><a name="table-reference"></a>如何：创建表引用

若要访问或更新数据，请创建到后端表的引用。 将 `TodoItem` 替换为表名称

**Objective-C** ：

```objc
MSTable *table = [client tableWithName:@"TodoItem"];
```

**Swift** ：

```swift
let table = client.tableWithName("TodoItem")
```

## <a name="how-to-query-data"></a><a name="querying"></a>如何：查询数据

若要创建数据库查询，请查询 `MSTable` 对象。 以下查询将获取 `TodoItem` 中的所有项，并记录每个项的文本。

**Objective-C** ：

```objc
[table readWithCompletion:^(MSQueryResult *result, NSError *error) {
        if(error) { // error is nil if no error occurred
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) { // items is NSArray of records that match query
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift** ：

```swift
table.readWithCompletion { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```

## <a name="how-to-filter-returned-data"></a><a name="filtering"></a>如何：筛选返回的数据

可以使用许多可用选项来筛选结果。

若要使用谓词进行筛选，请使用 `NSPredicate` 和 `readWithPredicate`。 以下筛选器返回的数据只用于查找未完成的待办事项。

**Objective-C** ：

```objc
// Create a predicate that finds items where complete is false
NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
// Query the TodoItem table
[table readWithPredicate:predicate completion:^(MSQueryResult *result, NSError *error) {
        if(error) {
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) {
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift** ：

```swift
// Create a predicate that finds items where complete is false
let predicate =  NSPredicate(format: "complete == NO")
// Query the TodoItem table
table.readWithPredicate(predicate) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```

## <a name="how-to-use-msquery"></a><a name="query-object"></a>如何：使用 MSQuery

若要执行复杂查询（包括排序和分页），请使用谓词直接创建 `MSQuery` 对象：

**Objective-C** ：

```objc
MSQuery *query = [table query];
MSQuery *query = [table queryWithPredicate: [NSPredicate predicateWithFormat:@"complete == NO"]];
```

**Swift** ：

```swift
let query = table.query()
let query = table.queryWithPredicate(NSPredicate(format: "complete == NO"))
```

`MSQuery` 允许用户控制几种查询行为。

* 指定结果顺序
* 限制要返回的字段
* 限制要返回的记录数
* 指定响应中的总计数
* 指定请求中的自定义查询字符串参数
* 应用其他函数

通过对对象调用 `readWithCompletion` 来执行 `MSQuery` 查询。

## <a name="how-to-sort-data-with-msquery"></a><a name="sorting"></a>如何：使用 MSQuery 对数据排序

让我们先看一个示例，来了解如何对结果排序。 如果要对字段“text”进行升序排序，并对“complete”进行降序排序，请调用 `MSQuery` ，如下所示：

**Objective-C** ：

```objc
[query orderByAscending:@"text"];
[query orderByDescending:@"complete"];
[query readWithCompletion:^(MSQueryResult *result, NSError *error) {
        if(error) {
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) {
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift** ：

```swift
query.orderByAscending("text")
query.orderByDescending("complete")
query.readWithCompletion { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```

## <a name="how-to-limit-fields-and-expand-query-string-parameters-with-msquery"></a><a name="selecting"></a><a name="parameters"></a>如何：使用 MSQuery 限制字段和展开查询字符串参数

若要限制在查询中返回的字段，请在 **selectFields** 属性中指定字段的名称。 此示例只返回 text 和 completed 字段：

**Objective-C** ：

```objc
query.selectFields = @[@"text", @"complete"];
```

**Swift** ：

```swift
query.selectFields = ["text", "complete"]
```

若要在服务器请求中包含其他查询字符串参数（例如，某个自定义服务器端脚本要使用这些参数），请按如下所示填充 `query.parameters`：

**Objective-C** ：

```objc
query.parameters = @{
    @"myKey1" : @"value1",
    @"myKey2" : @"value2",
};
```

**Swift** ：

```swift
query.parameters = ["myKey1": "value1", "myKey2": "value2"]
```

## <a name="how-to-configure-page-size"></a><a name="paging"></a>如何：配置页面大小

在 Azure 移动应用中，页面大小控制每次从后端表提取的记录数。 `pull` 数据的调用稍后会基于此页面大小对数据进行批量处理，直到没有更多要提取的记录。

可使用 **MSPullSettings** 配置页面大小，如下所示。 默认的页面大小为 50，下面的示例将其更改为 3。

可以配置不同的页面大小，以提高性能。 如果有大量小型数据记录，增大页面大小可减少服务器往返次数。

此设置仅控制客户端侧的页面大小。 如果客户端请求的页面大小超过移动应用后端支持的大小，则页面大小受限于后端配置为支持的最大值。

此设置也是数据记录的数目  ，而不是字节大小  。

如果要增加客户端页面大小，还应增加服务器上的页面大小。 请参阅[“如何：调整表分页大小”](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)以获取执行此操作的步骤。

**Objective-C** ：

```objc
  MSPullSettings *pullSettings = [[MSPullSettings alloc] initWithPageSize:3];
  [table  pullWithQuery:query queryId:@nil settings:pullSettings
                        completion:^(NSError * _Nullable error) {
                               if(error) {
                    NSLog(@"ERROR %@", error);
                }
                           }];
```

**Swift** ：

```swift
let pullSettings = MSPullSettings(pageSize: 3)
table.pullWithQuery(query, queryId:nil, settings: pullSettings) { (error) in
    if let err = error {
        print("ERROR ", err)
    }
}
```

## <a name="how-to-insert-data"></a><a name="inserting"></a>如何：插入数据

若要插入新的表行，请创建 `NSDictionary` 并调用 `table insert`。 如果启用[动态架构]，Azure App Service 移动后端将根据 `NSDictionary` 自动生成新列。

如果未提供 `id` ，后端会自动生成新的唯一 ID。 提供自己的 `id`，以使用电子邮件地址、用户名或自己的自定义值作为 ID。 提供自己的 ID 可以让联接和业务导向型数据库逻辑变得更容易。

`result` 包含插入的新项。 根据服务器逻辑，与传递给服务器的项相比，它可能包含其他或已修改的数据。

**Objective-C** ：

```objc
NSDictionary *newItem = @{@"id": @"custom-id", @"text": @"my new item", @"complete" : @NO};
[table insert:newItem completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift** ：

```swift
let newItem = ["id": "custom-id", "text": "my new item", "complete": false]
table.insert(newItem) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let item = result {
        print("Todo Item: ", item["text"])
    }
}
```

## <a name="how-to-modify-data"></a><a name="modifying"></a>如何：修改数据

若要更新现有的行，请修改项并调用 `update`：

**Objective-C** ：

```objc
NSMutableDictionary *newItem = [oldItem mutableCopy]; // oldItem is NSDictionary
[newItem setValue:@"Updated text" forKey:@"text"];
[table update:newItem completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift** ：

```swift
if let newItem = oldItem.mutableCopy() as? NSMutableDictionary {
    newItem["text"] = "Updated text"
    table2.update(newItem as [NSObject: AnyObject], completion: { (result, error) -> Void in
        if let err = error {
            print("ERROR ", err)
        } else if let item = result {
            print("Todo Item: ", item["text"])
        }
    })
}
```

或者，提供行 ID 和更新的字段：

**Objective-C** ：

```objc
[table update:@{@"id":@"custom-id", @"text":"my EDITED item"} completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift** ：

```swift
table.update(["id": "custom-id", "text": "my EDITED item"]) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let item = result {
        print("Todo Item: ", item["text"])
    }
}
```

进行更新时，至少必须设置 `id` 属性。

## <a name="how-to-delete-data"></a><a name="deleting"></a>如何：删除数据

若要删除某个项，请对该项调用 `delete` ：

**Objective-C** ：

```objc
[table delete:item completion:^(id itemId, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item ID: %@", itemId);
    }
}];
```

**Swift** ：

```swift
table.delete(newItem as [NSObject: AnyObject]) { (itemId, error) in
    if let err = error {
        print("ERROR ", err)
    } else {
        print("Todo Item ID: ", itemId)
    }
}
```

或者，提供行 ID 来进行删除：

**Objective-C** ：

```objc
[table deleteWithId:@"37BBF396-11F0-4B39-85C8-B319C729AF6D" completion:^(id itemId, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item ID: %@", itemId);
    }
}];
```

**Swift** ：

```swift
table.deleteWithId("37BBF396-11F0-4B39-85C8-B319C729AF6D") { (itemId, error) in
    if let err = error {
        print("ERROR ", err)
    } else {
        print("Todo Item ID: ", itemId)
    }
}
```

进行删除时，至少必须设置 `id` 属性。

## <a name="how-to-call-custom-api"></a><a name="customapi"></a>如何：调用自定义 API

使用自定义 API 可以公开任何后端功能。 无需映射到表操作。 不仅能进一步控制消息，甚至还可以读取或设置标头，并更改响应正文格式。

若要调用自定义 API，请调用 `MSClient.invokeAPI`。 请求和响应内容被视为 JSON。 若要使用其他媒体类型，[请使用 `invokeAPI` 的其他重载][5]。  若要发出 `GET` 请求而不是 `POST` 请求，请将参数 `HTTPMethod` 设置为 `"GET"`，将参数 `body` 设置为 `nil`（因为 GET 请求没有消息正文）。如果自定义 API 支持其他 HTTP 谓词，请相应地更改 `HTTPMethod`。

**Objective-C** ：

```objc
[self.client invokeAPI:@"sendEmail"
                  body:@{ @"contents": @"Hello world!" }
            HTTPMethod:@"POST"
            parameters:@{ @"to": @"bill@contoso.com", @"subject" : @"Hi!" }
               headers:nil
            completion: ^(NSData *result, NSHTTPURLResponse *response, NSError *error) {
                if(error) {
                    NSLog(@"ERROR %@", error);
                } else {
                    // Do something with result
                }
            }];
```

**Swift** ：

```swift
client.invokeAPI("sendEmail",
            body: [ "contents": "Hello World" ],
            HTTPMethod: "POST",
            parameters: [ "to": "bill@contoso.com", "subject" : "Hi!" ],
            headers: nil)
            {
                (result, response, error) -> Void in
                if let err = error {
                    print("ERROR ", err)
                } else if let res = result {
                          // Do something with result
                }
        }
```

## <a name="how-to-register-push-templates-to-send-cross-platform-notifications"></a><a name="templates"></a>如何：注册推送模板以发送跨平台通知

若要注册模板，请在客户端应用中连同 **client.push registerDeviceToken** 方法一起传递模板即可。

**Objective-C** ：

```objc
[client.push registerDeviceToken:deviceToken template:iOSTemplate completion:^(NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    }
}];
```

**Swift** ：

```swift
client.push?.registerDeviceToken(NSData(), template: iOSTemplate, completion: { (error) in
    if let err = error {
        print("ERROR ", err)
    }
})
```

模板类型是 NSDictionary，可以包含采用以下格式的多个模板：

**Objective-C** ：

```objc
NSDictionary *iOSTemplate = @{ @"templateName": @{ @"body": @{ @"aps": @{ @"alert": @"$(message)" } } } };
```

**Swift** ：

```swift
let iOSTemplate = ["templateName": ["body": ["aps": ["alert": "$(message)"]]]]
```

出于安全考虑，从请求中删除所有标记。  若要将标记添加到安装或安装中的模板，请参阅[使用适用于 Azure 移动应用的 .NET 后端服务器 SDK][4]。  若要使用这些注册的模板发送通知，请参阅[通知中心 API][3]。

## <a name="how-to-handle-errors"></a><a name="errors"></a>如何：处理错误

调用 Azure 应用服务移动后端时，完成块包含 `NSError` 参数。 如果出错，此参数为非 nil 值。 应该在代码中检查此参数，并根据需要处理错误，如前面的代码片段中所示。

文件 [`<WindowsAzureMobileServices/MSError.h>`][6] 定义常量 `MSErrorResponseKey`、`MSErrorRequestKey` 和 `MSErrorServerItemKey`。 若要获取与错误相关的更多数据，请执行以下操作：

**Objective-C** ：

```objc
NSDictionary *serverItem = [error.userInfo objectForKey:MSErrorServerItemKey];
```

**Swift** ：

```swift
let serverItem = error.userInfo[MSErrorServerItemKey]
```

此外，该文件还定义每个错误代码的常量：

**Objective-C** ：

```objc
if (error.code == MSErrorPreconditionFailed) {
```

**Swift** ：

```swift
if (error.code == MSErrorPreconditionFailed) {
```

## <a name="how-to-authenticate-users-with-the-active-directory-authentication-library"></a><a name="adal"></a>如何：使用 Active Directory 身份验证库对用户进行身份验证

可以借助 Active Directory 身份验证库 (ADAL) 使用 Azure Active Directory 将用户登录到应用程序。 使用标识提供者 SDK 的客户端流身份验证优于使用 `loginWithProvider:completion:` 方法。  客户端流身份验证提供更自然的 UX 体验，并允许进行额外的自定义。

1. 根据 [How to configure App Service for Active Directory login][7] （如何为 Active Directory 登录配置应用服务）教程的说明，为 AAD 登录配置移动应用。 请务必完成注册本机客户端应用程序的可选步骤。 对于 iOS，建议重定向 URI 采用 `<app-scheme>://<bundle-id>` 格式。 有关详细信息，请参阅 [ADAL iOS 快速入门][8]。
2. 使用 Cocoapods 安装 ADAL。 编辑 Podfile 以包含以下定义，将 **YOUR-PROJECT** 替换为 Xcode 项目的名称：

        source 'https://github.com/CocoaPods/Specs.git'
        link_with ['YOUR-PROJECT']
        xcodeproj 'YOUR-PROJECT'

   Pod：

        pod 'ADALiOS'

3. 使用终端，从包含项目的目录运行 `pod install`，然后打开生成的 Xcode 工作区（而不是项目）。
4. 根据使用的语言，将以下代码添加到应用程序。 在每个应用程序中，进行以下替换：

   * 将 **INSERT-AUTHORITY-HERE** 替换为在其中预配应用程序的租户的名称。 格式应为 https://login.chinacloudapi.cn/contoso.onmicrosoft.com 。 可以在 [Azure 门户]中从 Azure Active Directory 的域选项卡复制此值。
   * 将 **INSERT-RESOURCE-ID-HERE** 替换移动应用后端的客户端 ID。 可以在门户中“Azure Active Directory 设置”  下面的“高级”  选项卡获取此客户端 ID。
   * 将 **INSERT-CLIENT-ID-HERE** 替换为从本机客户端应用程序复制的客户端 ID。
   * 将 **INSERT-REDIRECT-URI-HERE** 替换为站点的 */.auth/login/done* 终结点（使用 HTTPS 方案）。 此值应类似于 *https://contoso.chinacloudsites.cn/.auth/login/done* 。

**Objective-C** ：

```objc
#import <ADALiOS/ADAuthenticationContext.h>
#import <ADALiOS/ADAuthenticationSettings.h>
// ...
- (void) authenticate:(UIViewController*) parent
            completion:(void (^) (MSUser*, NSError*))completionBlock;
{
    NSString *authority = @"INSERT-AUTHORITY-HERE";
    NSString *resourceId = @"INSERT-RESOURCE-ID-HERE";
    NSString *clientId = @"INSERT-CLIENT-ID-HERE";
    NSURL *redirectUri = [[NSURL alloc]initWithString:@"INSERT-REDIRECT-URI-HERE"];
    ADAuthenticationError *error;
    ADAuthenticationContext *authContext = [ADAuthenticationContext authenticationContextWithAuthority:authority error:&error];
    authContext.parentController = parent;
    [ADAuthenticationSettings sharedInstance].enableFullScreen = YES;
    [authContext acquireTokenWithResource:resourceId
                                    clientId:clientId
                                redirectUri:redirectUri
                            completionBlock:^(ADAuthenticationResult *result) {
                                if (result.status != AD_SUCCEEDED)
                                {
                                    completionBlock(nil, result.error);;
                                }
                                else
                                {
                                    NSDictionary *payload = @{
                                                            @"access_token" : result.tokenCacheStoreItem.accessToken
                                                            };
                                    [client loginWithProvider:@"aad" token:payload completion:completionBlock];
                                }
                            }];
}
```

**Swift** ：

```swift
// add the following imports to your bridging header:
//        #import <ADALiOS/ADAuthenticationContext.h>
//        #import <ADALiOS/ADAuthenticationSettings.h>

func authenticate(parent: UIViewController, completion: (MSUser?, NSError?) -> Void) {
    let authority = "INSERT-AUTHORITY-HERE"
    let resourceId = "INSERT-RESOURCE-ID-HERE"
    let clientId = "INSERT-CLIENT-ID-HERE"
    let redirectUri = NSURL(string: "INSERT-REDIRECT-URI-HERE")
    var error: AutoreleasingUnsafeMutablePointer<ADAuthenticationError?> = nil
    let authContext = ADAuthenticationContext(authority: authority, error: error)
    authContext.parentController = parent
    ADAuthenticationSettings.sharedInstance().enableFullScreen = true
    authContext.acquireTokenWithResource(resourceId, clientId: clientId, redirectUri: redirectUri) { (result) in
            if result.status != AD_SUCCEEDED {
                completion(nil, result.error)
            }
            else {
                let payload: [String: String] = ["access_token": result.tokenCacheStoreItem.accessToken]
                client.loginWithProvider("aad", token: payload, completion: completion)
            }
        }
}
```

<!-- ## <a name="facebook-sdk"></a>How to: Authenticate users with the Facebook SDK for iOS -->

<!-- ## <a name="twitter-fabric"></a>How to: Authenticate users with Twitter Fabric for iOS -->


<!-- ## <a name="google-sdk"></a>How to: Authenticate users with the Google Sign-In SDK for iOS -->

<!-- Anchors. -->

[What is Mobile Services]: #what-is
[Concepts]: #concepts
[Setup and Prerequisites]: #Setup
[How to: Create the Mobile Services client]: #create-client
[How to: Create a table reference]: #table-reference
[How to: Query data from a mobile service]: #querying
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[How to: Bind data to the user interface]: #binding
[How to: Insert data into a mobile service]: #inserting
[How to: Modify data in a mobile service]: #modifying
[How to: Authenticate users]: #authentication
[Cache authentication tokens]: #caching-tokens
[How to: Upload images and large files]: #blobs
[How to: Handle errors]: #errors
[How to: Design unit tests]: #unit-testing
[How to: Customize the client]: #customizing
[Customize request headers]: #custom-headers
[Customize data type serialization]: #custom-serialization
[Next Steps]: #next-steps
[How to: Use MSQuery]: #query-object

<!-- Images. -->

<!-- URLs. -->
[Azure 移动应用快速入门]: app-service-mobile-ios-get-started.md

[Mobile Services SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[iOS SDK]: https://developer.apple.com/xcode
[Azure 门户]: https://portal.azure.cn/
[Handling Expired Tokens]: https://go.microsoft.com/fwlink/p/?LinkId=301955
[Live Connect SDK]: https://go.microsoft.com/fwlink/p/?LinkId=301960
[动态架构]: https://go.microsoft.com/fwlink/p/?LinkId=296271
[Create a table]: https://msdn.microsoft.com/library/windowsazure/jj193162.aspx
[NSDictionary object]: https://go.microsoft.com/fwlink/p/?LinkId=301965
[CLI to manage Mobile Services tables]: /cli/get-started-with-az-cli2?view=azure-cli-lastest
[Fabric Dashboard]: https://www.fabric.io/home
[Fabric for iOS - Getting Started]: https://docs.fabric.io/ios/fabric/getting-started.html
[1]: https://github.com/Azure/azure-mobile-apps-ios-client/blob/master/README.md#ios-client-sdk
[2]: https://azure.github.io/azure-mobile-apps-ios-client/
[3]: https://msdn.microsoft.com/library/azure/dn495101.aspx
[4]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#tags
[5]: https://azure.github.io/azure-mobile-services/iOS/v3/Classes/MSClient.html#//api/name/invokeAPI:data:HTTPMethod:parameters:headers:completion:
[6]: https://github.com/Azure/azure-mobile-services/blob/master/sdk/iOS/src/MSError.h
[7]: ../app-service/configure-authentication-provider-aad.md
[8]:../active-directory/develop/quickstart-v2-ios.md
