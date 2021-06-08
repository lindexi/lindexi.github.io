# 让 dotnet 命令行输出作为英文的方法

在我的设备上，默认的 dotnet 命令行都是输出中文，如我输入 dotnet build 命令，里面的错误提示也是中文。在我想要和国外的小伙伴报坑时，如果里面有很多中文，我觉得他将会看不懂，从而不理我。本文来告诉大家如何让 dotnet 的命令行输出英文，从中文切换语言为英文

<!--more-->
<!-- 发布 -->

方法需要有两步，第一步是设置环境变量，如果执行 dotnet build 命令，是在命令行里面执行，可以通过如下代码设置环境变量。通过如下代码设置，只有在这个命令行里面有用，不会影响其他应用

```
set DOTNET_CLI_UI_LANGUAGE=en
```

第二步是删除对应的 sdk 版本的 zh 开头的语言。如我的 sdk 是 6.0.100-preview.1.21103.13 那么我就需要去 dotnet 的安装文件夹里面删除 zh 开头的语言

```
删除 c:\Program Files\dotnet\sdk\6.0.100-preview.1.21103.13\ 的 zh 语言
```

删除的时候需要管理员权限，我推荐只是将 zh 开头的文件夹修改命名即可，这样在切换到中文的时候，只需要修改回文件夹的命名

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
