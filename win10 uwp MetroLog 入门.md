# win10 uwp MetroLog 入门

在实际的项目，需要做很多记录，记录日志可以作为调试。在 UWP 如果自己写日志，放在文件，那么需要很多重复代码。

在 UWP 写文件是很慢，而且需要异步，所以很多小伙伴不喜欢写日志。

在以前可以使用 log4net ，一个很好用的日志。在 UWP 没有支持，但是有另一个日志，也是很多大神在使用的 MetroLog 。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn --> 

## 安装

可以通过 Nuget 的方式安装

右击解决方法，选择 Nuget 搜索 MetroLog 安装第一个

![](http://image.acmx.xyz/lindexi%2F2018572033225407.jpg)

最近有大神和我说 Log4Net 支持 UWP ，现在我先告诉大家如何用 MetroLog ，我尝试用这两个，还是 MetroLog 简单。

## 使用

使用的方法很简单，首先需要告诉日志，当前使用的是哪个日志。为什么需要告诉他使用的是哪个日志？

因为一个软件有很多模块，如我有一个是网络通信，那么如果写的和计算模块相同日志，那么就很难知道哪里是计算模块写的。

因为只是告诉大家如何使用，就不分模块，使用 逗比 日志。

```csharp
            var yehaserebuBodojair = MetroLog.LogManagerFactory.CreateLogManager().GetLogger("逗比");
```

记录的等级有很多个，按照重要从小到重要排列

 - Trace 记录，这个等级最不重要，什么东西都可以记
 - Debug 调试，只有在调试才使用
 - Info 信息，写入或不写入都不重要
 - Warn 警告，程序出现了诡异
 - Error 错误，这个信息重要
 - Fatal 失败，软件崩溃，主要信息

那么如何记录信息，刚才拿到 yehaserebuBodojair 就可以用来写入信息

直接调用 `yehaserebuBodojair.Error` 就是可以写入信息

```csharp
            yehaserebuBodojair.Error("点击确定");

```

因为默认的配置是 Error 和以上就写入文件，所以这时可以去看文件

在界面添加一个按钮，在按钮点击添加代码

```csharp
       private void PassairjirqaPeazoo_OnClick(object sender, RoutedEventArgs e)
        {
            var yehaserebuBodojair = MetroLog.LogManagerFactory.CreateLogManager().GetLogger("逗比");

            yehaserebuBodojair.Error("点击确定");
        }
```

点击一下按钮可以看到输出显示

```csharp
3|2018-05-07T12:58:45.5958738+00:00|ERROR|3|逗比|点击确定
```

## 应用本地缓存

拿到应用本地的数据的方式很简单

双击打开 Package.appxmanifest 文件

![](http://image.acmx.xyz/lindexi%2F2018572054392144.jpg)

点击打包可以看到包系列名

然后从资源管理器打开 `%appdata%` 打开里面 的 `Local\Packages` 找到应用的包系列名。

打开 `LocalState\MetroLogs` 就可以看到日志

![](http://image.acmx.xyz/lindexi%2F20185720574241.jpg)

大概的路径是

```csharp
C:\Users\lindexi\AppData\Local\Packages\0384ceff-e9d9-49eb-b1a4-9bba2a6d6a40_rdbbrz3qfe7gm\LocalState\MetroLogs
```

打开文件可以看到日志

![](http://image.acmx.xyz/lindexi%2F20185720596462.jpg)

但是我会告诉大家这么难的方法？实际上使用一句代码就可以打开所在文件

修改刚才按钮点击，添加代码

```csharp
            var wadairfikeeRaycirralljair = Launcher.LaunchFolderAsync(ApplicationData.Current.LocalFolder);
```

这个代码就是打开应用所在的文件，可以快速打开文件不需要去找。

[Building a Universal Windows Platform (UWP) Application (Part 4) – Logging w/ MetroLog - Intertech Blog](https://www.intertech.com/Blog/building-a-universal-windows-platform-uwp-application-part-4-logging-w-metrolog/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
