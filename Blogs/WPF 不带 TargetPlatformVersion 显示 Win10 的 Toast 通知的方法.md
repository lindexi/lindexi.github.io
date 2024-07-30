本文将告诉大家如何在 WPF 不安装 WindowsAppSDK 包，且不在 TargetFramework 带上 TargetPlatformVersion 而弹出 Win10 的 Toast 通知的方法

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文这里的 TargetPlatformVersion 指的是在 TargetFramework 里面的内容，如下面的代码里的 10.0.17763.0 就是 TargetPlatformVersion 的值

```xml
  <PropertyGroup>
    <TargetFramework>net9.0-windows10.0.17763.0</TargetFramework>
  </PropertyGroup>
```

不带 TargetPlatformVersion 即不在 TargetFramework 里加上 10.0.x 的版本号

默认微软官方推荐使用的是千年不更新的 Microsoft.Toolkit.Uwp.Notifications 库，配合设置了 TargetPlatformVersion 至少为 10.0.17763.0 版本进行 Toast 通知

其默认推荐方法的 csproj 内容大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net9.0-windows10.0.17763.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Toolkit.Uwp.Notifications" Version="7.1.3" />
  </ItemGroup>
</Project>
```

此方式需要引用 Microsoft.Toolkit.Uwp.Notifications 且在 TargetFramework 里加上 10.0.17763.0 版本。其使用方法非常简单，如下面代码即可弹出文本

```csharp
        var builder = new ToastContentBuilder()
                .AddText("林德熙是逗比")
            ;
        builder.Show();
```

然而以上方法我感觉不够清真。接下来来将告诉大家一个我感觉比较清真的方法

使用 [WPF 不安装 WindowsAppSDK 使用 WinRT 功能的方法](https://blog.lindexi.com/post/WPF-%E4%B8%8D%E5%AE%89%E8%A3%85-WindowsAppSDK-%E4%BD%BF%E7%94%A8-WinRT-%E5%8A%9F%E8%83%BD%E7%9A%84%E6%96%B9%E6%B3%95.html ) 这篇博客提到的方法，即可不用指定 TargetPlatformVersion 就可以使用 WinRT 的功能

正好 Toast 就是 WinRT 的功能

具体的做法是先取出 Microsoft.Windows.SDK.NET.dll 和 WinRT.Runtime.dll 两个文件作为引用，我这里放在了我的 `C:\lindexi\Library` 文件夹里，修改 csproj 引用这两个文件，修改之后的 csproj 文件代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

  <ItemGroup>
    <Reference Include="Microsoft.Windows.SDK.NET">
      <HintPath>C:\lindexi\Library\Microsoft.Windows.SDK.NET.dll</HintPath>
    </Reference>
    <Reference Include="WinRT.Runtime">
      <HintPath>C:\lindexi\Library\WinRT.Runtime.dll</HintPath>
    </Reference>
  </ItemGroup>
  
</Project>
```

如此可以看到 csproj 不需要加上 TargetPlatformVersion 的引用，也没有 WindowsAppSDK 的引用。看起来清真

完成以上代码之后，我在 MainWindow 的 Loaded 事件尝试弹出通知内容。先根据 <https://learn.microsoft.com/en-us/windows/apps/design/shell/tiles-and-notifications/adaptive-interactive-toasts?tabs=xml> 文档构建出 XML 代码，我这里的例子代码只显示一行文本

```xml
 <toast>
     <visual>
         <binding template='ToastText01'>
             <text id="1">显示文本内容</text>
         </binding>
     </visual>
 </toast>
```

完成构建 XML 代码之后，需要转换为 XmlDocument 对象，代码如下

```csharp
            var xmlDocument = new XmlDocument();
            // lang=xml
            var toast = """
                      <toast>
                          <visual>
                              <binding template='ToastText01'>
                                  <text id="1">显示文本内容</text>
                              </binding>
                          </visual>
                      </toast>
                      """;
            xmlDocument.LoadXml(xml: toast);
```

使用 XML 直接写比较适合简单的业务，可以看到以上的代码十分简单

除了直接编写 XML 之外，还可以使用模版辅助，如下面代码，在 ToastNotificationManager 里面获取模版，然后在模版里面添加内容

```csharp
xmlDocument = ToastNotificationManager.GetTemplateContent(ToastTemplateType.ToastText01);
XmlNodeList stringElements = xmlDocument.GetElementsByTagName("text");
stringElements[0].AppendChild(xmlDocument.CreateTextNode("显示文本内容"));
```

以上这两个方式的效果都是差不多的，大家可以选自己喜欢的方式

完成基础配置之后，接下来使用 ToastNotificationManager 将通知弹出，代码如下

```csharp
            var toastNotification = new ToastNotification(xmlDocument);
            var toastNotificationManagerForUser = ToastNotificationManager.GetDefault();
            var toastNotifier = toastNotificationManagerForUser.CreateToastNotifier(applicationId: "应用名");
            toastNotifier.Show(toastNotification);
```

以上代码有一个细节是 CreateToastNotifier 需要传入应用名，如果没有传入将炸异常，这是微软设计问题

最后别忘记了在开始调用 WinRT 之前，使用 ComWrappersSupport 进行初始化

```csharp
            global::WinRT.ComWrappersSupport.InitializeComWrappers();
```

完成之后的代码如下

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        Loaded += MainWindow_Loaded;
    }

    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        if (OperatingSystem.IsWindowsVersionAtLeast(10, 0, 15063))
        {
            global::WinRT.ComWrappersSupport.InitializeComWrappers();

            // 以下 XML 的构建，请看
            // https://learn.microsoft.com/en-us/windows/apps/design/shell/tiles-and-notifications/adaptive-interactive-toasts?tabs=xml
            var xmlDocument = new XmlDocument();
            // lang=xml
            var toast = """
                      <toast>
                          <visual>
                              <binding template='ToastText01'>
                                  <text id="1">显示文本内容</text>
                              </binding>
                          </visual>
                      </toast>
                      """;
            xmlDocument.LoadXml(xml: toast);

            var toastNotification = new ToastNotification(xmlDocument);
            var toastNotificationManagerForUser = ToastNotificationManager.GetDefault();
            var toastNotifier = toastNotificationManagerForUser.CreateToastNotifier(applicationId: "应用名");
            toastNotifier.Show(toastNotification);
        }
    }
}
```

尝试运行以上代码，就可以看到在窗口加载之后，弹出一条通知消息

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ffad2f4c67a9e53fb9121f5d807191a5a913098d/WPFDemo/LenukelbawChejeabecacar/HeregemdibeHeaqereweganilai) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ffad2f4c67a9e53fb9121f5d807191a5a913098d/WPFDemo/LenukelbawChejeabecacar/HeregemdibeHeaqereweganilai) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ffad2f4c67a9e53fb9121f5d807191a5a913098d
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ffad2f4c67a9e53fb9121f5d807191a5a913098d
```

获取代码之后，进入 WPFDemo/LenukelbawChejeabecacar/HeregemdibeHeaqereweganilai 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
