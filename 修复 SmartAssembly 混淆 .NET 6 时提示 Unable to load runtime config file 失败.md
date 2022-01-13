# 修复 SmartAssembly 混淆 .NET 6 时提示 Unable to load runtime config file 失败

本文告诉大家如何修复使用 SmartAssembly 7.5 混淆 .NET 6 应用时提示 Unable to load runtime config file 失败

<!--more-->
<!-- CreateTime:2022/1/11 21:09:35 -->

<!-- 发布 -->

因为在 SmartAssembly 7.5 发布时，还没有正式发布 .NET 6 版本，好在 IL 是兼容的，因此跑起来也没有什么问题。为了让 SmartAssembly 能跑起来，只需要新建一个 Xxx.runtimeconfig.json 文件，在文件里面放入以下内容

```json
{
  "runtimeOptions": 
  {
    "tfm": "net6.0",
    "framework": 
    {
      "name": "Microsoft.NETCore.App",
      "version": "6.0.0"
    }
  }
}
```

这个 `Xxx.runtimeconfig.json` 里的 Xxx 没啥要求，似乎写啥都行，写 Xxx 也行，只要有这个文件即可。那同文件夹下有多个 runtimeconfig 文件？没问题，放多个文件也没问题

更多请看 [Failed to load assembly. Unable to load runtime config file. — Redgate forums](https://forum.red-gate.com/discussion/88584/failed-to-load-assembly-unable-to-load-runtime-config-file )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
