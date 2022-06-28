# WPF 通过 Switch.MS.Internal.EnableWeakEventMemoryImprovements 开关开启弱事件内存优化

默认的 WPF 为了保持行为兼容，没有开启弱事件的内存优化。可以在 WPF 中指定 Switch.MS.Internal.EnableWeakEventMemoryImprovements 和 Switch.MS.Internal.EnableCleanupSchedulingImprovements 开关来让 WPF 运行在 .NET Framework 4.8 或 .NET Core 3.0 以上版本时，自动开启弱事件内存优化。通过这个开关，将会更改部分行为，但是基本上不会有影响，因为影响的都是内存啥时候回收。这些开关和 WPF 应用所使用的开发版本无关，只和 WPF 应用所运行在的设备环境有关，如果在运行的设备上安装了 .NET Framework 4.8 版本，那么自动将会应用上，否则这个开关就和没有写一样

<!--more-->
<!-- CreateTime:2021/3/23 21:10:43 -->

<!-- 发布 -->

这个功能是在 .NET Framework 4.8 新建的，同时也在 .NET Core 3.0 中。在代码中开启的方法如下

打开 App.xaml.cs 文件，在构造函数添加下面代码

```csharp
        public App()
        {
            AppContext.SetSwitch("Switch.MS.Internal.EnableWeakEventMemoryImprovements", true);
            AppContext.SetSwitch("Switch.MS.Internal.EnableCleanupSchedulingImprovements", true);
        }
```

在开启这个功能之后，影响的是 WPF 框架本身，通过开源的 WPF 框架源代码可以了解到，在 `src\Microsoft.DotNet.Wpf\src\WindowsBase\MS\Internal\BaseAppContextSwitches.cs` 文件有如下定义

```csharp
        /// <summary>
        /// Enable/disable various perf and memory improvements related to WeakEvents
        /// and the cleanup of WeakReference-dependent data structures
        /// </summary>
        #region EnableWeakEventMemoryImprovements

        internal const string SwitchEnableWeakEventMemoryImprovements = "Switch.MS.Internal.EnableWeakEventMemoryImprovements";
        private static int _enableWeakEventMemoryImprovements;

        public static bool EnableWeakEventMemoryImprovements
        {
            [MethodImpl(MethodImplOptions.AggressiveInlining)]
            get
            {
                return LocalAppContext.GetCachedSwitchValue(SwitchEnableWeakEventMemoryImprovements, ref _enableWeakEventMemoryImprovements);
            }
        }

        #endregion
```

这个属性的使用方是在 WPF 中定义的弱事件管理类和相关使用的逻辑，包括

- src\Microsoft.DotNet.Wpf\src\PresentationFramework\MS\Internal\Data\StaticPropertyChangedEventManager.cs
- src\Microsoft.DotNet.Wpf\src\PresentationFramework\MS\Internal\Data\ValueChangedEventManager.cs
- src\Microsoft.DotNet.Wpf\src\WindowsBase\MS\Internal\WeakEventTable.cs
- src\Microsoft.DotNet.Wpf\src\WindowsBase\System\ComponentModel\PropertyChangedEventManager.cs
- src\Microsoft.DotNet.Wpf\src\Shared\MS\Internal\ReaderWriterLockWrapper.cs

这几个类都是用来监听各个事件的，如依赖属性本身的通知和 INotifyPropertyChanged 接口的事件等，详细请看 WPF 框架源代码

我的建议是可以在项目中加上此开关，因为阅读完成了 WPF 框架的源代码，没有发现会影响业务功能的逻辑，逻辑上都是性能优化。 未来可能这个属性的默认值会改为 true 表示开启，但当前 .NET 6 下还没有此计划，可以通过以下代码进行判断默认值

```csharp
            // WindowsBase
            var assembly = typeof(DependencyObject).Assembly;

            // Switch.MS.Internal.EnableWeakEventMemoryImprovements 默认没有开启
            var baseAppContextSwitchesType = assembly.GetType("MS.Internal.BaseAppContextSwitches");

            var enableWeakEventMemoryImprovementsProperty = baseAppContextSwitchesType.GetProperty("EnableWeakEventMemoryImprovements");

            var value = enableWeakEventMemoryImprovementsProperty.GetMethod.Invoke(null, null);
```

以上的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/9ed2848163f22415f8aec33b07a8a592f744b729/LeekilerelalljarFayjululeayem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9ed2848163f22415f8aec33b07a8a592f744b729/LeekilerelalljarFayjululeayem) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9ed2848163f22415f8aec33b07a8a592f744b729
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9ed2848163f22415f8aec33b07a8a592f744b729
```

获取代码之后，进入 LeekilerelalljarFayjululeayem 文件夹

文档请看 [dotnet-framework-early-access/changes.md at master · microsoft/dotnet-framework-early-access](https://github.com/microsoft/dotnet-framework-early-access/blob/master/release-notes/NET48/build-3734/changes.md )

更多开关请看 [WPF Application Compatibility switches list](https://blog.lindexi.com/post/WPF-Application-Compatibility-switches-list.html )

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
