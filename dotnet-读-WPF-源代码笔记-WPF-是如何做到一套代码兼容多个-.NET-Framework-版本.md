
# dotnet 读 WPF 源代码笔记 WPF 是如何做到一套代码兼容多个 .NET Framework 版本

在 .NET Framework 时代里面，有一组有趣的概念，那就是 SDK 和 Runtime 这两个概念。开发模式十分有趣，在开发者设备上，可以指定 .NET Framework 的 SDK 版本，例如指定 .NET Framework 4.5 版本。开发完成之后，分发给到用户，用户的电脑上所安装的 .NET Framework 基本都是 Runtime 版本。应用程序要求运行的 Runtime 版本一定要大于等于 SDK 的指定版本号
这就有一个非常有趣的问题了，我开发环境使用的 SDK 是低版本，例如 .NET Framework 4.5 版本。但用户的电脑上所安装的 .NET Framework 的 Runtime 版本是高版本，例如是 .NET Framework 4.7 版本，中间距离过了几年的版本。那行为如何保证相同？如果行为不能保证相同，那将会出现行为差异，在我的开发设备上跑得好好的，在一些用户电脑上，将会因为运行时基础库的版本差异从而变更行为，应用程序就不能符合预期跑起来了。事实上，咱没有碰到过这个问题，这是因为在 .NET Framework 层做了很多兼容处理逻辑，其中就包括本文要和大家聊的 WPF 框架的兼容行为

<!--more-->


<!-- CreateTime:2022/6/27 8:20:53 -->

<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

本文所聊的仅仅只是 .NET Framework 的老故事，本文对 .NET Core 3.0 及以上版本基本无效，因为其开发模式变更了

在 .NET Framework 的开发模式里面，开发者采用的是 .NET Framework SDK 进行开发，可以指定版本号。指定版本号后，生成的应用程序要求在运行的设备上，一定要安装大于或等于指定版本的运行时才可以

在 .NET Framework 下，无法让 .NET Framework 的运行时跟随应用进行分发，应用只能采用系统里安装的版本。存在的问题就是应用在开发阶段是无法预知实际运行在用户端，用户的设备上所安装的 .NET Framework 版本的

如果对于相同的一套代码，在不同的 .NET Framework 版本下，有行为上的差异，那这是一个非常可怕的事情，甚至开发端啥都做不了。想想如以下的例子，假如我是 .NET Framework 版本的 WPF 框架的开发者，我写了一个叫 Foo 的方法，这个方法我是在 .NET Framework 4.5 定义的，用于支持某个关键逻辑。但是我在 .NET Framework 4.6 版本，我觉得这个方法应该抛出异常，于是就让这个 Foo 方法被调用时一定抛出异常

这个时候，预计开发者将会寄刀片给我。因为开发者将会发现他的应用能在一些用户的设备上，符合预期工作。在一些用户的设备上，一定会炸掉。其原因就是我在高版本上变更了行为。根本原因就是开发者无法独立分发 .NET Framework 导致应用程序的运行受实际的环境影响

当然了，这个问题在 .NET Core 或 .NET 下已经被彻底解决，那就是允许开发者自己独立发布，不依赖系统的 .NET 环境。如此的开发模式，既可以降低基础运行时框架开发者的压力，也能让上层应用业务端的开发者能更好的控制应用运行，减少奇怪的环境问题

作为考古的博客，继续回到主题

在实际的开发里面，几乎没有遇到上面说的由于 .NET Framework 的 Runtime 版本不同，而导致的行为不同从而导致的问题

这是因为在 WPF 里面做了很多的兼容的处理逻辑。例如大家都知道的，在 .NET Framework 4.0 到 .NET Framework 4.5 是存在极大的变动的。在 WPF 里面，为了编写代码方便，就添加了一个叫 FrameworkCompatibilityPreferences 的类型，里面有一个属性如下

```csharp
    public static class FrameworkCompatibilityPreferences
    {
        static FrameworkCompatibilityPreferences()
        {
#if NETFX && !NETCOREAPP
            _targetsDesktop_V4_0 = BinaryCompatibility.AppWasBuiltForFramework == TargetFrameworkId.NetFramework
                && !BinaryCompatibility.TargetsAtLeast_Desktop_V4_5;
#elif NETCOREAPP
            // When building for NETCOREAPP, set this to false
            // to indicate that quirks should be treated as if they are running on 
            // .NET 4.5+
            _targetsDesktop_V4_0 = false;
#else
            _targetsDesktop_V4_0 = false;
#endif
        }

        // CLR's BinaryCompatibility class doesn't expose a convenient way to determine
        // if the app targets 4.0 exactly.  We use that a lot, so encapsulate it here
        static bool _targetsDesktop_V4_0;

        internal static bool TargetsDesktop_V4_0
        {
            get { return _targetsDesktop_V4_0; }
        }
    }
```

