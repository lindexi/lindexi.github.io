# 如何在 RunAs 启动的软件传入带空格的路径带空格参数

使用 RunAs 可以让程序使用普通用户或管理员权限运行，本文告诉大家如何 传入带空格的路径

<!--more-->
<!-- CreateTime:2020/2/6 11:52:14 -->

<!-- 发布 -->

用 runas 可以[以指定的权限启动一个进程（非管理员、管理员）](https://blog.walterlv.com/post/start-process-in-a-specific-trust-level.html ) 在传入参数如下

```
runas /trustlevel:0x20000 .\lindexi.exe
```

如果我的文件是放在带空格文件夹 `E:\带空格 文件夹\lindexi.exe` 可以如何运行？ 请加上引号

```
runas /trustlevel:0x20000 "E:\带空格 文件夹\lindexi.exe"
```

如果我需要传入参数，可以如何写

```
runas /trustlevel:0x20000 "E:\带空格 文件夹\lindexi.exe 参数"
```

如果我的参数有空格，可以如何写

```
runas /trustlevel:0x20000 "E:\带空格 文件夹\lindexi.exe \"空格 内容\" 第二个参数 "
```

也就是在 runas 传入运行的文件，存在空格只需要将路径放在引号内。如果要传入参数，那么将传入路径和参数放在相同的引号内。如果参数有空格，将参数放在 `\"` 内

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
