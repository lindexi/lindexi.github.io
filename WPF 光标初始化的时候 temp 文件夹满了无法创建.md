# WPF 光标初始化的时候 temp 文件夹满了无法创建

在 WPF 切换光标的时候，如果是通过本地资源的方法传入 stream 的，需要先复制到临时文件夹里面的文件，然后读取文件指针释放文件。如果此时的 temp 文件夹满了，那么复制文件的时候就无法继续了，于是就无法创建完成光标

<!--more-->
<!-- CreateTime:2019/5/16 19:16:27 -->

<!-- csdn -->

最近有老师找我说软件无法使用了，我尝试调试他的电脑，发现任何修改光标的代码就无法继续，因为无法创建光标

大概的修改光标的代码是这样写的，从解决方案里面放一个光标文件，设置为资源通过[访问解决方案文件](https://blog.lindexi.com/post/win10-uwp-%E8%AE%BF%E9%97%AE%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%96%87%E4%BB%B6.html ) 拿到资源

```csharp
var uri = new Uri("pack://application:,,,/Text.cur");
var resource = Application.GetResourceStream(uri);
Cursor = new Cursor(resource.Stream);
```

看到的堆栈如下

```
   在 System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
   在 System.IO.Directory.InternalCreateDirectory(String fullPath, String path, Object dirSecurityObj, Boolean checkHost)
   在 System.IO.Directory.InternalCreateDirectoryHelper(String path, Boolean checkHost)
   在 System.IO.Directory.CreateDirectory(String path)
   在 System.IO.FileHelper.CreateAndOpenTemporaryFile(String& filePath, FileAccess fileAccess, FileOptions fileOptions, String extension, String subFolder)
   在 System.Windows.Input.Cursor.LoadFromStream(Stream cursorStream)
   在 System.Windows.Input.Cursor..ctor(Stream cursorStream, Boolean scaleWithDpi)
   在 System.Windows.Input.Cursor..ctor(Stream cursorStream)
   在 FawlalnejajerelaWhallgemcurkear.MainWindow..ctor() 位置 D:\lindexi\程序\FawlalnejajerelaWhallgemcurkear\FawlalnejajerelaWhallgemcurkear\MainWindow.xaml.cs:行号 32
```

通过读源代码，发现在 [LoadFromStream](https://referencesource.microsoft.com/#PresentationCore/Core/CSharp/System/Windows/Input/Cursor.cs,090cb505b6310a4e) 方法里面是这样写的

```csharp
        private void LoadFromStream(Stream cursorStream)
        {
            string filePath = null;
 
            try
            {
                // Generate a temporary file based on the memory stream.
                // 从 temp 文件夹创建一个文件
                using (FileStream fileStream = FileHelper.CreateAndOpenTemporaryFile(out filePath))
                {
                	// 复制到文件
                    cursorStream.CopyTo(fileStream);
                }
 
                // 从文件里面读取光标
                // create a cursor from the temp file
                _cursorHandle = UnsafeNativeMethods.LoadImageCursor(IntPtr.Zero, filePath,
                    NativeMethods.IMAGE_CURSOR,
                    0, 0,
                    NativeMethods.LR_DEFAULTCOLOR |
                    NativeMethods.LR_LOADFROMFILE |
                    (_scaleWithDpi? NativeMethods.LR_DEFAULTSIZE : 0x0000));
                if (_cursorHandle == null || _cursorHandle.IsInvalid)
                {
                     throw new ArgumentException(SR.Get(SRID.Cursor_InvalidStream));
                }
            }
            finally
            { 
            	// 尝试删除这个文件，因为光标已经读取了
                FileHelper.DeleteTemporaryFile(filePath);
            }
        }
```

在 FileHelper.CreateAndOpenTemporaryFile 将会读取到一个 temp 文件夹里面的文件，但是如果这个文件无法访问，那么将不能继续

在我的设备上是很难做到让 temp 文件夹无法访问的，但是可以通过[通过修改环境变量修改当前进程使用的系统 Temp 文件夹的路径](https://blog.walterlv.com/post/redirect-environment-temp-folder.html )设置一个无法访问的文件夹作为 temp 文件夹

做一个无法访问的文件夹很简单，只需要右击属性安装，去掉用户就可以了

运行代码就会发现提示对路径访问拒绝

```csharp
System.UnauthorizedAccessException:“对路径“D:\lindexi\无法访问文件夹\WPF”的访问被拒绝。”
```

可以的解决方法有两个

1. 通过环境变量修改 temp 文件夹作为程序自己内部的数据文件夹，这和 UWP 的相同，每个程序都可以有自己独立的 temp 文件夹，可以解决有一些逗比软件会更改整个 temp 文件夹或里面某些文件夹的访问权限或有逗比在 temp 文件夹写入了 65535 个文件让其他程序无法写入文件。从微软官方[文档](https://docs.microsoft.com/en-us/windows/desktop/api/fileapi/nf-fileapi-gettempfilenamea) 可以知道 temp 文件夹的文件限制。
1. 只对光标的修改将解决方案里面的文件修改为输出的文件，此时将会调用 LoadFromFile 方法，这个方法是读取文件不需要复制文件，相对性能比较快

上面提供的两个方法，第一个方法除了解决光标的问题，还可以解决其他问题。第二个方法可以提升一点性能，同时两个方法可以一起使用

这个问题提交给微软，欢迎小伙伴点击 [Full temporary folder will crash cursor initialization](https://github.com/dotnet/wpf/issues/696 ) 帮我点赞

[通过修改环境变量修改当前进程使用的系统 Temp 文件夹的路径 - walterlv](https://blog.walterlv.com/post/redirect-environment-temp-folder.html )

[GetTempFileNameA function (fileapi.h)](https://docs.microsoft.com/en-us/windows/desktop/api/fileapi/nf-fileapi-gettempfilenamea?wt.mc_id=MVP )

[Path.GetTempFileName Method (System.IO)](https://docs.microsoft.com/en-us/dotnet/api/system.io.path.gettempfilename?wt.mc_id=MVP )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
