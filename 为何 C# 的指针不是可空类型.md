# 为何 C# 的指针不是可空类型

在 C# 里面的指针实际上使用 int32 或 int64 存储，在 C# 里面的指针需要开启不安全代码才能使用，这里的指针是一个结构体，而结构体是存在值的

<!--more-->
<!-- CreateTime:4/7/2020 8:53:00 AM -->



我尝试写出 `byte*? foo` 的时候，构建的时候 VS 提示下面代码

```csharp
// Error CS1519: Invalid token '?' in class, struct, or interface member declaration
```

原因是 `byte*` 实际上等价一个 int32 或 int64 的结构体，看了下面代码就知道

```csharp
byte* foo = null;
// 和下面代码是等价的
byte* foo = (byte*)0;
```

也就是此时的 `byte*?` 是不对的

如果要使用可空，可以使用 `IntPtr?` 代替，但是作用不大

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
