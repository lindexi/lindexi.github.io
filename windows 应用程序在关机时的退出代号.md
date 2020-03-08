# windows 应用程序在关机时的退出代号

在 windows 应用程序关闭的时候可以通过进程的 ExitCode 了解这个应用是如何关闭的

<!--more-->
<!-- CreateTime:2018/11/19 14:31:38 -->

<!-- csdn -->

因为所有的应用都可以自己定义应用关闭的 ExitCode 所以如果应用是自己开发的，那么可以通过 ExitCode 作为约定。

普通的应用都是使用 0 作为应用程序正确处理了关闭，也就是正常的关闭。

在用户关机的时候，可以通过在注册表设置等待应用多久才会退出应用。

打开注册表，在 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control` 的 WaitToKillServiceTimeout 就是设置的超时时间，单位是毫秒

在关机的过程，如果发现应用没有自己退出，系统就会关闭软件。

这时软件拿到的是 ExitCode 是 1073807364 (0x40010004) 这个值相当于  -1073741510 (0xc000013a) 表示应用程序是在系统关闭的时候关闭，或者软件收到了 `ctrl+C` 或 `ctrl+Break` 关闭。也有找到某个应用程序里面越界访问被系统关闭也是 `-1073741510` 所以通过这个值判断是关机也是不对的
 
当前用户也可以自己设置 ExitCode 为 1073807364 所以如果不是自己写的程序，通过这个方式判断第三方程序的关闭是否是在系统关闭是不一定

其他的 ExitCode 请看 [System Error Codes](https://docs.microsoft.com/en-us/windows/desktop/Debug/system-error-codes )

[Exit code 1073807364 (0x40010004) - BOINC Wiki](https://boinc.mundayweb.com/wiki/index.php?title=Exit_code_1073807364_(0x40010004) )

[System Error Codes](https://docs.microsoft.com/en-us/windows/desktop/Debug/system-error-codes )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
