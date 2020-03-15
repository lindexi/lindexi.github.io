# Xamarin 和 WPF 的控件和属性的替换

基本上 Xamarin 和 WPF 的技术是相同的，但是有一些小细节和属性不同，本文记录一些不同的点，方便小伙伴将 WPF 项目迁移为 Xamarin 项目

<!--more-->
<!-- CreateTime:2020/2/3 11:15:57 -->

<!-- 发布 -->

需要注意的是 Xamarin 原生支持作为 WPF 控件运行，支持在 WPF 运行，反过来不可以

也就是用 Xamarin.Forms 写的应用能作为 UWP 和 WPF 应用运行，也可以作为 Android 和 IOS 运行，也可以使用 GTK# 在 Linux 下运行。本文只是告诉大家如何从一个已有的 WPF 项目迁移到 Xamarin 上

## 控件

通用控件的属性需要修改

```csharp
Visibility-IsVisible
```

## Panel

通用的容器属性需要修改

```csharp
HorizontalAlignment-HorizontalOptions
VerticalAlignment-VerticalOptions
MinWidth-MinimumWidthRequest
```

## StackPanel

在 WPF 的 StackPanel 需要换 StackLayout 布局


## TextBlock

用 Label 替换

## TextBox

用 Editor 替换

## Button

如果是文本按钮将 Content 替换为 Text 属性

将 Click 事件替换为 Clicked 事件，后台代码替换

```csharp

从

        private void Button_OnClick(object sender, RoutedEventArgs e)
        {

        }

替换为

        private void Button_Clicked(object sender, EventArgs e)
        {

        }
```

[UI 控件比较 - Xamarin](https://docs.microsoft.com/zh-cn/xamarin/cross-platform/desktop/controls/ )

[WPF 与 Xamarin：相似性 & 差异 - Xamarin](https://docs.microsoft.com/zh-cn/xamarin/cross-platform/desktop/controls/wpf )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
