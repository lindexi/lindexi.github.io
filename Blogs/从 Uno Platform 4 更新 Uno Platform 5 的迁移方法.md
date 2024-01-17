本文记录我的一个小项目从 Uno Platform 4 更新 Uno Platform 5 的一些变更和迁移方法，由于项目太小，可能踩到的坑不多

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

官方文档： [Migrating to Uno Platform 5.0](https://platform.uno/docs/articles/migrating-to-uno-5.html )

表扬一下官方，文档写的很详细

## Directory.Packages.props

所有的 Uno 相关包的版本更新：

```diff
-    <PackageVersion Include="Uno.WinUI" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.Lottie" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.Lottie" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.RemoteControl" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.RemoteControl" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.Skia.Gtk" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.Skia.Gtk" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.Skia.Linux.FrameBuffer" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.Skia.Linux.FrameBuffer" Version="5.0.143" />
-    <PackageVersion Include="Uno.WinUI.Skia.Wpf" Version="4.10.13" />
+    <PackageVersion Include="Uno.WinUI.Skia.Wpf" Version="5.0.143" />
```

## Skia.WPF

这里有大改的部分，不仅仅只是修改命名空间

之前的 WpfHost 是放入到具体的 WPF 的窗口里面的，现在更新版本是直接放入到 App 级的。迁移步骤如下

- 在 `App.xaml` 文件里面删除 `StartupUri="Wpf/MainWindow.xaml"` 属性的配置
- 删除 `MainWindow.xaml` 和 `MainWindow.xaml.cs` 文件
- 在 `App.xaml.cs` 里添加以下代码到构造函数

```csharp
public App()
{
    var host = new WpfHost(Dispatcher, () => new AppHead());
    host.Run();
}
```

## Skia.Framebuffer

更换了命名空间，从原本的 `Uno.UI.Runtime.Skia.FrameBufferHost` 变更为 `Uno.UI.Runtime.Skia.Linux.FrameBuffer.FrameBufferHost` 命名空间下的类型

只需加上 `using Uno.UI.Runtime.Skia.Linux.FrameBuffer;` 即可

## Skia.Gtk

更换了命名空间，从原本的 `Uno.UI.Runtime.Skia.GtkHost` 变更为 `Uno.UI.Runtime.Skia.Gtk.GtkHost` 命名空间下的类型

只需加上 `using Uno.UI.Runtime.Skia.Gtk;` 即可
