# dotnet 6 通过 DOTNET_ROOT 让调起的应用的进程拿到共享的运行时文件夹

我的应用是独立发布的，在用户的设备上不需要额外去安装 .NET 运行时。但是我的应用有一个需求是下载另一个应用作为插件，由本应用调起插件进程。本文告诉大家如何解决调用插件的进程时，赋值给插件进程运行时的文件夹路径，解决环境依赖

<!--more-->
<!-- CreateTime:2022/4/20 20:05:02 -->

<!-- 博客 -->
<!-- 发布 -->

我遇到的问题是，如何让调起的插件的进程共用我所在的应用的运行时，而不需要下载插件的时候，另外下载运行时文件

如果插件和我的应用在相同的文件夹下，那自然没有问题，插件默认就能使用我应用的运行时文件。但我的应用默认是被安装到 Program File 文件夹下的，如果下载的插件也放入到此文件夹下，那就需要管理员权限了

而如果我将插件放入到 AppData 文件夹下，那自然不需要管理员权限，但是插件运行就找不到运行时文件夹了。如果将运行时所需要的文件也拷贝过去，那就会被用户吐槽我占用了太多 C 盘空间了

一个好的解决方法是通过环境变量的方式，在 dotnet 6 里面，可以通过 `DOTNET_ROOT` 环境变量，或者 x86 的 `DOTNET_ROOT(x86)` 环境变量，让应用执行起来时，了解去哪里获取运行时依赖

解决方法是在当前应用进程里设置环境变量，如此通过 Windows 的环境变量机制，由当前应用启动的进程都默认会继承当前应用的环境变量的机制，可以让被当前应用启动的插件进程拿到环境变量，从而了解需要从哪个文件夹加载运行时

```csharp
        /// <summary>
        /// 加上环境变量，让调用的启动进程也自动能找到运行时
        /// </summary>
        static void AddEnvironmentVariable()
        {
            string key;
            if (Environment.Is64BitOperatingSystem)
            {
                // https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-environment-variables
                key = "DOTNET_ROOT(x86)";
            }
            else
            {
                key = "DOTNET_ROOT";
            }

            var runtimeFolder = "xxx 绝对路径，例如当前应用所在的文件夹";
            Environment.SetEnvironmentVariable(key, runtimeFolder);
        }
```

给自己的应用执行如上代码，即可给当前进程加上环境变量，从而让当前进程启动的其他进程也拿到相同的变量

以上代码是给 x86 应用设置的，如果自己的应用和插件应用同时都是 x64 的，那么只需要使用 `DOTNET_ROOT` 即可

另外，以上有一个坑就是对 x86 和 x64 混合应用不友好，如果有混合使用的，记得需要做额外的拆分逻辑，让对应的应用加载到符合预期的运行时

更多请看 

[.NET environment variables - .NET CLI](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-environment-variables?WT.mc_id=WD-MVP-5003260 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
