# Inno Setup 安装包脚本 Run 的 Flags 标记

在制作安装包的时候，可以在 Inno Setup 安装包脚本的 Run 里面添加在解压缩安装包文件完成之后，整个安装结束之前执行指定的命令，是作为定制化最高的内容

<!--more-->
<!-- CreateTime:2019/10/26 8:42:39 -->

<!-- csdn -->
<!-- 标签: 安装包,InnoSetup -->

有小伙伴觉得安装包脚本比较难写，也不熟悉，也不好调试。推荐的方法是自己写安装辅助 exe 程序，在安装包解压缩完成之后调用辅助安装程序，这样安装逻辑可以放在安装程序，而安装程序本身可以使用自己熟悉的语言开发

在 Inno Setup 安装包脚本，可以在 Run 里面添加执行命令，如下面代码

```Inno
[Run]
Filename: "{app}\After.bat"; Description: "{cm:LaunchProgram,安装程序}";Flags: runhidden 
```

上面代码就是在解压缩完成之后调用 After.bat 执行批处理，上面的 `{app}` 就是程序解压缩文件夹，也就是路径是绝对的

而想要执行命令，就会遇到如何执行命令的问题，有一些命令是让用户勾选才执行，如打钩点击完成启动 xx 程序，这部分就需要用到 Flags 的值

在 Flags 多个不同值用空格分开，可选内容如下

## 32bit

将 `{sys}` 常量应用到 32 的系统，将会修改 Filename 和 WorkingDir 的内容，默认在32系统上安装将会应用。这个标记设置之后不能加上 `shellexec` 内容

## 64bit

当使用 Filename 和 WorkingDir 参数将会设置里面用到的 `{sys}` 常量为 64 系统的工作路径，默认在 64 系统上安装将会应用

```inno
[Run]
Filename: "{sys}\After.bat"; Description: "欢迎访问我博客 blog.lindexi.com 大量 WPF 博客";Flags: 64bit 
```

此时上面代码的 `{sys}` 将会根据设置的标记选用 32 或 64 系统文件夹

## nowait

执行命令的时候，安装包进程不等待此命令执行完成

不能和 `waituntilidle` 和 `waituntilterminated` 组合

## runascurrentuser

用当前用户权限运行

## runasoriginaluser

传递权限运行

## runhidden

执行命令，但是隐藏命令的界面

主要是调用批处理或命令行程序时，不会显示控制台界面

用上此标记可以在安装完成之前调用批处理程序时，不会让安装包调用时显示控制台界面

## runmaximized

让调用的程序最大化

## runminimized

让调用的程序最小化

## shellexec

用默认程序打开传入的文件，在传入的文件不是可执行文件时，可以加上这个标记

## skipifdoesntexist

如果传入的文件不存在，那么什么都不做就跳过

## skipifnotsilent

如果当前不是静默安装模式，那么跳过

在 Inno Setup 安装包，可以通过 `/silent` 命令或 `/verysilent` 命令进行静默安装

使用 `/silent` 时，静默安装，但如果又报错，还是会提示，并且有进度条也就是用户能看见进度条界面，加上这个命令可以让安装包自动安装不需要让用户选择

使用 `/verysilent` 时，静默安装，更强制，不过是否报错，都不会有任何提示，也就是用户什么都没看见，用这个选项可以在后台静默安装。这样就能做到安装包在后台无界面安静安装

## skipifsilent

如果当前是静默安装模式，那么跳过

## postinstall

将会在安装完成界面创建一个可选按钮，让用户勾选或不勾选，当用户勾选时将会执行。默认选项是勾选

## unchecked

配合 `postinstall` 将会修改默认值是不勾选，也就是点击完成不会执行命令

## waituntilidle

等待调用的命令在等待用户输入时才执行下一步

## waituntilterminated

等待调用的命令暂停或退出

如果以上内容有不明白的，请点击官方文档 [Inno Setup Help](http://www.jrsoftware.org/ishelp/index.php?topic=compilercmdline )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
