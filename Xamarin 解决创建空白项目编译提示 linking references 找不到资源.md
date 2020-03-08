# Xamarin 解决创建空白项目编译提示 linking references 找不到资源

在新建一个 Xamarin 应用，编译时提示了 linking references 和一些诡异的文件无法访问，原因是文件路径问题和 NuGet 还原问题

<!--more-->
<!-- CreateTime:2019/12/24 9:28:30 -->

<!-- 发布 -->

如果创建的 Xamarin 项目所在的文件路径比较深，同时包含中文等字符时，在编译时可以会诡异失败

```csharp
oid\Xamarin.Android.Aapt2.targets(155,3): error APT2062: failed linking references.
```

或者

```csharp
error APT0000: resource integer/google_play_services_version (aka com.softoursistemas.rutesc.droid:integer/google_play_services_version) not found
```

也有比较明确的提示

```csharp
Failed to create JavaTypeInfo for class: Android.Support.V13.View.Inputmethod.InputConnectionCompat/IOnCommitContentListenerImplementor due to MAX_PATH: System.IO.DirectoryNotFoundException: 未能找到路径“g:\lindexi\ChuwheaweaharfelkalBelqearjurnawnere\ChuwheaweaharfelkalBelqearjurnawnere\ChuwheaweaharfelkalBelqearjurnawnere.Android\obj\Debug\90\android\src\mono\android\support\v13\view\inputmethod\InputConnectionCompat_OnCommitContentListenerImplementor.java”的一部分。
   在 System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
   在 System.IO.File.InternalDelete(String path, Boolean checkHost)
   在 System.IO.File.Delete(String path)
   在 Xamarin.Android.Tools.Files.CopyIfStreamChanged(Stream stream, String destination)
   在 Xamarin.Android.Tasks.Generator.CreateJavaSources(TaskLoggingHelper log, IEnumerable`1 javaTypes, String outputPath, String applicationJavaClass, String androidSdkPlatform, Boolean useSharedRuntime, Boolean generateOnCreateOverrides, Boolean hasExportReference)	ChuwheaweaharfelkalBelqearjurnawnere.Android	
```

也有很诡异的提示

```csharp
Android\obj\Debug\90\lp\39\jl\res : error APT2097: failed to open directory: 绯荤粺
```

有明确的提示的可以看到，其实 Xamarin 的设计有点坑，就是路径太长了，加上了我的项目命名，在 Windows 下的路径就超过了 260 字符限制，此时就编译不通过

解决方法是尝试将项目移动到磁盘的第二级文件夹，然后使用删除所有的 `bin` 和 `obj` 文件夹，此时就可以解决这几个诡异的坑

[Failed linking references in Android project (Xamarin - Visual Studio Community 2019) · Issue #3822 · xamarin/xamarin-android](https://github.com/xamarin/xamarin-android/issues/3822)

[Create a new Mobile.App (Xamarin.Forms) project, try to compile, but "Failed linking references" - Stack Overflow](https://stackoverflow.com/questions/57592847/create-a-new-mobile-app-xamarin-forms-project-try-to-compile-but-failed-lin)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
