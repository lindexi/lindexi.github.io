
# dotnet 运行时获取某类型的对象占用内存大小

本文将告诉大家一个黑科技方法在运行时动态获取对象本身占用空间，不包括对象引用的其他对象的空间大小的方法

<!--more-->


<!-- CreateTime:2021/4/27 21:15:57 -->

<!-- 发布 -->

此方法是在开源仓库 [sidristij/dotnetex: Gets size of .Net Framework objects, can change type of object to incompatible and can alloc .Net objects at unmanaged memory area](https://github.com/sidristij/dotnetex ) 找到的方法

通过不安全代码和反射获取对象类型的 MethodTableInfo 即可在 MethodTableInfo 里面读取 Size 属性，关于 MethodTableInfo 的定义如下

```csharp
        /// <summary>
        /// Description of GCEnumerator.
        /// </summary>
        [StructLayout(LayoutKind.Explicit)]
        public unsafe struct MethodTableInfo
        {
            #region Basic Type Info

            [FieldOffset(0)]
            public MethodTableFlags Flags;

            [FieldOffset(4)]
            public int Size;

            [FieldOffset(8)]
            public short AdditionalFlags;

            [FieldOffset(10)]
            public short MethodsCount;

            [FieldOffset(12)]
            public short VirtMethodsCount;

            [FieldOffset(14)]
            public short InterfacesCount;

            [FieldOffset(16)]
            public MethodTableInfo* ParentTable;

            #endregion

            [FieldOffset(20)]
            public ObjectTypeInfo* ModuleInfo;

            [FieldOffset(24)]
            public ObjectTypeInfo* EEClass;
        }
```

以上代码关键属性是 Size 属性，通过 Size 属性可以拿到运行时的对象占用空间大小。在 CLR 里面默认将会做内存的对齐，因此对象占用空间大小将会大于等于字段占用空间大小的总数

获取某类型对象占用空间大小的方法如下

```csharp
        /// <summary>
        /// Gets type size in unmanaged memory (directly in SOH/LOH) by type
        /// </summary>
        public static unsafe Int32 SizeOf<T>()
        {
            return ((MethodTableInfo*)(typeof(T).TypeHandle.Value.ToPointer()))->Size;
        }
```

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b4784765aae3a9ea35547fff620305966c750c05/HaynogelwhaiFaycemferlerluja) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b4784765aae3a9ea35547fff620305966c750c05/HaynogelwhaiFaycemferlerluja) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。