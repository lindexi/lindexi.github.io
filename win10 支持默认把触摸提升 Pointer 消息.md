# win10 支持默认把触摸提升 Pointer 消息

在 WPF 经常需要重写一套触摸事件，没有UWP的Pointer那么好用。

如果一直都觉得 WPF 的触摸做的不好，或想解决 WPF 的触摸问题，但是没有方法，那么请看下面。

<!--more-->
<!-- CreateTime:2019/11/9 15:32:31 -->

<!-- 标签：WPF，触摸 -->

只要新建框架为 .net 4.7 以上，运行的系统是 `Windows 10 Creators Update` 1703 10.0.15063 就可以。

打开新建的工程，设置框架。

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017417165611.jpg)

然后打开 App.config，添加支持把触摸和笔到鼠标


```csharp
     <runtime>
        <AppContextSwitchOverrides value="Switch.System.Windows.Input.Stylus.EnablePointerSupport=true"/>
    </runtime>
```

需要知道，这个特性不支持实时的笔迹。

因为笔迹需要运行在UI线程，会导致比较差的性能。

开启了这个属性就可以使用 Pointer 消息。

因为有小伙伴说使用了我上面代码无法打开 Pointer 消息，我看了他代码，发现他写错了，所以我把全部  App.config 代码写出来。

```csharp
<?xml version="1.0" encoding="utf-8"?>

<configuration>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7" />
  </startup>
  <runtime>
    <AppContextSwitchOverrides value="Switch.System.Windows.Input.Stylus.EnablePointerSupport=true" />
  </runtime>
</configuration>
```

参见：[Mitigation: Pointer-based Touch and Stylus Support](https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/mitigation-pointer-based-touch-and-stylus-support?redirectedfrom=MSDN )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
