# win10 uwp 如何开始写 uwp 程序

本文告诉大家如何创建一个 UWP 程序。

这是一系列的 uwp 入门博客，所以写的很简单

<!--more-->
<!-- CreateTime:2019/11/29 10:12:42 -->

<div id="toc"></div>
本文来告诉大家如何创建一个简单的程序

## 安装 VisualStudio 

在开始写 UWP 需要安装 VisualStudio ，安装需要从官网下载程序，或者使用网上大神做的离线安装。

但是 VisualStudio 大概一个月更新一次，所以不建议使用离线安装。

在睡觉前可以从 [官网](https://www.visualstudio.com/zh-hans/ ) 下载 VisualStudio IDE ，提供有几个版本，大概下载 Community 版本就可以了。这是一个免费的版本。

打开安装界面，下面的版本是企业版，如果想使用企业版但是没有秘钥可以找我。

然后选择 通用 windows 开发，其他的选项可以选也可以忽略。

![](http://image.acmx.xyz/lindexi%2F20184141612579935.jpg)

注意不要去修改默认安装的，因为 VisualStudio 必须安装在 C 盘，如果安装在其他的盘可能无法运行。

## 创建应用

在安装之后，大概是第二天，点击文件新建就可以看到这个界面

![](http://image.acmx.xyz/lindexi%2F20184141615352914.jpg)

点击空白项目就可以创建，记得设置软件名和解决方案名，存放的位置。

点击确定就可以看到下面的界面

![](http://image.acmx.xyz/lindexi%2F20184141616397927.jpg)

需要注意，目标版本就是当前编译面对的版本，但是最低版本指的是需要支持的版本。

如果选择的目标版本越高，理论可以使用的接口越多，如果选择的最低版本很低，那么就必须使用越低的接口。

![](http://image.acmx.xyz/lindexi%2F2018414162014647.jpg)

所以一般我都不会选很低的版本，需要说的是，如果想要支持亚克力，那么就需要选择最低版本 16299 ，因为现在是入门，所以建议选择最高版本。

## 启动流程

很多小伙伴不知道启动流程，实际上在开发的时候就需要使用一个启动流程。

![](http://image.acmx.xyz/lindexi%2F2018414162242577.jpg)

在开始启动的时候加载的是一张静态的图片，在用户点击应用的时候就可以显示。然后立刻跳转到动态的图片，一般都是做一个动画。这样用户就可以看到软件正在运行，不会觉得软件启动很慢。

这时就进行后台读取应用数据，一般的软件在启动的时候都需要读取配置，读取文件的方法请看[win10 UWP读写文件](https://blog.csdn.net/lindexi_gd/article/details/49007841 )。

在读取完成配置之后一般还做模块初始化，注入。

关于启动流程的，因为不属于入门的博客，所以我会在别的博客说。

在读取配置之后，初始化了模块就可以跳到主页面，这时就可以在主页面显示内容。

## 显示 hellow

大概所有的入门博客都会告诉大家如何写一个 hellow ，我这里也是这样告诉大家。

打开 MainPage.xaml ，一般使用双击文件的方法

![](http://image.acmx.xyz/lindexi%2F2018414162710847.jpg)

然后可以看到这个界面

![](http://image.acmx.xyz/lindexi%2F20184141627479596.jpg)

在下面的代码，估计需要滚动才可以看到，因为我默认设置是打开代码而不是设计，如果安装默认的 VisualStudio 显示的是一半设计，下面一半是代码，所以需要滚动代码才可以看到下面代码。

```csharp
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">

    </Grid>
```

我不会告诉大家每个控件的意思，我建议快速复制粘贴代码，尝试运行，对于代码的意思可以后面慢慢看。

添加文本 hellow ，添加文本的方法是使用 TextBlock ，至于对于 Text 文本之外的其他属性，我就不多说了。这些属性大家可以使用控制面板的属性窗口看到很多设置，去试试控件的设置然后运行看效果。

```csharp
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <TextBlock Text="林德熙逗比" HorizontalAlignment="Center" VerticalAlignment="Center"></TextBlock>
    </Grid>
```

然后按 F5 运行，大概就可以看到界面有文字。

尝试修改文字为你自己想要的，然后运行。

实际上桌面开发的入门难度相对比较大，因为需要学很多东西。如果学比较深，那么需要的时间是比较长的。在 UWP 开发，建议界面使用 xaml 来写，什么是 xaml ，就是文件后缀是 xaml 的文件。

大概需要学 C# 的知识，现在已经到了 C# 7 ，所以新的功能是需要学。

然后需要学习 dot net core 或 dot net Framework 的知识，学习只需要了解里面有哪些类，如何使用他。

实际上 dot net core 就已经需要学很久了，但是开发时还需要学 VisualStudio 和一些插件的使用。我看到有些小伙伴写代码的速度实在太慢，还有调试的能力也很差，因为他不会使用 VisualStudio ，如果学会使用，那么开发速度很快很多。除了 VisualStudio 还需要去了解一些常用的工具，请看[高效率工具](https://lindexi.gitee.io/post/%E9%AB%98%E6%95%88%E7%8E%87%E5%B7%A5%E5%85%B7.html )

另外软件开发还有基础的部分，关于数据结构、基础算法、设计模式都需要去学一下。但是必须告诉大家的是，关于数据结构千万要学 dot net 提供的数据结构而不是对于每个数据结构都去自己写，如果自己写轮子在 dot net 开发大神看来这就是新手。尽量使用 dot net 提供的算法才是一个高手需要做的。

更多的 UWP 博客请直接看我的博客，csdn博客：https://blog.csdn.net/lindexi_gd 个人博客： https://lindexi.gitee.io 在国内我是写 UWP 博客最多的人，但是文章水平比较低，如果发现我博客写的不好懂，请告诉我

如果遇到任何的问题，欢迎加入一些 dot net 交流的群交流。

欢迎大家加入我的组织 [telegrma](https://t.me/dotnet_campus) ，如果发现无法加入，请看[如何使用 Telegram](https://blog.lindexi.com/post/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-Telegram.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
