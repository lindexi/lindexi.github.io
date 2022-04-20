# Total Commander 使用 mklink 建立文件夹链接 将 C 盘文件迁移到其他盘

在安装完成了 100000000 个软件之后，我 1T 的 C 盘的空间终于不足了，由于安装了大量的特别挑的不专业的软件，强行放在其他的盘将水土不服。于是在老师傅的指导下，我采用了 mklink 神奇命令行，通过 mklink 可以实现将实际的文件放在其他的盘，然后建立链接到原先的地方，在应用上层基本感知不到文件的实际存储地方已经被更换。也就是物理上的文件存放是可以在另一个磁盘上，但软件以为文件还在原来的地方

这是一个不错的方法，我需要处理的文件夹还稍微有点多，就想着写一点小工具，在 Total Commander 里面，辅助我快速完成工作

<!--more-->
<!-- CreateTime:2022/4/18 8:36:59 -->


<!-- 标签：TotalCommander -->
<!-- 发布 -->
<!-- 博客 -->

使用 Total Commander 的一个大优势就是可定制性特别强，例如本文用到的功能就是自定义工具条，新建一个命令按钮，在命令按钮上通过命令行调用的形式，调用到其他工具，例如本文用到的 mklink 工具

本文用到的 mklink 工具属于一个低频使用的功能，不合适作为快捷键，否则过几天就忘了。而做成工具条上的一个按钮，说不定下次我快忘了的时候，还能通过看到这个图标记得有这个功能

在 Total Commander 上给工具条上添加新的按钮的方法有很多，我推荐的就是在工具条上右击，点击更改，进入编辑界面，在编辑界面上添加功能

![](http://image.acmx.xyz/lindexi%2F20224171453223174.jpg)

先点击添加按钮，接着输入命令 cmd 和参数 `/C " mklink /d %T%N %P%N "` 最后选一个好看的图标就完成了，如下图

![](http://image.acmx.xyz/lindexi%2F2022417151158058.jpg)

使用方法就是，先在一侧选中一个文件夹，然后导航期望建立链接的文件夹到另一侧，点击一下上面新建的按钮即可完成建立文件夹软链接

以上的命令细节就是通过 cmd 调用 mklink 工具建立文件夹软链接

使用 cmd 调用其他命令需要传参数的时候，可选使用 `/C` 参数，也就是 `cmd /C "其他命令行的其他工具"` 的格式。另外的 cmd 的可选参数，还请自行谷歌

使用 mklink 时，可选使用 `/d` 命令建立软链接，细节请参阅 [解决 mklink 使用中的各种坑（硬链接，软链接/符号链接，目录链接） - walterlv - 博客园](https://www.cnblogs.com/walterlv/p/10236475.html)

而 `%T` 和 `%N` 等，这是 Total Commander 提供的参数，可以分别替换为对侧文件夹路径，以及当前选择的文件名或文件夹名，详细请看如下从官方拷贝的描述

```
%P causes the source path to be inserted into the command line, including a backslash (\) at the end.
%N places the filename under the cursor into the command line. 
%T inserts the current target path. Especially useful for packers.
%M places the current filename in the target directory into the command line.
%O places the current filename without extension into the command line. 
%E places the current extension (without leading period) into the command line.
%S insert the names of all selected files into the command line. Names containing spaces will be surrounded by double quotes. Please note the maximum command line length of 32767 characters.
%S10
insert the names of the first 10 selected files (max.) into the command line. Allows you to limit the number of file names passed to the program. You can use any other number.
%P%S
insert the names of all selected files into the command line, with full path. Names containing spaces will be surrounded by double quotes. Do NOT put quotes around %P%S yourself!
%R like %S, but with selected names from the target panel
%C1 Like the first parameter of "Compare by content": First selected file, or file under cursor
%C2 Like the second parameter of "Compare by content": Second selected file, or first selected in target panel, or file with same name in target panel. Note: If the right panel is active and less than 2 files are selected, %C1 and %C2 will be reversed.
%C3..%C9
Selected files Nr. 3 .. 9 in source panel, empty if not enough selected
%c1..%c9
Like %C1..%C9, but with 8.3 names and paths
Notes: %N and %M insert the long name, while %n and %m insert the DOS alias name (8.3). %P and %T insert the long path name, and %p and %t the short path name (Same for %o, %e and %s). %p/%t inserts the ftp URL of the directory for ftp connections.
By putting %P, %p, %T or %t directly in front of %S or %s, the path name is inserted with the file name for each file. Example: %P%S inserts the long path and file name for all selected files.
%% inserts the percent-sign.
%L, %l, %F, %f, %D, %d, %WL, %WF, %UL, %UF
create a list file in the TEMP directory with the names of the selected files and directories, and appends the name of the list file to the command line. The list is deleted automatically when the called program quits. Only one list per command is supported. 10 types of list files can be created:
%L Long file names including the complete path, e.g. c:\Program Files\Long name.exe
%l (lowercase L) Short file names including the complete path, e.g. C:\PROGRA~1\LONGNA~1.EXE
%F Long file names without path, e.g. Long name.exe
%f Short file names without path, e.g. LONGNA~1.EXE
%D Short file names including the complete path, but using the DOS character set for accents.
%d Short file names without path, but using the DOS character set for accents.
%UL,%UF
like %L and %F, but with UTF-8 Unicode list file (with byte order mark)
%WL,%WF
like %L and %F, but with a UTF-16 Unicode list file (with byte order mark)
%v Insert virtual file name in case of file system plugins like "virtual panel", where %N pastes the name of the real file (in the file system) to which the entry points to
%V Like %v, but including the full path (including the plugin name)
%X Interprets the following parameters after this parameter as left/right instead of source/target:
  %P, %p (left path), %T, %t (right path), %N, %n (left name), %M, %m (right name),
  %S, %s (left selected), %R, %r (right selected)
  Example: %X%P %T  sends left and right path to e.g. an external sync tool
%x Interprets the following parameters after this parameter again as source/target
  Example: %X%P %x%P sends left and source path to the called program
%Y anywhere in the parameters: Pass empty list to program when using one of the List parameters like %L. Otherwise, the file under the cursor would be passed.
%Z anywhere in the parameters: Allow to pass archives as path to programs for %P or %T when inside an archive
  Example: %Z%P passes name of archive to external tool when TC shows contents of an archive
Only when defining alias commands:
%A Inserts the rest of the entered command line
%A1..%A9
Inserts the first until the ninth parameter.
Example: Alias op points to Command: totalcmd.exe Parameter: /L=%A1 /R=%A2
-> Command line: op c:\dir1 d:\dir2 will create command: totalcmd.exe /L=c:\dir1 /R=d:\dir2
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
