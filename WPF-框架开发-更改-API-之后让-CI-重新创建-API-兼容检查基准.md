
# WPF 框架开发 更改 API 之后让 CI 重新创建 API 兼容检查基准

本文是记录在开发 WPF 框架遇到的问题。
在 WPF 框架里面，限制了 API 的变更，所有关于 API 的变动都会触发 API 兼容检查。只有在明确此 API 变更是合理的之后，才能被接受更改。而此兼容判断将会让一些本地构建失败，本文将告诉大家如何使用官方的方法重新创建 API 基准，让本地构建通过

<!--more-->


<!-- CreateTime:2021/1/9 8:50:33 -->


<!-- 发布 -->

本文的 WPF 框架开发指的是给 WPF 这个框架进行开发，而不是基于 WPF 开发应用。当前整个 WPF 已完全开源，任何人都可以获取所有源代码以及参与开发

在 WPF 框架里面，限制了 API 的变更，表现就是在 CI 的时候，如果发现了存在 API 的变更，那么将 CI 不通过，提示代码如下

```
 error : TypesMustExist : Type 'Path_to_my_class' does not exist in the implementation but it does exist in the contract
. [...\src\PresentationFramework\PresentationFramework.csproj]
...\eng\WpfArcadeSdk\tools\ApiCompat.targets(239,5): error : ApiCompat failed for '...\artifacts\bin\PresentationFramework\Debug\netcoreapp5.0\PresentationFramework.dll' [...\src\Microsoft.DotNet.Wpf\src\PresentationFramework\PresentationFramework.csproj]
```

上面代码提示的 `ApiCompat.targets(239,5): error : ApiCompat failed` 就是告诉开发者说 API 兼容判断不通过

如果我明确是需要接受此 API 变更的，如加入新方法或者删除某个属性等，那么只需要重新运行生成 API 兼容数据就可以了。此时将自己的变更加入到 API 兼容数据里

重新运行的方法是在 Build.cmd 里面加上 BaselineAllAPICompatError 参数，如下面代码

```
Build.cmd /p:BaselineAllAPICompatError=true
```

官方文档请看 [wpf/Documentation/api-compat.md](https://github.com/dotnet/wpf/blob/9c5dd1acfe9037f92f6c91b05ab2f7e18edf55b7/Documentation/api-compat.md )

当然了，如果在进行频繁更改，那么不断执行重新生成 API 兼容数据，是一件低效率的事情。另一个方法就是不开启 API 兼容判断，做法就是删除 ApiCompat.targets 的判断代码，代码放在 [https://github.com/dotnet/wpf/blob/d5673edfbd274e4081d46d377ca4c0e6d3028ae2/eng/WpfArcadeSdk/tools/ApiCompat.targets#L239](https://github.com/dotnet/wpf/blob/d5673edfbd274e4081d46d377ca4c0e6d3028ae2/eng/WpfArcadeSdk/tools/ApiCompat.targets#L239) 请在你本地找到对应的代码删除

```
<Error Condition="'$(ApiCompatExitCode)' != '0'" Text="ApiCompat failed for '$(TargetPath)'" /> 
```

在 eng/WpfArcadeSdk/tools/ApiCompat.targets 找到上面代码，删除上面代码就可以

特别感谢 [Ryland](https://github.com/ryalanms) 大佬告诉我这个方法

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。