# gif 格式

现在使用gif的场景有很多，很多老师喜欢在课件添加 gif 图片。

<!--more-->
<!-- CreateTime:2019/8/31 16:55:59 -->

<div id="toc"></div>

在开始讲gif之前，先告诉大家 gif 的格式。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017111014494.jpg)

请看图片，gif 图分为图片文件头（File Header），gif信息（GIF Data Stream）和文件结尾（Trailer）三个部分，最主要的是 gif 信息。gif信息是由控制块（Control Block）和数据块（Data Sub-blocks）组成的。

文件头包括了GIF文件署名(Signature)和版本号(Version)，文件署名就是“gif”字符串，版本号有 87a 和 89a 两个。表示提出的时间，但是现在使用的图片格式有很多，很难说有支持现在全部格式的库。

### gif 信息

gif 信息包括逻辑屏幕标识符(Logical Screen Descriptor)，全局颜色列表(Global Color Table)，图片块

**逻辑屏幕标识符**

逻辑屏幕标识符定义了 gif 图片的逻辑屏幕宽度、逻辑屏幕高度，颜色深度，背景色有无全局颜色列表(Global Color Table)和颜色列表的索引数(Index Count)，请看下表

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017111015233.jpg)

需要知道，图片的位是反过来写的，也就是从屏幕标识符的第5个byte开始，第0-2位表示的是pixel（ 全局颜色列表大小，pixel+1确定颜色列表的索引数（2的pixel+1次方）），第3位是 s 分类标志(Sort Flag)，如果置位表示全局颜色列表分类排列。然后就是 cr ，颜色深度(Color ResoluTion)，cr+1确定图象的颜色深度。m - 全局颜色列表标志(Global Color Table Flag)，当置位时表示有全局颜色列表，pixel值有意义。

**全局颜色列表**

全局颜色列表必须紧跟在逻辑屏幕标识符后面，每个颜色列表索引条目由三个字节组成，按R、G、B的顺序排列。

看到名字可以想到，有全局颜色列表也有局部颜色表，因为一张图像最多只会包含256个RGB值，在一张连续动态GIF里，每一帧之间信息差异不大，颜色是被大量重复使用的。在存储时，我们用一个公共的索引表，把图片中用到的颜色提取出来，这就是颜色列表，所以可以减少存放的数据，因为颜色需要使用 4 个 byte 来放。

假如一个图片使用了3个颜色 x0、x1、x2 ，如果没有使用全局颜色列表，图片长度1000，宽度1000那么每个点都存放颜色，一个颜色需要 4 byte （rbg和透明），存放的空间就为 `1000*1000*4` ，而有颜色表就直接指定颜色表的位置就可以，可以剩下3倍的空间。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017111015158.jpg)


### 图片块

这里就是gif 的数据，可以有很多张图片，图片之间存储连续，图片里面包括控制块和图片数据。

这里的图片叫帧，他的信息包括：

 - 帧分隔符
 - 帧数据说明
 - 点阵数据（它存储的不是颜色值，而是颜色索引）
 - 帧数据扩展(只有89a标准支持）

图片的控制块包括图片的图象标识符、图象的性质，一共需要10字节，请看下面

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171114103751.jpg)

 - m - 局部颜色列表标志(Local Color Table Flag) 置位时标识紧接在图象标识符之后有一个局部颜色列表，供紧跟在它之后的一幅图象使用；值否时使用全局颜色列表，忽略pixel值。	 
 - i - 交织标志(Interlace Flag)，置位时图象数据使用连续方式排列，否则使用顺序排列。	
 - s - 分类标志(Sort Flag)，如果置位表示紧跟着的局部颜色列表分类排列.	
 - r - 保留，必须初始化为0.	
 - pixel - 局部颜色列表大小(Size of Local Color Table)，pixel+1就为颜色列表的位数	



和全局颜色列表不相同的，局部颜色列表需要有 x 方向偏移、y 方向偏移、图象宽度、图象高度

图片块包括图片数据和图形控制扩展。

**图形控制扩展(Graphic Control Extension)**

包括

 - 扩展块标识	Extension Introducer - 标识这是一个扩展块，固定值0x21
 - 图形控制扩展标签	Graphic Control Label - 标识这是一个图形控制扩展块，固定值0xF9
 - 块大小	Block Size - 不包括块终结器，固定值4
 - 处置方法
 - i - 用户输入标志(Use Input Flag)：指出是否期待用户有输入之后才继续进行下去，置位表示期待，值否表示不期待。用户输入可以是按回车键、鼠标点击等，可以和延迟时间一起使用，在设置的延迟时间内用户有输入则马上继续进行，或者没有输入直到延迟时间到达而继续
 - t - 透明颜色标志(Transparent Color Flag)：置位表示使用透明颜色
 - 延迟时间	Delay Time - 单位1/100秒，如果值不为1，表示暂停规定的时间后再继续往下处理数据流
 - 透明色索引	Transparent Color Index - 透明色索引值
 - 块终结器	Block Terminator - 标识块终结，固定值0

处置方法(Disposal Method)：指出处置图形的方法，当值为：
 -  0 - 不使用处置方法
 -  1 - 不处置图形，把图形从当前位置移去
 -  2 - 回复到背景色
 -  3 - 回复到先前状态
 -  4-7 - 自定义

处置方法、i、t 在一个byte，其中第0bit为t，bit1为i，bit2-4处置方法

所有的控制都可以这样跳过，先读byte0，是否是扩展块，固定值0x21，然后读取byte1，可以知道是什么控制。接着就是读取长度byte2，跳过他就可以拿到下一个数据块或控制。如果拿到数据块，那么数据块byte0就是表示数据长度，跳过他就可以拿到下一个数据块或控制。

 - byte0 扩展块
 - byte1 信息
 - byte2 信息长度
 - byte n n的大小为信息长度+2，这是块终结器。

读取到 byte n 下一个就可以重复判断是扩展块还是数据。

** 图片数据 **

图片数据如下

 - 编码长度 LZW Code Size - LZW压缩的编码长度，也就是要压缩的数据的位数
 - ...	数据块开始
 - 块大小	数据块，如果需要可重复多次
 - 编码数据
 - ...	数据块结束
 - 块终结器	 - 一个图象的数据编码结束，固定值0

因为gif使用lzw压缩算法，所以解析gif需要先解析lzw，然后就可以得到图片的数据。

gif 会把相同的图片作为索引，放在lzw，之后相同的数据就使用索引拿到，这样可以减少文件大小。

关于 lzw，请看 http://blog.csdn.net/abcjennifer/article/details/7995426

本文的格式大部分参考 http://www.cnblogs.com/think/archive/2006/04/12/372942.html

关于 gif 解析请看

[wpf 如何使用 Magick.NET 播放 gif 图片](https://lindexi.github.io/lindexi/post/wpf-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-Magick.NET-%E6%92%AD%E6%94%BE-gif-%E5%9B%BE%E7%89%87.html )

[wpf GifBitmapDecoder 解析 gif 格式](https://lindexi.github.io/lindexi/post/wpf-GifBitmapDecoder-%E8%A7%A3%E6%9E%90-gif-%E6%A0%BC%E5%BC%8F.html )

[gif的故事：解剖表情动图的构成](http://www.alloyteam.com/2017/09/13121/ )

一个 gif 解析的方法 https://github.com/vurdalakov/abandoned/tree/master/gifdotnet/src/GifDotNet

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
