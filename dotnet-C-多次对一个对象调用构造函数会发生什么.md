
# dotnet C# 多次对一个对象调用构造函数会发生什么

今天来玩一点变态的，使用反射获取到某个类型的构造函数，接着多次对此类型的某个对象调用构造函数方法。请问此时会发生什么

<!--more-->


<!-- CreateTime:2021/9/7 19:44:40 -->

<!-- 博客 -->
<!-- 发布 -->

假定有一个类型 Foo 的定义如下

```csharp
        class Foo : IDisposable
        {
            public Foo()
            {
            }

            public int F1 { set; get; }
            public int F2 { set; get; } = 10;
        }
```

先使用 RuntimeHelpers 的 [GetUninitializedObject](https://docs.microsoft.com/zh-cn/dotnet/api/system.runtime.compilerservices.runtimehelpers.getuninitializedobject?WT.mc_id=WD-MVP-5003260) 方法创建对象而不调用构造函数

```csharp
        var foo = (Foo) RuntimeHelpers.GetUninitializedObject(typeof(Foo));
```

如果给 Foo 的构造函数添加断点，那么在运行上面代码的时候，可以看到断点是不会进入。详细请看 [dotnet C# 只创建对象不调用构造函数方法](https://blog.lindexi.com/post/dotnet-C-%E5%8F%AA%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E4%B8%8D%E8%B0%83%E7%94%A8%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0%E6%96%B9%E6%B3%95.html )

此时虽然 Foo 对象 foo 创建了，但是此对象还没有经过构造函数。接下来咱给此对象赋值，请看代码

```csharp
                var foo = (Foo)RuntimeHelpers.GetUninitializedObject(typeof(Foo));
                foo.F1 = 2;
                foo.F2 = 2;
```

请问此时的 Foo 里面的 F1 和 F2 属性分别是什么？当然就是 2 了

那如果用反射取出构造函数，对 foo 对象调用构造函数呢

```csharp
                var constructorInfo = typeof(Foo).GetConstructor(new Type[0]);
                constructorInfo!.Invoke(foo, null);
```

此时可以看到 foo 对象里面，两个属性的值不同。具体是啥？自己去本文末尾拉代码跑跑看

接着再给 foo 对象赋值，如下面代码

```csharp
                foo.F1 = 5;
                foo.F2 = 5;
```

然后再次调用构造函数，如下面代码

```csharp
                foo.F1 = 5;
                foo.F2 = 5;
                constructorInfo!.Invoke(foo, null);
```

请问此时的 F1 和 F2 属性的值是什么？

回顾一下基础知识，在类里面写的 `public int F2 { set; get; } = 10;` 代码其实是 C# 语言带来的功能，在构建的时候，会被转写为大概如下的构造函数代码

```csharp
            public Foo()
            {
                F2 = 10;
            }
```

通过 IL 代码可以看到实际的逻辑如下

```
    .method public hidebysig specialname rtspecialname instance void
      .ctor() cil managed
    {
      .maxstack 8

      // [53 43 - 53 45]
      IL_0000: ldarg.0      // this
      IL_0001: ldc.i4.s     10 // 0x0a
      IL_0003: stfld        int32 KicheyurcherNiwhiyuhawnelkeera.Program/Foo::'<F2>k__BackingField'

      // [48 13 - 48 25]
      IL_0008: ldarg.0      // this
      IL_0009: call         instance void [System.Runtime]System.Object::.ctor()
      IL_000e: nop

      // [49 13 - 49 14]
      IL_000f: nop

      // [50 13 - 50 14]
      IL_0010: ret

    } // end of method Foo::.ctor
```

在 C# 中，其实构造函数也是一个函数而已，如上面代码，只有写给 F2 赋值的逻辑，而没有给 F1 赋值的逻辑。因此在调用构造函数的时候，只会改变 F2 属性的值，而不会更改 F1 属性的任何值。也因为构造函数只是一个函数，因此调用多次就和调用一个方法多次是一样的



本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/5eb2ea112f2861791fafda9ed326657fd05572dd/KicheyurcherNiwhiyuhawnelkeera) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5eb2ea112f2861791fafda9ed326657fd05572dd/KicheyurcherNiwhiyuhawnelkeera) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 5eb2ea112f2861791fafda9ed326657fd05572dd
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 KicheyurcherNiwhiyuhawnelkeera 文件夹





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。