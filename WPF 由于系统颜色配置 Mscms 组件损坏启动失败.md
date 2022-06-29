# WPF 由于系统颜色配置 Mscms 组件损坏启动失败

本文记录 WPF 应用程序，因为系统的颜色配置 Mscms.dll 组件损坏导致应用加载图片失败，从而启动失败的原因和解决方法

<!--more-->

<!-- 博客 -->
<!-- 发布 -->

在 WPF 应用加载图片时，将会调用到系统的 Mscms.dll 组件。刚好我遇到一个用户的设备上，他的 Mscms.dll 是损坏的，在 `C:\Windows\SysWOW64\` 文件夹里面的 Mscms.dll 是 x64 的，于是在 WPF 加载将会抛出如下异常

```
System.Windows.Markup.XamlParseException: 初始化“System.Windows.Media.Imaging.BitmapImage”时引发了异常。
 ---> System.BadImageFormatException: 试图加载格式不正确的程序。 (0x8007000B)
   at MS.Win32.PresentationCore.UnsafeNativeMethods.Mscms.OpenColorProfile(PROFILE& pProfile, UInt32 dwDesiredAccess, UInt32 dwShareMode, UInt32 dwCreationMode)
   at System.Windows.Media.ColorContext.FromRawBytes(Byte[] data, Int32 dataLength, Boolean dontThrowException)
   at System.Windows.Media.ColorContext..ctor(SafeMILHandle colorContextHandle)
   at System.Windows.Media.ColorContext.GetColorContextsHelper(GetColorContextsDelegate getColorContexts)
   at System.Windows.Media.Imaging.BitmapFrameDecode.get_ColorContexts()
   at System.Windows.Media.Imaging.BitmapImage.FinalizeCreation()
   at System.Windows.Media.Imaging.BitmapImage.EndInit()
   at MS.Internal.Xaml.Runtime.ClrObjectRuntime.InitializationGuard(XamlType xamlType, Object obj, Boolean begin)
   --- End of inner exception stack trace ---
   at System.Windows.Markup.XamlReader.RewrapException(Exception e, IXamlLineInfo lineInfo, Uri baseUri)
   at System.Windows.Markup.WpfXamlLoader.Load(XamlReader xamlReader, IXamlObjectWriterFactory writerFactory, Boolean skipJournaledProperties, Object rootObject, XamlObjectWriterSettings settings, Uri baseUri)
   at System.Windows.Markup.WpfXamlLoader.LoadBaml(XamlReader xamlReader, Boolean skipJournaledProperties, Object rootObject, XamlAccessLevel accessLevel, Uri baseUri)
   at System.Windows.Markup.XamlReader.LoadBaml(Stream stream, ParserContext parserContext, Object parent, Boolean closeStream)
   at System.Windows.Application.LoadComponent(Object component, Uri resourceLocator)
   at Lindexi.Demo.MainWindow.InitializeComponent()
   at Lindexi.Demo.MainWindow..ctor()
```

修复方法是去下载对应系统版本的 Mscms.dll 进行替换。我从 [https://www.dll-files.com/mscms.dll.html](https://www.dll-files.com/mscms.dll.html) 里找到对应的版本，也就是 Win10 对应 Win10 的，而 Win7 对应 Win7 的，对 32 位的对应 32 位，对 64 位的对应 64 位，下载解压缩，放在对应的文件夹里面。如 32 位的放在 `C:\Windows\SysWOW64\` 文件夹，如 64 位的放在 `C:\Windows\System32\` 文件夹里面，替换原有文件即可

另外的可能由于 Mscms.dll 颜色配置导致起不来的，是在系统的颜色配置里面设置很诡异，解决方法是还原到默认。设置的入口还请自行搜 `配置显示器颜色配置文件` 的方法
