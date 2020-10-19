
# WPF 手绘对称图形控件

本文来安利大家一个我刚做的控件，这个控件支持触摸下多指进行笔迹绘制，绘制过程中将会对称显示出水平和垂直翻转的笔迹。可以用来绘制对称图形。当然了，本文还会告诉大家这个控件是如何制作的

<!--more-->


<!-- 发布 -->

啥都不用说，先来一张图

<!-- ![](image/WPF 手绘对称图形控件/WPF 手绘对称图形控件0.gif) -->

![](http://image.acmx.xyz/lindexi%2F%25E5%25AF%25B9%25E7%25A7%25B0%25E5%259B%25BE%25E5%25BD%25A2.gif)

我将这个控件在GitHub上完全开源，代码放在 [https://github.com/lindexi/uwp](https://github.com/lindexi/uwp)

虽然这是放在 UWP 仓库的控件，但现在只支持 WPF 框架

下面来告诉大家如何使用这个控件

在 dotnet 里面的使用控件的套路都是第一步安装 NuGet 第二步引用命名空间，第三步使用控件

## 安装 NuGet 库

在 NuGet 控制台输入下面命令就可以安装这个库

```
Install-Package Lindexi.Control.WPFFlipDrawingCanvas -Version 1.0.0
```

如果是 SDK 风格的 csproj 文件格式，可以在此文件添加下面代码安装

```xml
    <ItemGroup>
        <PackageReference Include="Lindexi.Control.WPFFlipDrawingCanvas" Version="1.0.0" />
    </ItemGroup>
```

## 使用控件

先在 XAML 中添加下面代码，添加命名空间

```xml
        xmlns:wpfFlipDrawingCanvas="clr-namespace:Lindexi.Control.WPFFlipDrawingCanvas;assembly=WPFFlipDrawingCanvas"
```

接着就可以使用这个控件

```xml
            <wpfFlipDrawingCanvas:FlipDrawingCanvas x:Name="FlipDrawingCanvas"></wpfFlipDrawingCanvas:FlipDrawingCanvas>
```

现在尝试运行代码就可以看到界面上有一个可以手绘的控件

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/188f8cfccd709841878262e42e96f33a302bdfdf/GayallhakereKuherferelegi ) 欢迎小伙伴访问

## 制作方法

那么这个控件是如何制作的，在 WPF 里面如何进行多指的书写？在 WPF 中如何让控件进行水平和垂直的翻转？ 了解了这两个问题，就可以很简单制作这个控件

我有一篇博客，告诉大家如何在 WPF 中使用简单的代码制作一个支持多指笔迹的控件，请看 [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html )

在 WPF 中实现翻转特别简单，只需要让缩放作为负数就可以了，如下面代码

```csharp
<Button Content="欢迎访问我博客 https://blog.lindexi.com 里面有大量 UWP WPF 博客" Padding="5">
  <Button.RenderTransform>
    <ScaleTransform ScaleX="-1" />
  </Button.RenderTransform>
</Button>
```

<!-- ![](image/WPF 手绘对称图形控件/WPF 手绘对称图形控件0.png) -->

![](http://image.acmx.xyz/lindexi%2F20201017161531776.jpg)

详细请看 [How to: Flip a UIElement Horizontally or Vertically - WPF .NET Framework](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/advanced/how-to-flip-a-uielement-horizontally-or-vertically?view=netframeworkdesktop-4.8&WT.mc_id=DX-MVP-5003606 )

我这个控件很简单，因此去安装一个 NuGet 有点坑，所以我更推荐你去抄抄我的代码哈

如果发现看不懂我的代码，证明还没入门，欢迎加入入门级的群： 874752819  进群之后和群主说是德熙推荐过来的，群主就会看心情给你退回进群费

想要更进阶的交流，还请加入进群费更贵的 858784803 群，当然了进群之后告诉群主说你是我推荐过来的，也许群主想不开会给你发一个红包，或者给我发一个红包

以上两个群的进群费和我没一毛钱关系……





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。