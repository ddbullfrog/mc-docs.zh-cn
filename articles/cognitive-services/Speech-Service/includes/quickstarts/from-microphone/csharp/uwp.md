---
title: 快速入门：从麦克风中识别语音，C# (UWP) - 语音服务
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
origin.date: 04/02/2020
ms.date: 10/16/2020
ms.author: v-tawe
ms.custom: devx-track-csharp
ms.openlocfilehash: 8b5b42f507a0d4f02025dbcce7b1d3a732255891
ms.sourcegitcommit: 6f66215d61c6c4ee3f2713a796e074f69934ba98
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92128887"
---
## <a name="prerequisites"></a>先决条件

准备工作：

> [!div class="checklist"]
> * [创建一个 Azure 搜索资源](../../../../overview.md#try-the-speech-service-for-free)
> * [设置开发环境并创建空项目](../../../../quickstarts/setup-platform.md?tabs=uwp&pivots=programming-language-csharp)
> * 请确保你有权访问麦克风，以便进行音频捕获
> * 需要语音 SDK 1.11.0 或更高版本。

如果尚未执行此操作，很好！ 让我们继续。

## <a name="open-your-project-in-visual-studio"></a>在 Visual Studio 中打开项目

第一步是确保在 Visual Studio 中打开项目。

## <a name="start-with-some-boilerplate-code"></a>从一些样本代码入手

添加一些代码作为项目的框架。

1. 在“解决方案资源管理器”  中打开 `MainPage.xaml`。

2. 在设计器的 XAML 视图中，将以下 XAML 代码片段插入到“Grid”标记中（位于 `<Grid>` 和 `</Grid>` 之间）：

   ```xml
   <StackPanel Orientation="Vertical" HorizontalAlignment="Center"  Margin="20,50,0,0" VerticalAlignment="Center" Width="800">
        <Button x:Name="EnableMicrophoneButton" Content="Enable Microphone"  Margin="0,0,10,0" Click="EnableMicrophone_ButtonClicked" Height="35"/>
        <Button x:Name="SpeechRecognitionButton" Content="Speech recognition with microphone input" Margin="0,10,10,0" Click="SpeechRecognitionFromMicrophone_ButtonClicked" Height="35"/>
        <StackPanel x:Name="StatusPanel" Orientation="Vertical" RelativePanel.AlignBottomWithPanel="True" RelativePanel.AlignRightWithPanel="True" RelativePanel.AlignLeftWithPanel="True">
            <TextBlock x:Name="StatusLabel" Margin="0,10,10,0" TextWrapping="Wrap" Text="Status:" FontSize="20"/>
            <Border x:Name="StatusBorder" Margin="0,0,0,0">
                <ScrollViewer VerticalScrollMode="Auto"  VerticalScrollBarVisibility="Auto" MaxHeight="200">
                    <!-- Use LiveSetting to enable screen readers to announce the status update. -->
                    <TextBlock x:Name="StatusBlock" FontWeight="Bold" AutomationProperties.LiveSetting="Assertive"
                    MaxWidth="{Binding ElementName=Splitter, Path=ActualWidth}" Margin="10,10,10,20" TextWrapping="Wrap"  />
                </ScrollViewer>
            </Border>
        </StackPanel>
    </StackPanel>
   ```

3. 在“解决方案资源管理器”中，打开代码隐藏源文件 `MainPage.xaml.cs`。 （其分组在 `MainPage.xaml` 下。）

4. 将此代码替换为以下基代码：

    ```csharp
    using System;
    using System.Text;
    using Windows.UI.Xaml;
    using Windows.UI.Xaml.Controls;
    using Windows.UI.Xaml.Media;
    using Microsoft.CognitiveServices.Speech;
    
    namespace helloworld
    {
        /// <summary>
        /// An empty page that can be used on its own or navigated to within a Frame.
        /// </summary>
        public sealed partial class MainPage : Page
        {
            public MainPage()
            {
                this.InitializeComponent();
            }
    
            private async void EnableMicrophone_ButtonClicked(object sender, RoutedEventArgs e)
            {
                bool isMicAvailable = true;
                try
                {
                    var mediaCapture = new Windows.Media.Capture.MediaCapture();
                    var settings = new Windows.Media.Capture.MediaCaptureInitializationSettings();
                    settings.StreamingCaptureMode = Windows.Media.Capture.StreamingCaptureMode.Audio;
                    await mediaCapture.InitializeAsync(settings);
                }
                catch (Exception)
                {
                    isMicAvailable = false;
                }
                if (!isMicAvailable)
                {
                    await Windows.System.Launcher.LaunchUriAsync(new Uri("ms-settings:privacy-microphone"));
                }
                else
                {
                    NotifyUser("Microphone was enabled", NotifyType.StatusMessage);
                }
            }
    
            private async void SpeechRecognitionFromMicrophone_ButtonClicked(object sender, RoutedEventArgs e)
            {
                try
                {
                }
                catch(Exception ex)
                {
                    NotifyUser($"Enable Microphone First.\n {ex.ToString()}", NotifyType.ErrorMessage);
                }
            }
    
            private enum NotifyType
            {
                StatusMessage,
                ErrorMessage
            };
    
            private void NotifyUser(string strMessage, NotifyType type)
            {
                // If called from the UI thread, then update immediately.
                // Otherwise, schedule a task on the UI thread to perform the update.
                if (Dispatcher.HasThreadAccess)
                {
                    UpdateStatus(strMessage, type);
                }
                else
                {
                    var task = Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () => UpdateStatus(strMessage, type));
                }
            }
    
            private void UpdateStatus(string strMessage, NotifyType type)
            {
                switch (type)
                {
                    case NotifyType.StatusMessage:
                        StatusBorder.Background = new SolidColorBrush(Windows.UI.Colors.Green);
                        break;
                    case NotifyType.ErrorMessage:
                        StatusBorder.Background = new SolidColorBrush(Windows.UI.Colors.Red);
                        break;
                }
                StatusBlock.Text += string.IsNullOrEmpty(StatusBlock.Text) ? strMessage : "\n" + strMessage;
    
                // Collapse the StatusBlock if it has no text to conserve real estate.
                StatusBorder.Visibility = !string.IsNullOrEmpty(StatusBlock.Text) ? Visibility.Visible : Visibility.Collapsed;
                if (!string.IsNullOrEmpty(StatusBlock.Text))
                {
                    StatusBorder.Visibility = Visibility.Visible;
                    StatusPanel.Visibility = Visibility.Visible;
                }
                else
                {
                    StatusBorder.Visibility = Visibility.Collapsed;
                    StatusPanel.Visibility = Visibility.Collapsed;
                }
                // Raise an event if necessary to enable a screen reader to announce the status update.
                var peer = Windows.UI.Xaml.Automation.Peers.FrameworkElementAutomationPeer.FromElement(StatusBlock);
                if (peer != null)
                {
                    peer.RaiseAutomationEvent(Windows.UI.Xaml.Automation.Peers.AutomationEvents.LiveRegionChanged);
                }
            }
        }
    }
    ```

## <a name="create-a-speech-configuration"></a>创建语音配置

在初始化 `SpeechRecognizer` 对象之前，需要创建一个使用订阅密钥和订阅区域的配置。 将此代码插入 `SpeechRecognitionFromMicrophone_ButtonClicked()` 方法。

> [!NOTE]
> 此示例使用 `FromSubscription()` 方法来生成 `SpeechConfig`。 有关可用方法的完整列表，请参阅 [SpeechConfig 类](https://docs.microsoft.com/dotnet/api/)

```csharp
// Creates an instance of a speech config with specified subscription key and service region.
// Replace with your own subscription key and service region (e.g., "chinaeast2").
var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
```

## <a name="initialize-a-speechrecognizer"></a>初始化 SpeechRecognizer

现在，让我们创建 `SpeechRecognizer`。 此对象是在 using 语句中创建的，以确保正确释放非托管资源。 将此代码插入语音配置下的 `SpeechRecognitionFromMicrophone_ButtonClicked()` 方法。

```csharp
using (var recognizer = new SpeechRecognizer(config))
{
}
```

## <a name="recognize-a-phrase"></a>识别短语

在 `SpeechRecognizer` 对象中，调用 `RecognizeOnceAsync()` 方法。 此方法是告知语音服务你要发送单个需识别的短语，在确定该短语后会停止识别语音。

在 using 语句中，添加以下代码。

```csharp
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
```

## <a name="display-the-recognition-results-or-errors"></a>显示识别结果（或错误）

语音服务返回识别结果后，将需要对其进行处理。 我们会简单地将结果输出到状态面板。

```csharp
 // Checks result.
StringBuilder sb = new StringBuilder();
if (result.Reason == ResultReason.RecognizedSpeech)
{
    sb.AppendLine($"RECOGNIZED: Text={result.Text}");
}
else if (result.Reason == ResultReason.NoMatch)
{
    sb.AppendLine($"NOMATCH: Speech could not be recognized.");
}
else if (result.Reason == ResultReason.Canceled)
{
    var cancellation = CancellationDetails.FromResult(result);
    sb.AppendLine($"CANCELED: Reason={cancellation.Reason}");

    if (cancellation.Reason == CancellationReason.Error)
    {
        sb.AppendLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        sb.AppendLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
        sb.AppendLine($"CANCELED: Did you update the subscription info?");
    }
}

// Update the UI
NotifyUser(sb.ToString(), NotifyType.StatusMessage);
}
```

## <a name="build-and-run-the-application"></a>生成并运行应用程序

现已准备好构建并测试应用程序。

1. 从菜单栏中，选择“构建” > “构建解决方案”以构建应用程序。 现在，编译代码时应不会提示错误。

1. 选择“调试” > “开始调试”（或按 F5）以启动应用程序。 此时将显示“helloworld”窗口。

   ![C# 中的示例 UWP 语音识别应用程序 - 快速入门](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-helloworld-window.png)

1. 选择“启用麦克风”，并在弹出访问权限请求时选择“是”。

   ![麦克风访问权限请求](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-10-access-prompt.png)

1. 选择“使用麦克风输入进行语音识别”，然后对着设备的麦克风说一个英文短语或句子。 你的语音将传输到语音服务并转录为文本，该文本将显示在窗口中。

   ![语音识别用户界面](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-11-ui-result.png)

## <a name="next-steps"></a>后续步骤

[!INCLUDE [Speech recognition basics](../../speech-to-text-next-steps.md)]

