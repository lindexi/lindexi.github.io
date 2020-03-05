# 断点调试 Windows 源代码

本文来告诉大家如何使用断点的方式，在 PotPeek 工具的反编译看到报告的异常。

这个方法对于 DUMP 调试比较有用，可以知道是在哪个函数哪一句抛出的异常。

<!--more-->
<!-- CreateTime:2018/9/20 17:37:55 -->

<!-- csdn -->
<!-- 标签：VisualStudio，调试 -->

首先先让软件出现一个异常，那么最简单的异常就是跨线程访问主线程

在 页面写一个依赖属性，然后在另一个线程设置这个属性

```csharp
        public MainWindow()
        {
            InitializeComponent();

            new Thread(() =>
            {
                TowbiboujeJerexakipa = "123";

            }).Start();
        }

        public static readonly DependencyProperty TowbiboujeJerexakipaProperty = DependencyProperty.Register(
            "TowbiboujeJerexakipa", typeof(string), typeof(MainWindow), new PropertyMetadata(default(string)));

        public string TowbiboujeJerexakipa
        {
            get { return (string) GetValue(TowbiboujeJerexakipaProperty); }
            set { SetValue(TowbiboujeJerexakipaProperty, value); }
        }
```

按一下运行就可以看到下面的异常，但是大家想知道这个异常是在底层哪里抛出的？


