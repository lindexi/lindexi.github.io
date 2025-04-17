
# WPF 使用 Win10 的 WinRT 自带 Windows.Media.Ocr 实现图片转文本

世界上有很多 OCR 识别技术，本文来和大家介绍如果在 WPF 里，在运行到 win10 的设备上，通过 Windows Runtime 自带的 Windows.Media.Ocr 实现在给定的图片里面识别文本的功能

<!--more-->


<!-- CreateTime:2022/7/25 8:28:00 -->

<!-- 发布 -->


我采用的是 dotnet 6 框架版本，在 dotnet 5 和更高版本，在 dotnet 底层就删除了对 WinRT (Windows Runtime) 的底层支持，不再支持直接引用 `.winmd` 文件。移除的原因如 [Breaking change: Built-in support for WinRT is removed from .NET - .NET Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/core/compatibility/interop/5.0/built-in-support-for-winrt-removed) 所说

由此在使用 WinRT 自带 Windows.Media.Ocr 的第一步是让 WPF 在 dotnet 6 框架上支持 WinRT 开发

在 dotnet 6 下，使用方法特别简单，只需要编辑 csproj 项目文件，在 TargetFramework 上，将原本的 `net6.0-windows` 的后面加上期望使用的 WinRT 的 SDK 版本，例如 19041 版本，更改后的代码如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net6.0-windows10.0.19041.0</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <OutputType>WinExe</OutputType>
  </PropertyGroup>

</Project>
```

对应的 SDK 版本和所支持的系统的关系如 [Call Windows Runtime APIs in desktop apps - Windows apps Microsoft Docs](https://docs.microsoft.com/en-us/windows/apps/desktop/modernize/desktop-to-uwp-enhance) 描述：

- net6.0-windows10.0.17763.0: 对应 Windows 10 1809 版本
- net6.0-windows10.0.18362.0: 对应 Windows 10 1903 版本
- net6.0-windows10.0.19041.0: 对应 Windows 10 2004 版本

开发过 UWP 的伙伴都知道，这里的 SDK 版本指的是目标版本。例如设置为 19041 对应的 Windows 10 2004 版本，而如果期望在更低的 Win10 版本，例如 1809 上运行，可以再指定一个最低版本，加上以下配置

```xml
   <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
```

更改后的 csproj 项目文件的内容如下

```xml
<Project Sdk="Microsoft.NET.Sdk">
 <PropertyGroup>
    <TargetFramework>net6.0-windows10.0.19041.0</TargetFramework>
    <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <OutputType>WinExe</OutputType>
 </PropertyGroup>
</Project>
```

如以上步骤，即可完成对接 WinRT 的支持。其支持的原理也特别有趣，是通过给你引用一个叫 `Microsoft.Windows.SDK.NET.dll` 的 23 MB 以上的 dll 和一个叫 `WinRT.Runtime.dll` 的文件实现的

在 Microsoft.Windows.SDK.NET.dll 实现了对 WinRT 的一层包装，如此可以看到 WinRT 是多么庞大，只是对他的定义的包装，还没有包含实现，就有 23 MB 这么多

完成了对接 WinRT 之后，接下来就可以开始编写图片识别文本的功能了

假定已获取到图片文件 `file` 的文件路径，在对此图片进行转换为文本时，可以分为三步：

第一步解码图片文件

开始解码之前，需要从 `file` 文件路径读取出 IRandomAccessStream 对象，这个 IRandomAccessStream 对象表示的是一个 Stream 且此 Stream 支持随机访问。随机访问是和顺序访问相对，指的是可以从 Stream 的任意地方开始读写

```csharp
        using IRandomAccessStream stream = await FileRandomAccessStream.OpenAsync(file, Windows.Storage.FileAccessMode.Read);
```

读取到了 IRandomAccessStream 对象，即可采用 `Windows.Graphics.Imaging.BitmapDecoder` 进行解码，在解码器里面将会自动识别图片文件是什么编码。如果感觉识别不靠谱，也可以自己传入指定的编码格式

```csharp
        // 解码图片
        var decoder = await Windows.Graphics.Imaging.BitmapDecoder.CreateAsync(stream);
```

自己指定编码是通过传入对应编码的 GUID 值，此 GUID 值对应 WIC 层的定义，也就是说底层也是调用 WIC 层解码。如以下代码，传入指定 png 编码

```csharp
        var decoder = await Windows.Graphics.Imaging.BitmapDecoder.CreateAsync(Windows.Graphics.Imaging.BitmapDecoder.PngDecoderId, stream);
```

解码完成之后即可通过 GetSoftwareBitmapAsync 方法获取 `Windows.Graphics.Imaging.SoftwareBitmap` 对象，这就是一个在 WinRT 里面用来表示图片的一个对象

```csharp
        // 获取图像
        Windows.Graphics.Imaging.SoftwareBitmap? softwareBitmap = await decoder.GetSoftwareBitmapAsync();
```

第二步是判断本机的 OCR 引擎是否支持指定的语言。相当于判断设置里面的语言是否有对应的选项支持

例如已知识别的是中文的图片，可以先通过 `OcrEngine.IsLanguageSupported` 判断是否支持简体中文识别

```csharp
        // 准备识别
        Windows.Globalization.Language lang = new Windows.Globalization.Language("zh-CN");
        // 判断是否支持简体中文识别
        if (Windows.Media.Ocr.OcrEngine.IsLanguageSupported(lang))
        {
            // 如果支持识别才能继续
        }
```

如果本机系统不支持，那就只能告诉用户，你的电脑不支持识别，然后咱的应用就啥都做不了

如果本机系统支持，那就可以开始创建识别引擎

```csharp
            var engine = Windows.Media.Ocr.OcrEngine.TryCreateFromLanguage(lang);
            if (engine != null)
            {
            }
```

以上是尝试创建，其实如果上面的判断通过了，那基本上都是成功的。那为什么不采用一句话判断加创建？原因是创建的代码是比较消耗资源的，判断的代码是比较快速的，完全看自己的业务

第三步就是本文的重点，开始进行识别

```csharp
                OcrResult? result = await engine.RecognizeAsync(softwareBitmap);
```

如此即可完成识别，这个识别速度我测试是非常快的。实际的识别和具体的图片和设备有关

这里拿到的 OcrResult 可以通过 Text 属性获取到识别的文本，如以下代码，将获取的文本放入到一个 TextBox 控件

```csharp
                OcrText.Text = result.Text;
```

如果需要获取对应在图片上的每个字符所在的图片的坐标，可以采用高级的用法，遍历 OcrResult 的 Lines 属性，如以下代码

```csharp
                foreach (OcrLine line in result.Lines)
                {
                    foreach (var word in line.Words)
                    {
                        // 识别的字符所在相对图片的坐标和尺寸
                        Windows.Foundation.Rect bounds = word.BoundingRect;
                        var text = word.Text;
                    }
                }
```

以上就是在 WPF 通过 WinRT 调用系统自带的图片转文本功能，此功能不需要依赖其他第三方，但是依赖用户的系统环境，此功能免费，且使用简单

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/d2959ab2d217d57e9357684d7bf39510e4d9cae8/JagerekukibeHelcewhalay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/d2959ab2d217d57e9357684d7bf39510e4d9cae8/JagerekukibeHelcewhalay) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin d2959ab2d217d57e9357684d7bf39510e4d9cae8
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 JagerekukibeHelcewhalay 文件夹






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。