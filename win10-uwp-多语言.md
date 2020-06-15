# win10 uwp 多语言

<!-- 不发布 -->

<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>
<!-- csdn -->

## Multilingual App Toolkit

首先下载工具 Multilingual App Toolkit 可以到 https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkitv40 下载，如果发现他不能下载，可以去[csdn](http://download.csdn.net/detail/lindexi_gd/9726257)我上传的资源下载。

## 使用全球化

安装完成之后，我们修改Package.appxmanifest，输入我们的语言，我们将要翻译的语言，我是写 zh-CN ，每个地区有自己的标识，这个可以在官方寻找你想做的语言，不过不推荐你去写其他语言。

当然，如果是台湾或其他地区的朋友，用自己地区的语言编写是我建议的，这时就需要去查看文档。

我们把所有的需要做多语言的字符串提取出来。假如我们有一个 TextBlock ，他有一个 Text 。我们有一个 TextBox 他有 Header ，我们需要把他们改为多语言。

```xml
                <TextBlock Text="lindexi"></TextBlock>
                <TextBox Header="lindexi.oschina.io"></TextBox>

```

那么我们右击解决方案，新建 Strings 文件夹，在 Strings 新建 zh-CN -> Resources.resw

![](http://image.acmx.xyz/4ee0aeec-688d-462e-9bf6-0c91f74cc0bc20161231185335.jpg)

然后点击 Resources.resw 打开

输入名称 lindexi.Text 输入值 lindexi 。下一行，输入名称 lindexioschinaio.Header 按Tab 输入 lindexi.oschina.io

我们在输入资源需要知道

 1. 不能输入名称有`.`在`.`之后是属性

 2. 我们需要换行，输入 shift+enter 

写完 Resources 我们返回 xaml ，在 TextBlock 和 TextBox 使用Uid

```xml
               <TextBlock x:Uid="lindexi" Text="lindexi"></TextBlock>
               <TextBox x:Uid="lindexioschinaio" Header="lindexi.oschina.io"></TextBox>

```

我们可以留下Text 和 Header ，在使用时会被替换，但我们的 Uid 可以是中文，可能存在项目不让使用中文，于是我们保存 Text 可以知道我们的这个资源是什么

## 使用 Multilingual App Toolkit

写完了我们设置语言，我们右击方案，选择 Multilingual App Toolkit，添加语言

![](http://image.acmx.xyz/7fbd5d52-1ce3-4cd7-a438-1795c3f8517e2016123119328.jpg)

我们选英语和中文 ，参见 https://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/mt607079.aspx

然后我们可以看到多了 .xlf 文件

![](http://image.acmx.xyz/7fbd5d52-1ce3-4cd7-a438-1795c3f8517e2016123119516.jpg)

我们打开它，可以看到我们可以去修改

![](http://ooo.0o0.ooo/2016/12/31/5867912830dad.jpg)

如果不想每个都修改，可以点翻译.翻译所有 这样就好。

如果我们想翻译不要修改，我们可以右击 xlf ，Multilingual App Toolkit 机器翻译。这样就自动翻译，我们可以设置状态，设置了可以在我们添加新的名称或修改，可以让翻译所有不翻译我们设置的。可以让我们设置显示审查或其他的时候知道我们还有哪些翻译没翻译。

我们需要重新生成才生成我们新的 resw 文件，他会包含我们写的翻译。


## 没有自动填充翻译

我试过保存了 xlf，重新生成，resw还是空的，我在堆炸也看到有人问这个问题。

基本是没有一个好的解决的方法，我们可以把空的 resw 删除，右击添加语言，确定，然后重新生成，我试过一次这样是可以。

如果上面方法还是不能使用，那么去把 bin 和debug 文件夹删除，重新生成。

如果还是不能使用，我也不会。

如果大神找到一个方法可以让我们的 Multilingual App Toolkit 一定会修改 resw 请和我说。





<!-- 在属性输入 自定义工具：ReswFileCodeGenerator

如果没有自动生成resw -->

多谢 李继龙 大神

参见：http://www.cnblogs.com/yanxiaodi/p/5091970.html

https://msdn.microsoft.com/zh-cn/library/windows/apps/jj569303.aspx?f=255&MSPPError=-2147217396

http://blog.giovannimodica.com/post/how-to-use-multilingual-app-toolkit-in-universal-apps

但是微软做的多语言太垃圾了，所以我需要使用一个比较好的方法，我现在的项目使用的就是一个很好的方法，尝试把他放在 UWP ，看可不可以。
