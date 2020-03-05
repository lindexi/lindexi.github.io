# win10 sdk 是否向下兼容

向下兼容（downward compatibility），又称向后兼容（backward compatibility）、回溯兼容，在计算机中指在一个程序、库或硬件更新到较新版本后，用旧版本程序创建的文档或系统仍能被正常操作或使用（包括输入数据）、在旧版本库的基础上开发的程序仍能正常编译运行，或较旧版的硬件仍可在新版使用的情况。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


在我们安装了 sdk 为 14393 之后，选择项目工程为 14393 ，那么编译出来的程序是不是不可以在win10 版本为红石之前的运行？

答案是：不可以

但是之后是不是存在向下兼容，以后可能是。

但是现在是不可以的，从 10240 编译的程序经常无法在 14393 以上的系统打开，微软没有做出保证，所以做出的程序是在自己的版本可以运行，但是在和他不同版本的就无法正常运行。

微软在向下兼容这里做不好，原因：因为用户都会更新系统，所以开发只有使用最新的就可以，不需要关心那些比较老的系统。


微软系统是自动更新，所以会自动把用户更新为最新的，开发的时候，如果在新版本发布一个月内，那么需要使用最低版本为上一个的，其他的情况建议使用最新的版本。

现在开发需要安装所有的sdk？微软认为开发只需要安装最新的sdk就好，对于老的版本，不建议开发。

上面说的都不是来自微软官方。

参见：http://stackoverflow.com/a/40233780/6116637

[Microsoft releases new Windows 10 SDK ](https://www.onmsft.com/news/microsoft-releases-new-windows-10-sdk-preview-associated-mobile-emulator)




如果打开一个之前版本的，会提示

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201751892557.jpg)

这时可以修改 项目 来打开

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201751892636.jpg)

把10240改为 15063 我就可以打开



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。