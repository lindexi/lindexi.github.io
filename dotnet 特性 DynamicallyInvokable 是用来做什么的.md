# dotnet 特性 DynamicallyInvokable 是用来做什么的

我在 Linq 很多函数都看到 `__DynamicallyInvokable` 这个特性，这是一个没有官方文档的特性，也许是用来优化反射

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在[堆栈](https://stackoverflow.com/a/12552079/6116637) 网找到了以下描述

这个 `__DynamicallyInvokable` 特性是没有官方文档的，好像是在 .NET Framework 4.5 的一个优化添加的特性，这个特性看起来是在优化反射缓存的值，可以让随后的反射代码运行更快。从源代码里面的 [`System.Reflection.Assembly.cs`](https://referencesource.microsoft.com/#mscorlib/system/reflection/assembly.cs,1282) 文件可以看到以下描述

```csharp
 // 每个神奇的(blessed)的 API 都会添加 "__DynamicallyInvokableAttribute" 注释
 // Each blessed API will be annotated with a "__DynamicallyInvokableAttribute".
 // 这个 "__DynamicallyInvokableAttribute" 特性类是在他自己的程序集定义
 // This "__DynamicallyInvokableAttribute" is a type defined in its own assembly.
 // 所以他的构造函数总是一个 MethodDef 同时是 TypeDef 类型
 // So the ctor is always a MethodDef and the type a TypeDef.
 // 我们缓存此构造的 MethodDef 标记以便更快地进行自定义属性查找
 // We cache this ctor MethodDef token for faster custom attribute lookup.
 // 如果在程序集里面不包含这个特性，那么意味着这个程序集不存在任何神奇的(blessed)的 API 方法
 // If this attribute type doesn't exist in the assembly, it means the assembly
 // doesn't contain any blessed APIs.
 Type invocableAttribute = GetType("__DynamicallyInvokableAttribute", false);
 if (invocableAttribute != null)
 {
     Contract.Assert(((MetadataToken)invocableAttribute.MetadataToken).IsTypeDef);

     ConstructorInfo ctor = invocableAttribute.GetConstructor(Type.EmptyTypes);
     Contract.Assert(ctor != null);

     int token = ctor.MetadataToken;
     Contract.Assert(((MetadataToken)token).IsMethodDef);

     flags |= (ASSEMBLY_FLAGS)token & ASSEMBLY_FLAGS.ASSEMBLY_FLAGS_TOKEN_MASK;
 }
```

源代码请看 [Assembly.cs](https://referencesource.microsoft.com/#mscorlib/system/reflection/assembly.cs,1282)	

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
