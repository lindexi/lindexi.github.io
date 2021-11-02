
# WPF 获取 MediaContext 的方法

本文告诉大家如何通过反射获取 MediaContext 对象。在 WPF 里面，通过 MediaContext 对象可以用来控制渲染

<!--more-->



<!-- 发布 -->

在调试下，通过 Dispatcher 对象，可以看到有 Reserved0 不公开的属性，此属性就是 MediaContext 对象，如下面方法，通过反射获取此属性

```csharp
            var propertyInfo = typeof(Dispatcher).GetProperty("Reserved0", BindingFlags.NonPublic | BindingFlags.Instance);
            var mediaContext = propertyInfo.GetMethod.Invoke(Dispatcher, null);
```

如此即可获取到 MediaContext 对象

接下来可以通过程序集获取 MediaContext 类型，从而实现反射调用方法

```csharp
            var mediaContextType = typeof(Visual).Assembly.GetType("System.Windows.Media.MediaContext");
```

如尝试自己调用 DUCE 触发渲染

```csharp
            var methodInfoGetChannels = mediaContextType.GetMethod("GetChannels", BindingFlags.NonPublic | BindingFlags.Instance);
            var channelSet = methodInfoGetChannels.Invoke(mediaContext, null);

            var typeChannelSet = typeof(Visual).Assembly.GetType("System.Windows.Media.Composition.DUCE+ChannelSet");
            var fieldInfoChannel = typeChannelSet.GetField("Channel", BindingFlags.NonPublic | BindingFlags.Instance);
            var channel = fieldInfoChannel.GetValue(channelSet);

            var typeChannel = typeof(Visual).Assembly.GetType("System.Windows.Media.Composition.DUCE+Channel");
            var hChannelFieldInfo = typeChannel.GetField("_hChannel", BindingFlags.NonPublic | BindingFlags.Instance);
            var hChannel = (IntPtr)hChannelFieldInfo.GetValue(channel);

            ResourceHandle handle = default;
            var result = MilResource_CreateOrAddRefOnChannel(hChannel, ResourceType.TYPE_BRUSH, ref handle);

        [DllImport(MilCore)]
        internal static extern /*HRESULT*/ int MilResource_CreateOrAddRefOnChannel(
            IntPtr pChannel,
            ResourceType resourceType,
            ref ResourceHandle hResource
        );

        internal const string WCP_VERSION_SUFFIX = "_cor3";
        internal const string MilCore = "wpfgfx" + WCP_VERSION_SUFFIX + ".dll";
```

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/23c0515af613acc5025f95c17ea254fc573375ce/LereleweawaLinojairgeefonechal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/23c0515af613acc5025f95c17ea254fc573375ce/LereleweawaLinojairgeefonechal) 欢迎访问

可以通过如下方式获取本文代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 23c0515af613acc5025f95c17ea254fc573375ce
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 LereleweawaLinojairgeefonechal 文件夹






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。