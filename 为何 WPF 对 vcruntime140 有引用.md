# 为何 WPF 对 vcruntime140 有引用

通过阅读 WPF 官方开源仓库的代码和文档，可以了解到在进行独立发布的时候会在仓库里面带上 vcruntime140 的原因

<!--more-->
<!-- 发布 -->

在独立发布的时候，可以在仓库里面找到 `vcruntime140.dll` 这个文件

这是因为 WPF 框架用到了两个使用 `C++\CLI` 的项目，分别是 `DirectWriteForwarder.dll` 和 `System.Printing.dll` 库

这两个库在进行发布的时候需要引用 `vcruntime140.dll` 这个文件，如果进行 Debug 下发布的时候需要引用 `vcruntime140d.dll` 文件

如果不将这个文件带入独立发布的文件里面，那么将要求用户去安装 VC 的运行时，这个将会带来一个神坑，就是 C++ 环境问题。而这里的 WPF 只是需要这个文件而已，所以在独立发布的时候将带上这个文件，此时用户端就无需安装 VC 运行时

因为默认的 `vcruntime140.dll` 会在应用程序运行的时候根据环境的不同会有选择冲突，同时如果有依赖某些有趣的库，这些库也许对 vc 运行时也有依赖，但是 WPF 用的版本也许和引用的库是不相同的，为了减少这部分的冲突，微软改名部将重写分发的`vcruntime140.dll` 修改为 `vcruntime140_cor3.dll` 也就是实际发布的时候看到的文件

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
