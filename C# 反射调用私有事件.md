# C# 反射调用私有事件

在 C# 反射调用私有事件经常会不知道如何写，本文告诉大家如何调用

<!--more-->
<!-- CreateTime:2019/11/29 8:51:13 -->


<!-- 标签：C#，反射 -->

假设有 A 类的代码定义了一个私有的事件

```csharp
    class A
    {
        private event EventHandler Fx
        {
            add { }
            remove { }
        }
    }
```

通过反射可以拿到 A 的事件 Fx 但是无法直接添加事件

```csharp
            var eventInfo = typeof(A).GetEvent("Fx", BindingFlags.Instance | BindingFlags.NonPublic);
```

如果这时直接调用 AddEventHandler 就会出现下面异常

```csharp
            var eventInfo = typeof(A).GetEvent("Fx", BindingFlags.Instance | BindingFlags.NonPublic);

            var a = new A();

            eventInfo.AddEventHandler(a, new EventHandler(Fx));

            void Fx(object sender, EventArgs e)
            {
            }
```

```csharp
System.InvalidOperationException:“由于不存在此事件的公共添加方法，因此无法添加该事件处理程序。”
```

解决的方法是调用 GetAddMethod 的方法请看下面

```csharp
            var eventInfo = typeof(A).GetEvent("Fx", BindingFlags.Instance | BindingFlags.NonPublic);
            var addFx = eventInfo.GetAddMethod(true);
            var removeFx = eventInfo.GetRemoveMethod(true);

            var a = new A();

            addFx.Invoke(a, new[] {new EventHandler(Fx)});
            removeFx.Invoke(a, new[] {new EventHandler(Fx)});

            void Fx(object sender, EventArgs e)
            {
            }
```

参见 https://stackoverflow.com/a/6423886/6116637

如果可能遇到类型转换的异常`System.ArgumanetException:'Object of type 'System.EventHandler1[System.EventArgs]' cannot be converted to type 'System.EventHandler'. `，请看[.NET/C# 使用反射注册事件 - walterlv](https://walterlv.com/post/add-event-handler-using-reflection.html )

更多反射请看

[win10 uwp 反射](https://blog.lindexi.com/post/win10-uwp-%E5%8F%8D%E5%B0%84.html )

[.NET Core/Framework 创建委托以大幅度提高反射调用的性能 - walterlv](https://walterlv.com/post/create-delegate-to-improve-reflection-performance.html )

[设置 .NET Native 运行时指令以支持反射（尤其适用于 UWP） - walterlv](https://walterlv.com/uwp/2017/09/21/reflection-using-dotnet-native-runtime-directive.html )

[.NET/C# 使用反射调用含 ref 或 out 参数的方法 - walterlv](https://walterlv.com/post/handle-ref-or-out-arguments-using-reflection.html )

[.NET/C# 推荐一个我设计的缓存类型（适合缓存反射等耗性能的操作，附用法） - walterlv](https://walterlv.com/post/design-a-cache-pool.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
