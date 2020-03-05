# dotnet 方法名 To 和 As 有什么不同

在看到 dotnet 框架里面有很多方法里面用了 ToXx 和 AsXx 好像都是从某个类转换为另一个类，那么这两个方法命名有什么不同

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在约定的方法命名里面，用 To 的方法表示从类 A 转为类 B 同时这两个类将没有任何关联，也就是对类 B 做的内容不会影响到原有的类 A 例如 ToString 方法

```csharp
            var str = new StringBuilder();
            var foo = str.ToString();
```

上面代码的 str 在调用 ToString 方法之后，返回值将和原来的 StringBuilder 没有关系

而在用 As 的方法表示转换类之后，转换的类和原有的类有关联，例如 List 的 AsReadOnly 方法

```csharp
            var foo = Enumerable.Range(0,100).ToList();
            var readOnlyCollection = foo.AsReadOnly();
            Console.WriteLine(readOnlyCollection.Count); // 100
            foo.RemoveAt(0);
            Console.WriteLine(readOnlyCollection.Count); // 99
```

虽然调用 AsReadOnly 返回了 ReadOnlyCollection 类型，但是原有的 foo 和 readOnlyCollection 是有关联的，对 foo 的修改将会影响转换类的值如上面代码，将 foo 移除了第一个之后，相应的值也会修改

在方法命名里面用 To 开始的表示转换类，同时转换的类和原有的没有关联，而使用 As 开始的表示从观测角度可以作为另一个类观测，转换的类和原有的存在关联

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