在各处代码里面对此属性的判断，从而加上兼容的逻辑，例如绑定的代码

```csharp
    public class Binding : BindingBase
    {
        public PropertyPath Path
        {
            get { return _ppath; }
            set
            {
                _ppath = value;

                if (_ppath != null && _ppath.StartsWithStaticProperty)
                {
                    if (_sourceInUse == SourceProperties.None || _sourceInUse == SourceProperties.StaticSource 
                    	// 在 .NET Framework 4.5 和以上的版本更改了行为，需要加上兼容逻辑
                    	|| FrameworkCompatibilityPreferences.TargetsDesktop_V4_0) 
                    {
                        // net 4.5 breaks static bindings - this is for compat
                        SourceReference = StaticSourceRef;
                    }
                    else
                        throw new InvalidOperationException(SR.Get(SRID.BindingConflict, SourceProperties.StaticSource, _sourceInUse));
                }
            }
        }
    }
```

毫无疑问，通过一个属性的方式确实简单明了。缺点也很明显，代码里面将会充斥版本判断逻辑，这个方式是走不远的。这个也算是兼容的包袱

作为一群对代码有追求的开发者，肯定是要做一些设计。于是大家可以看到 WPF 开源仓库里面，实际上现在主流的还有一层兼容逻辑，那就是功能开关

依托 System.AppContext 提供的组件之间建立松耦合的协定，从而将代码里面原本对版本强依赖的逻辑变更为对功能的支持

这是一个很好的思路，让功能的开关与具体的 .NET Framework 版本关联，让实际的功能逻辑和功能开关关联。如此即可进行逻辑控制且减少在代码里面判断版本的逻辑，也不会在后续随着版本越来越多，维护越来越难

另外，功能开关本身和具体的 .NET Framework 版本只是一个弱相关性而已，仅仅只是在应用程序运行的时候，在应用程序配置里面声明这个应用程序所期望运行的 .NET Framework 版本，再根据此声明使用的版本，决定开关的开启和关闭

例如在 AppContextDefaultValues 里的代码

```csharp
    internal static partial class AppContextDefaultValues
    {
        static partial void PopulateDefaultValuesPartial(string platformIdentifier, string profile, int targetFrameworkVersion)
        {
            switch (platformIdentifier)
            {
                case ".NETFramework":
                    {
                        if (targetFrameworkVersion <= 40601)
                        {
                            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.DoNotScaleForDpiChangesSwitchName, true);
                        }

                        if (targetFrameworkVersion <= 40602)
                        {
                            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.OverrideExceptionWithNullReferenceExceptionName, true);
                        }

                        if (targetFrameworkVersion <= 40702)
                        {
                            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.DoNotUsePresentationDpiCapabilityTier2OrGreaterSwitchName, true);
                        }

                        break;
                    }
                case ".NETCoreApp":
                    {
                        InitializeNetFxSwitchDefaultsForNetCoreRuntime();
                    }
                    break;
            }
        }

        private static void InitializeNetFxSwitchDefaultsForNetCoreRuntime()
        {
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.DoNotScaleForDpiChangesSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.OverrideExceptionWithNullReferenceExceptionName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.DoNotUsePresentationDpiCapabilityTier2OrGreaterSwitchName, false);

            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.DisableStylusAndTouchSupportSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.EnablePointerSupportSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.DisableDiagnosticsSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.AllowChangesDuringVisualTreeChangedSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.DisableImplicitTouchKeyboardInvocationSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.ShouldRenderEvenWhenNoDisplayDevicesAreAvailableSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.ShouldNotRenderInNonInteractiveWindowStationSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.DoNotUsePresentationDpiCapabilityTier3OrGreaterSwitchName, false);
            LocalAppContext.DefineSwitchDefault(CoreAppContextSwitches.AllowExternalProcessToBlockAccessToTemporaryFilesSwitchName, false);
        }
    }
```

当然，功能开关的作用可不只是用来做兼容逻辑，还可以用来实现特殊的配置，从而打开某些功能或者禁用某些功能，更方便开发者进行定制。更多的功能开关请看 [WPF Application Compatibility switches list](https://blog.lindexi.com/post/WPF-Application-Compatibility-switches-list.html)

这就是 WPF 框架对 .NET Framework 运行时版本不同进行的适配逻辑，通过在代码里面添加大量的判断条件，从而让开发者指定的运行版本和实际运行所采用的逻辑尽可能相同。这个兼容逻辑是 WPF 参考里面写代码实现的，也就是靠 WPF 框架的开发者保证的，这也就是本文开头我用了 `几乎` 而不是确定词来说不会在这里踩坑




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。