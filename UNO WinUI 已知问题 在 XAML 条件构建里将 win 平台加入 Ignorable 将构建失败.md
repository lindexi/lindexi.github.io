# UNO WinUI 已知问题 在 XAML 条件构建里将 win 平台加入 Ignorable 将构建失败

如果在 UNO 项目里面，为了进行 XAML 条件构建，将 win 平台加入到 mc:Ignorable 里面，将会在构建时提示 Xaml Internal Error error WMC9999: Unexpected 'NONE' in parse rule 'Element ::= . EmptyElement ( StartElement ElementBody ).'. 错误

<!--more-->
<!-- CreateTime:2024/3/3 11:58:03 -->

<!-- 发布 -->
<!-- 博客 -->

如以下的代码，将会在构建时失败，提示 `Xaml Internal Error error WMC9999: Unexpected 'NONE' in parse rule 'Element ::= . EmptyElement | ( StartElement ElementBody ).'.` 错误

```xml
<Page x:Class="KernarjeheeboLawbeeferedai.MainPage"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
      xmlns:local="using:KernarjeheeboLawbeeferedai"
      xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
      xmlns:win="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:not_win="http://uno.ui/not_win"
      Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"
      mc:Ignorable="d win not_win">
  <StackPanel
        HorizontalAlignment="Center"
        VerticalAlignment="Center">
    <win:TextBlock AutomationProperties.AutomationId="HelloTextBlock"
                  Text="Hello Uno Platform"
                  HorizontalAlignment="Center" />
    <not_win:TextBlock AutomationProperties.AutomationId="HelloTextBlock"
                       Text="Hello"
                       HorizontalAlignment="Center" />
  </StackPanel>
</Page>
```

以上的失败仅仅只会在 WinUI 3 平台构建失败，换句话说就是即使新建一个 WinUI 3 项目，直接抄以上的代码也是会出现完全相同的错误信息

失败核心代码在于 `mc:Ignorable="d win not_win"` 将 win 平台加入到忽略列表里面。本质原因是 win 平台指向的 `http://schemas.microsoft.com/winfx/2006/xaml/presentation` 刚好就是当前的页面的默认命名空间

换句话说就是 `xmlns:win="http://schemas.microsoft.com/winfx/2006/xaml/presentation"` 和 `xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"` 默认命名空间是相同的，这就导致了 Root 元素，即以上代码的 `<Page` 被忽略掉。因为 `<Page` 的命名空间就是 `http://schemas.microsoft.com/winfx/2006/xaml/presentation` 刚好就和加入到 mc:Ignorable 的 win 是相同的

根据 XAML 的规定，加入到 mc:Ignorable 为忽略列表，这也就是 d 设计时可用的原因。加入到 mc:Ignorable 忽略列表里面的元素可以被其他逻辑进行特殊处理，比如说 UNO 的 XAML 处理模块将会处理这些内容作为 XAML 条件构建

在 UNO 的[官方文档](https://platform.uno/docs/articles/platform-specific-xaml.html) 里面也有这样一段话，说明了不应该将 win 加入到 Ignorable 里面，内容如下

> For prefixes which will be excluded on Windows (e.g. android, ios), the actual namespace is arbitrary, since the Uno parser ignores it. The prefix should be put in the mc:Ignorable list. For prefixes which will be included on Windows (e.g. win, not_android) the namespace should be http://schemas.microsoft.com/winfx/2006/xaml/presentation and the prefix should not be put in the mc:Ignorable list.

根据上文所述的 XAML 规范可知，除了 win 前缀之外，其他的任何使用了 `http://schemas.microsoft.com/winfx/2006/xaml/presentation` 命名空间的前缀，都不能加入到 `mc:Ignorable` 里面，比如 `win` `not_android` `not_ios` `not_wasm` `not_macos` 和 `not_skia` 前缀。更多请看 UNO 的[官方文档](https://platform.uno/docs/articles/platform-specific-xaml.html) 里面的列表

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/eee7d7898c92da0bffd1cba4fdacaa64b3f79c01/KernarjeheeboLawbeeferedai) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/eee7d7898c92da0bffd1cba4fdacaa64b3f79c01/KernarjeheeboLawbeeferedai) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin eee7d7898c92da0bffd1cba4fdacaa64b3f79c01
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin eee7d7898c92da0bffd1cba4fdacaa64b3f79c01
```

获取代码之后，进入 KernarjeheeboLawbeeferedai 文件夹，即可获取到源代码

更多 UNO 和 WinUI 开发，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

如有 UNO 开发过程相关问题，欢迎加入 724181515 QQ群讨论

如有国产系统开发的相关问题，欢迎加入 810052083 QQ群讨论