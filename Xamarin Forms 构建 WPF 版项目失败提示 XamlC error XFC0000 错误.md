# Xamarin Forms 构建 WPF 版项目失败提示 XamlC error XFC0000 错误

在 Xamarin Forms 从 4.7 到 4.8.0.1560 版本，在 Xamarin Forms 构建 WPF 版项目也许会提示 XamlC error XFC0000 : Cannot resolve type "Application" 构建失败

<!--more-->
<!-- CreateTime:2020/10/22 9:10:35 -->

<!-- 发布 -->

提示内容如下

```csharp
App.xaml : error :  : XamlC error XFC0000 : Cannot resolve type "Application".
AppShell.xaml : error :  : XamlC error XFC0000 : Cannot resolve type "Shell".
Views/AboutPage.xaml : error :  : XamlC error XFC0000 : Cannot resolve type "ContentPage".
Views/ItemDetailPage.xaml : error :  : XamlC error XFC0000 : Cannot resolve type "ContentPage".
Views/ItemsPage.xaml : error :  : XamlC error XFC0000 : Cannot resolve type "ContentPage".
Views/NewItemPage.xaml : error :  : XamlC error XFC0000 : Cannot resolve type "ContentPage".
```

<!-- ![](image/Xamarin Forms 构建 WPF 版项目失败提示 XamlC error XFC0000 错误/Xamarin Forms 构建 WPF 版项目失败提示 XamlC error XFC0000 错误0.png) -->

![](http://image.acmx.xyz/lindexi%2F2020102292074124.jpg)

如 [[Bug] XAML compiler broke with 4.7.0 · Issue #11101 · xamarin/Xamarin.Forms](https://github.com/xamarin/Xamarin.Forms/issues/11101#issuecomment-713626002 ) 所说，一个可以使用的方法是 [Michael Schnerring](https://github.com/schnerring) 大佬提供的方法

在 csproj 中添加如下代码

```xml
<ItemGroup>
    <EmbeddedResource Remove="**/*.xaml" />
</ItemGroup>
```

详细请看 [https://github.com/xamarin/Xamarin.Forms/issues/11101#issuecomment-678329339](https://github.com/xamarin/Xamarin.Forms/issues/11101#issuecomment-678329339)

另一个方法是我的方法，删除所有的 App.xaml 和 MainWindow.xaml 等文件，如 [https://github.com/lindexi/lindexi_gd/tree/96c9063fdba9fe318eb099da67422de5cc9ae5af/XamarinNeller/XamarinNeller.WPF](https://github.com/lindexi/lindexi_gd/tree/96c9063fdba9fe318eb099da67422de5cc9ae5af/XamarinNeller/XamarinNeller.WPF) 项目所示，这样也能构建成功。细节请看 [WPF 从零手动创建承载 Xamarin Forms 项目](https://blog.lindexi.com/post/WPF-%E4%BB%8E%E9%9B%B6%E6%89%8B%E5%8A%A8%E5%88%9B%E5%BB%BA%E6%89%BF%E8%BD%BD-Xamarin-Forms-%E9%A1%B9%E7%9B%AE.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