![](http://image.acmx.xyz/lindexi%2F20185112020404476.jpg)

如果这时在我的设备，可以看到调用堆栈，我双击一下就可以进去代码

![断点调试 Windows 源代码 调试.gif](https://i.loli.net/2018/05/11/5af58c7027a2f.gif)

想知道我是怎么做的就继续看博客，不想知道的就点关闭页面

## 下载微软代码

打开 https://referencesource.microsoft.com/ 可以看到 Download 点击进入就可以看到这个界面

![](http://image.acmx.xyz/lindexi%2F20185112029396770.jpg)

自己需要什么版本就下载什么版本，不过我是把所有的版本都下载，然后创建 git 管理，这样需要哪个版本就可以快速切换

## 调试文件

但是下载好的文件，VisualStudio 怎么知道他需要从哪里找？虽然 VisualStudio 可以添加人工智能告诉你怎么写代码才有坑，但是他也不知道你下载的文件会放在哪。

通过右击解决方案，点击属性，可以看到可以设置源代码

![](http://image.acmx.xyz/lindexi%2F20185112031382786.jpg)

我就把我下载的源代码解压到 God 文件夹的 source 文件夹，也就是图片最后一行，我就这样添加了代码。

这时 VisualStudio 就会尝试去这里寻找源代码。不过不要想着这样就可以像我一样快速调试源代码，还需要另一个工具

## DotPeek

打开 DotPeek ，如果还没下载，那么请到[官网](https://www.jetbrains.com/decompiler/ )下载

或者到我上传的 CSDN 下载 [dotPeek 查看源代码-CSDN下载](https://download.csdn.net/download/lindexi_gd/10133189 )

在我的[调试 ms 源代码](https://lindexi.github.io/lindexi/post/%E8%B0%83%E8%AF%95-ms-%E6%BA%90%E4%BB%A3%E7%A0%81.html ) 已经有告诉大家如何调试 ms 源代码，但是没有告诉大家如何做到断点调试，直接看到微软源代码是哪里异常

### 符号服务器

打开 dotpeek 符号服务器，打开和使用方法请看[调试 ms 源代码](https://lindexi.github.io/lindexi/post/%E8%B0%83%E8%AF%95-ms-%E6%BA%90%E4%BB%A3%E7%A0%81.html )

### 加载代码

如果发现 dotpeek 的加载的代码太少了，如没有找到 WindowsBase 就需要在 GAC 添加代码

![](http://image.acmx.xyz/lindexi%2F20185112037396069.jpg)

点击 OpenFromGAC 就可以在 GAC 寻找代码，如果需要调试 WPF 代码，那么 WindowsBase 是必须添加的

![](http://image.acmx.xyz/lindexi%2F20185112038533331.jpg)

## 加载符号

这时很多小伙伴都关闭了符号加载，因为打开符号加载，开始调试需要等很久，如果大家有缓存自己的符号，那么调试加载符号还是很快的。

如果开始没有加载符号，在调试就需要打开点击工具 调试-> 窗口->模块

![](http://image.acmx.xyz/lindexi%2F2018511204032550.jpg)

如果看到自己没有加载符号，就需要右击这个模块，点击加载符号

![](http://image.acmx.xyz/lindexi%2F20185112041295487.jpg)

这时点击了就可以去喝咖啡，因为需要等待 dotpeek 创建符号，打开 dotpeek 可以看到进度，是非常慢的

除了WindowsBase还需要添加 PresentationCore ，如果自己的 PotPeek 没有，请自己添加

下面是需要检查的代码

 - mscorlib

 - System

 - System.Core

 - System.Xml

 - System.Xaml

 - WindowsBase

 - PresentationCore

 - PresentationFramework

## 找到符号

有时候发现虽然已经按照我说的做了，还是无法进入堆栈代码，而且弹出了这个选项，让你去找文件

![](http://image.acmx.xyz/lindexi%2F20185112042458107.jpg)

这时就需要使用 dotpeek 找到对应的代码，然后导出工程

在对应的代码右击，选择导出工程

如果自己有导出，那么把这个工程添加到设置源代码

![](http://image.acmx.xyz/lindexi%2F20185112031382786.jpg)

## 断点调试

如果已经看到了这里，请不要说我写了这么久还没说到主题，刚才只是准备，现在开始就是断点调试

点击调试->新建断点，例如下面需要调试 `PenContext.InitStylusPointDescription` 函数，就需要在断点写入 类名.函数名

![](http://image.acmx.xyz/lindexi%2F20185112049637.jpg)

这个函数在第一次触摸时触发，所以使用这个断点就可以在第一次触摸进去

运行程序，如果在断点可以看到一个红色的点，表示这个断点是可以进去

![](http://image.acmx.xyz/lindexi%2F20185112052133574.jpg)

如果看到一个黑点，表示这个断点无法进入，这时尝试加载符号，需要注意，必须要使用 dotPeek 符号服务器才可以加载，这时需要开着 dotpeek 只要加载第一次符号，之后加载还是很快

这时试试触摸一下，就可以看到跳转到这个函数

![](http://image.acmx.xyz/lindexi%2F20185112054106370.jpg)

下面就可以高兴对这个类的函数进行断点，但是不是全部语句都可以添加断点，因为调试的源代码是 Release 会优化很多代码。

## 简单的调试

如果这时为了调试简单的代码，还可以使用 dnspy 调试，使用这个调试十分快。

首先下载 [dnSpy](https://github.com/0xd4d/dnSpy )，这个软件需要区分 x86 和 x64 选择自己需要调试的程序的平台，运行。

首先拖入已经编译好的 exe 到 dnspy

![](http://image.acmx.xyz/lindexi%2F20185112058431625.jpg)

然后点击拖入的 exe 点击运行就可以调试这个 exe 了，但是这里是告诉大家如何调试源代码，首先寻找到需要调试的代码

![](http://image.acmx.xyz/lindexi%2F201851121163535.jpg)

还是告诉大家如何调试 PenContext.InitStylusPointDescription ，因为我知道 PenContext 在哪，于是我就一级级展开，找到 InitStylusPointDescription 函数，在这个函数按下 F9 没错，他的快捷键和 VisualStudio 差不多，这时按下 F5 调试程序

![](http://image.acmx.xyz/lindexi%2F201851121312292.jpg)

在使用这个软件还可以在自己的代码断点，然后按 F11 进入微软框架代码查看他是怎么做的。

虽然我告诉了大家这些方法用来断点调试，但是我无法说大家一定可以使用我的方法看到源代码，有一些源代码是无法拿到的，有一些是没有符号。

看到这里大家是否好奇为什么我在调试 InitStylusPointDescription ？ 因为我的 WPF 在一个特殊的屏幕点击就会崩溃，我拿到了 Dump ，看到了托管异常

![](http://image.acmx.xyz/lindexi%2F2018511212473343.jpg)

我使用了 dnspy 定位了堆栈，然后远程调试，加载了符号，进入源代码查看了这个函数

![](http://image.acmx.xyz/lindexi%2F20185112126196916.jpg)

我发现是在这一行代码崩溃的

```csharp
      StylusPointPropertyInfo pointPropertyInfo = new StylusPointPropertyInfo(new StylusPointProperty(guid, false), iMin, iMax, (StylusPointPropertyUnit) iUnits, flResolution);
```

我进入了 StylusPointPropertyInfo 构造函数

![](http://image.acmx.xyz/lindexi%2F20185112127405999.jpg)

很快就看到抛异常的代码

```csharp
      if (maximum < minimum)
        throw new ArgumentException(MS.Internal.PresentationCore.SR.Get("Stylus_InvalidMax"), nameof (maximum));
```

这里 maximum 是 0xFFFFFFFF 因为这里是 int 判断，最大值 -1 所以返回 false ，在 usb 论坛找到工具

发现是插入屏幕描述符是错的，所以就让硬件去修改。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
