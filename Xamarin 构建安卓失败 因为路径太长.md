# Xamarin 构建安卓失败 因为路径太长

如果将自己的应用放的路径比较深，那么构建安卓的时候可能因为路径超过长度失败

<!--more-->
<!-- CreateTime:2020/2/29 18:55:44 -->

<!-- 发布 -->

如果在你构建 Xamarin 安卓的时候看到如以下提示，那么请你确定以下你的应用程序路径是不是超过 250 个字符了

```csharp
obj\Debug\90\android\src\mono\android\support\v4\view\accessibility\AccessibilityManagerCompat_AccessibilityStateChangeListenerImplementor.java
```

解决方法是减少项目名的长度，移动到距离根文件夹比较近的文件夹，也就是让上面这个文件的总路径不要太长

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
