
# dotnet 给任意对象附加任意属性的库

在使用 dotnet 的时候，有时候会期望某个类多添加一个属性，但是这个类可能是放在引用库里面不能直接修改，或者添加一个属性会影响这个类的设计。那么有没有方法和 WPF 一样支持给任意对象附加任意的属性？本文告诉大家一个好用的库，用来给任意的对象附加任意的属性

<!--more-->


<!-- CreateTime:5/28/2020 8:16:49 PM -->

<!-- 发布 -->

要解决的问题是什么？

- 有一些参数传进来的对象，期望给这些对象添加很业务的属性
- 某些业务期望内部使用某个对象的某个属性，但是不期望在此业务之外这个属性被使用
- 期望给某个类添加额外的属性，但是改不动这个类

本文使用的库是一个在 GitHub 开源的 [dotnetCampus.ClrAttachedProperty](https://github.com/dotnet-campus/dotnetCampus.ClrAttachedProperty ) 库

这个库提供了两个不同的 NuGet 包，其中一个包是传统的 Dll 引用包。另一个包是使用 [SourceYard](https://github.com/dotnet-campus/SourceYard) 打出来的源代码包，源代码包安装之后将会引用源代码

安装传统的 Dll 引用包的方式如下

```
dotnet add package dotnetCampus.ClrAttachedProperty --version 1.0.0
```

安装源代码包的方式如下

```
dotnet add package dotnetCampus.ClrAttachedProperty.Source --version 1.0.0
```

在使用的时候两个包只需要选其中一个就可以

安装完成之后就可以给任意的对象附加任意的属性，请看代码

```csharp
using dotnetCampus.ClrAttachedProperty;

        public void Foo(object obj, string propertyName, object objectValue)
        {
            obj.SetAttachedProperty(propertyName, objectValue);

            var value = obj.GetAttachedProperty(propertyName);
        }
```

上面代码就是包含给某个对象设置额外的属性，然后获取额外的属性的代码。换句话说，只要属性名不相同，那么添加或获取的是不同的属性

也就是可以做到保密属性名，此时在业务之外就拿不到这个属性

这个附加属性的原理是使用 ConditionalWeakTable 实现在对象被回收的时候自动回收额外的属性，详细请看 

- [.NET/C# 使用 ConditionalWeakTable 附加字段（CLR 版本的附加属性，也可用用来当作弱引用字典 WeakDictionary） - walterlv](https://blog.walterlv.com/post/conditional-weak-table.html )

- [dotnet ConditionalWeakTable 的底层原理](https://blog.lindexi.com/post/dotnet-ConditionalWeakTable-%E7%9A%84%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。