# dotnet 列表 Linq 的 Take 用法

在 dotnet 可以使用 Take 获取指定数量的元素，获取顺序是从前向后，而获取到的数量是小于等于传入的指定数量。如数组中元素的数量小于传入的指定数量，则返回数组中的所有元素。如果数组中元素的数量大于等于传入的数量，则按照数组或列表顺序返回指定数量的元素

<!--more-->
<!-- CreateTime:2019/10/24 9:04:23 -->

<!-- csdn -->

在使用 Take 方法之前，请引用命名空间

```csharp
using System.Linq;
```

获取的时候通过在枚举类添加 Take 方法传入获取数量就可以返回小于或等于指定数量的元素

```csharp
            var list = new List<int>();
            for (int i = 0; i < 10; i++)
            {
                list.Add(i);
            }

            foreach (var temp in list.Take(100))
            {
                Console.WriteLine(temp);
            }
```

如上面代码，传入的获取数量是100而数组里面只有10个元素，那么将返回10个元素

一些细节如下：

如果传入的 Count 值小于等于 0 那么将会返回空列表

因为用的是延迟加载，所以没有枚举是不会执行逻辑

如果是列表将会返回 ListPartition 实例，但这是一个内部类 [ListPartition](https://github.com/dotnet/corefx/blob/fc89c884e99ef3fd920dbe75fbbaf797b02a944f/src/System.Linq/src/System/Linq/Partition.SpeedOpt.cs#L155)

更多请看 [Enumerable.Take](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.enumerable.take?view=netframework-4.8 ) 官方文档

源代码请看 [src/System.Linq/src/System/Linq/Take.cs](https://github.com/dotnet/corefx/blob/50fc80c8023060d61a826b01733a93840018fe92/src/System.Linq/src/System/Linq/Take.cs )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
