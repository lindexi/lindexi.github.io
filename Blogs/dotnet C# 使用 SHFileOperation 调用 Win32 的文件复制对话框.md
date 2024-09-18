---
title: dotnet C# 使用 SHFileOperation 调用 Win32 的文件复制对话框
description: 本文将和大家介绍如何在 dotnet C# 里面使用 SHFileOperation 调用 Windows 自带的文件复制对话框进行文件复制

<!--more-->

tags: dotnet C#
category: 
---

<!-- CreateTime:2024/08/10 07:19:12 -->

<!-- 发布 -->
<!-- 博客 -->

本文核心代码拷贝自 [C#中使用SHFileOperation调用Windows的复制文件对话框 - 季风哥 - 博客园](https://www.cnblogs.com/jifengg/archive/2013/05/13/3076131.html ) 文章，特别感谢大佬提供的方法

实现的效果图如下

<!-- ![](image/dotnet C# 使用 SHFileOperation 调用 Win32 的文件复制对话框/dotnet C# 使用 SHFileOperation 调用 Win32 的文件复制对话框0.png) -->
![](http://cdn.lindexi.site/lindexi%2F2024891944276857.jpg)

详细实现逻辑还请大家参阅： [C#中使用SHFileOperation调用Windows的复制文件对话框 - 季风哥 - 博客园](https://www.cnblogs.com/jifengg/archive/2013/05/13/3076131.html ) 

其中我遇到的坑是多个文件之间需要使用 `\0` 字符分割，我使用了 dotnet 新 API 对其进行更新，代码如下

```csharp
        pm.pFrom = string.Join(FILE_SPLITER, sourceFiles) + $"{FILE_SPLITER}{FILE_SPLITER}";
        pm.pTo = string.Join(FILE_SPLITER, targetFiles) + $"{FILE_SPLITER}{FILE_SPLITER}";
```

上述代码的 `FILE_SPLITER` 和 `pm` 都是从 [C#中使用SHFileOperation调用Windows的复制文件对话框](https://www.cnblogs.com/jifengg/archive/2013/05/13/3076131.html ) 博客里面抄的

所有代码如下

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;

namespace KachelearnemChurjawenikall;
internal class Program
{
    [SkipLocalsInit]
    static void Main(string[] args)
    {
        var sourceFile = @"F:\temp\1";
        var targetFile = @"F:\temp\2";

        if (!File.Exists(sourceFile))
        {
            var buffer = new byte[1024 * 1024];
            for (int i = 0; i < 1000; i++)
            {
                Random.Shared.NextBytes(buffer);
                File.AppendAllBytes(sourceFile, buffer);
            }
        }

        string[] sourceFiles = [sourceFile];
        string[] targetFiles = [targetFile];

        SHFILEOPSTRUCT pm = new SHFILEOPSTRUCT();
        pm.wFunc = wFunc.FO_COPY;

        //设置对话框标题，在win7中无效
        pm.lpszProgressTitle = "复制文件";
        pm.pFrom = string.Join(FILE_SPLITER, sourceFiles) + $"{FILE_SPLITER}{FILE_SPLITER}";
        pm.pTo = string.Join(FILE_SPLITER, targetFiles) + $"{FILE_SPLITER}{FILE_SPLITER}";

        pm.fFlags = FILEOP_FLAGS.FOF_NOCONFIRMATION | FILEOP_FLAGS.FOF_MULTIDESTFILES | FILEOP_FLAGS.FOF_ALLOWUNDO;

        SHFileOperation(pm);
    }

    /// <summary>

    /// 映射API方法

    /// </summary>

    /// <param name="lpFileOp"></param>

    /// <returns></returns>

    [DllImport("shell32.dll", SetLastError = true, CharSet = CharSet.Unicode)]

    private static extern int SHFileOperation(SHFILEOPSTRUCT lpFileOp);



    /// <summary>

    /// 多个文件路径的分隔符

    /// </summary>

    private const string FILE_SPLITER = "\0";



    /// <summary>

    /// Shell文件操作数据类型

    /// </summary>

    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]

    private class SHFILEOPSTRUCT

    {

        public IntPtr hwnd;

        /// <summary>

        /// 设置操作方式

        /// </summary>

        public wFunc wFunc;

        /// <summary>

        /// 源文件路径

        /// </summary>

        public string pFrom;

        /// <summary>

        /// 目标文件路径

        /// </summary>

        public string pTo;

        /// <summary>

        /// 允许恢复

        /// </summary>

        public FILEOP_FLAGS fFlags;

        /// <summary>

        /// 监测有无中止

        /// </summary>

        public bool fAnyOperationsAborted;

        public IntPtr hNameMappings;

        /// <summary>

        /// 设置标题

        /// </summary>

        public string lpszProgressTitle;

    }



    /// <summary>

    /// 文件操作方式

    /// </summary>

    private enum wFunc

    {

        /// <summary>

        /// 移动

        /// </summary>

        FO_MOVE = 0x0001,

        /// <summary>

        /// 复制

        /// </summary>

        FO_COPY = 0x0002,

        /// <summary>

        /// 删除

        /// </summary>

        FO_DELETE = 0x0003,

        /// <summary>

        /// 重命名

        /// </summary>

        FO_RENAME = 0x0004

    }


    /// <summary>
    /// fFlags枚举值，
    /// 参见：http://msdn.microsoft.com/zh-cn/library/bb759795(v=vs.85).aspx
    /// </summary>
    private enum FILEOP_FLAGS
    {
        ///<summary>
        ///pTo 指定了多个目标文件，而不是单个目录
        ///The pTo member specifies multiple destination files (one for each source file) rather than one directory where all source files are to be deposited.
        ///</summary>
        FOF_MULTIDESTFILES = 0x1,

        ///<summary>
        ///不再使用
        ///Not currently used.
        ///</summary>
        FOF_CONFIRMMOUSE = 0x2,

        ///<summary>
        ///不显示一个进度对话框
        ///Do not display a progress dialog box.
        ///</summary>
        FOF_SILENT = 0x4,

        ///<summary>
        ///碰到有抵触的名字时，自动分配前缀
        ///Give the file being operated on a new name in a move, copy, or rename operation if a file with the target name already exists.
        ///</summary>
        FOF_RENAMEONCOLLISION = 0x8,

        ///<summary>
        ///不对用户显示提示
        ///Respond with "Yes to All" for any dialog box that is displayed.
        ///</summary>

        FOF_NOCONFIRMATION = 0x10,

        ///<summary>
        ///填充 hNameMappings 字段，必须使用 SHFreeNameMappings 释放
        ///If FOF_RENAMEONCOLLISION is specified and any files were renamed, assign a name mapping object containing their old and new names to the hNameMappings member.
        ///</summary>
        FOF_WANTMAPPINGHANDLE = 0x20,

        ///<summary>
        ///允许撤销
        ///Preserve Undo information, if possible. If pFrom does not contain fully qualified path and file names, this flag is ignored.
        ///</summary>
        FOF_ALLOWUNDO = 0x40,

        ///<summary>
        ///使用 *.* 时, 只对文件操作
        ///Perform the operation on files only if a wildcard file name (*.*) is specified.
        ///</summary>
        FOF_FILESONLY = 0x80,

        ///<summary>
        ///简单进度条，意味着不显示文件名。
        ///Display a progress dialog box but do not show the file names.
        ///</summary>
        FOF_SIMPLEPROGRESS = 0x100,

        ///<summary>
        ///建新目录时不需要用户确定
        ///Do not confirm the creation of a new directory if the operation requires one to be created.
        ///</summary>
        FOF_NOCONFIRMMKDIR = 0x200,

        ///<summary>
        ///不显示出错用户界面
        ///Do not display a user interface if an error occurs.
        ///</summary>
        FOF_NOERRORUI = 0x400,

        ///<summary>
        /// 不复制 NT 文件的安全属性
        ///Do not copy the security attributes of the file.
        ///</summary>
        FOF_NOCOPYSECURITYATTRIBS = 0x800,

        ///<summary>
        /// 不递归目录
        ///Only operate in the local directory. Don't operate recursively into subdirectories.
        ///</summary>
        FOF_NORECURSION = 0x1000,

        ///<summary>
        ///Do not move connected files as a group. Only move the specified files.
        ///</summary>
        FOF_NO_CONNECTED_ELEMENTS = 0x2000,

        ///<summary>
        ///Send a warning if a file is being destroyed during a delete operation rather than recycled. This flag partially overrides FOF_NOCONFIRMATION.
        ///</summary>
        FOF_WANTNUKEWARNING = 0x4000,

        ///<summary>
        ///Treat reparse points as objects, not containers.
        ///</summary>
        FOF_NORECURSEREPARSE = 0x8000,
    }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9685aaf325b6d7deee15fd6488cd7533f7052ea2/Workbench/KachelearnemChurjawenikall) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9685aaf325b6d7deee15fd6488cd7533f7052ea2/Workbench/KachelearnemChurjawenikall) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9685aaf325b6d7deee15fd6488cd7533f7052ea2
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9685aaf325b6d7deee15fd6488cd7533f7052ea2
```

获取代码之后，进入 Workbench/KachelearnemChurjawenikall 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
