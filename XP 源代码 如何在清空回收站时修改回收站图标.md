# XP 源代码 如何在清空回收站时修改回收站图标

本文来聊聊在 XP 系统里面，是如何在清空回收站的时候修改回收站图标的

<!--more-->
<!-- 发布 -->

声明 我没有拿到微软泄露 XP 系统的源代码，以下逻辑都是我瞎说的，微软才不会写出如此逗比的代码

在 bitbuck.c 文件里面有 SHUpdateRecycleBinIcon 的实际实现，而这个方法实际上只是调用了 UpdateIcon 方法而已。大概的代码如下

```csharp
STDAPI_(void) SHUpdateRecycleBinIcon()
{
    UpdateIcon(!回收站是不是空的());
}
```

在 UpdateIcon 方法里面，先从注册表拿到回收站的图标，因为回收站的图标是可以让用户定制的

通过传入是否回收站是空的，获取不同的图标

拿到图标之后，调用 SHUpdateImage 方法将图标设置给回收站

这就是在 XP 系统里面的逻辑。上面代码也说明了 BitBucket 打不过 GitHub 的原因了，因为在 XP 上就是垃圾桶

感谢工具人 [少珺](https://blog.sdlsj.net/) 的协助

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
