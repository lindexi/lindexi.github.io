# WPF 播放 gif

本文告诉大家如何在 WPF 播放 Gif 图片，提供了几个方法进行播放，包括比较性能。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->
<!-- 标签：WPF，gif -->
<div id="toc"></div>

## MediaElement 方法

这是比较不推荐的方法，但是使用简单

```csharp
<MediaElement x:Name="gifMedia"  Source="x.gif" UnloadedBehavior="Manual"  LoadedBehavior="Play" />
```

参见：[WPF使用MediaElement显示gif图片 - CSDN博客](http://blog.csdn.net/SANYUNI/article/details/73608771 )

## Magick 方法

这个方法请参见博客 [WPF 一个性能比较好的 gif 解析库 ](https://lindexi.gitee.io/post/WPF-%E4%B8%80%E4%B8%AA%E6%80%A7%E8%83%BD%E6%AF%94%E8%BE%83%E5%A5%BD%E7%9A%84-gif-%E8%A7%A3%E6%9E%90%E5%BA%93.html )

## WinForm 的方法

### 使用

本文提供的类，可以直接在 Xaml 使用或者在 cs 使用，可以控制开始播放和停止。

在播放的过程中，使用很少的内存。在使用到一定的时间，会自动释放内存。而且比我现在项目使用的播放的 CPU 要少很多，我自己写的 gif 播放需要使用 3% 左右的 CPU，下面这个类使用的 CPU 只有 1% 。当然我的 gif 解析使用的内存会比下面的代码少，不然我就不敢把下面的代码开源了

在 xaml 使用的方法：

```csharp
            <local:GifImageControl x:Name="Image" Path="2017年3月23日 115958.gif"></local:GifImageControl>

```

在添加进之后就会自动开始播放

如果需要在后台代码添加，那么可以使用下面代码

```csharp
        <Grid x:Name="HlosqrrsDnqxv">
        </Grid>

            var image = new GifImageControl("2017年3月23日 115958.gif");
            HlosqrrsDnqxv.Children.Add(image);
```

因为代码很简单，所以需要其他的功能，请看源代码

## 源码

代码放在 github ，可以直接复制这个类到工程使用。下面代码可以用在正式项目中。

<script src="https://gist.github.com/lindexi/7c6d70c821fcb72f487812e58c564442.js"></script>

项目下载：[WPF 使用 WinForm 播放 gif](http://download.csdn.net/download/lindexi_gd/10249202 )

如果在运行项目出现 异常，那么请把 DeleteObject 方法修改为下面的代码

```csharp
        [DllImport("gdi32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        [return: MarshalAs(UnmanagedType.Bool)]
        private static extern bool DeleteObject(IntPtr hObject);
```

打开 .sln 文件，然后按 F5 运行，可以看到占用内存在 120M ，在运行一定时间，回收内存，占用内存70M，而CPU几乎都不需要。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018%25E5%25B9%25B42%25E6%259C%258810%25E6%2597%25A5%2520151339.gif)

参见：http://hi.baidu.com/mych/blog/item/1eb14f545f12a752564e00be.html

[WPF播放GIF控件完整代码 - CSDN博客](http://blog.csdn.net/Libby1984/article/details/52535085 )

如果在运行出现任何问题，请告诉我，上面这个方法不保证可以解决任何的gif图片。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
