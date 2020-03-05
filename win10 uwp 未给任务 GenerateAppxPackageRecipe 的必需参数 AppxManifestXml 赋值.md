# win10 uwp 未给任务 GenerateAppxPackageRecipe 的必需参数 AppxManifestXml 赋值

本文告诉大家如何修复使用Release正常，debug编译正常，手机正常，就是 上传应用商店关联后，release就出现错误 未给任务“GenerateAppxPackageRecipe”的必需参数“AppxManifestXml”赋值

<!--more-->
<!-- CreateTime:2019/6/23 10:57:03 -->

<!-- csdn -->

在点击编译的时候也提示

```csharp
error MSB4131: “CreatePriFilesForPortableLibraries”任务不支持“UnprocessedReswFiles”
```

修复方法是创建一个应用，把原有的所有东西导入

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
