# C# 遍历枚举

本文告诉大家如何遍历枚举

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


遍历枚举是很简单，请看下面代码

```csharp
            StringBuilder sdqsuhDboyowb=new StringBuilder();

            foreach (var temp in Enum.GetNames(typeof(MethodAttributes)))
            {
                sdqsuhDboyowb.Append(temp + "\r\n");
            }
```

使用 Enum.GetNames ，参数是枚举的类型就可以遍历

但是这个方法的性能比较差，可以使用一个库。首先打开 Nuget 安装 Enums.NET

然后使用下面的代码遍历

```csharp
           foreach (var temp in Enums.GetNames<MethodAttributes>())
            {
                sdqsuhDboyowb.Append(temp + "\r\n");
            }
```

实际上就是使用 Enums.GetNames 传入枚举类型

如果需要获得每个的值，可以使用下面方法

```csharp
            foreach (var temp in Enums.GetMembers<MethodAttributes>())
            {
                sdqsuhDboyowb.Append(temp.Name + " " + temp.ToInt32() + "\r\n");
            }
```

下面是对比性能，官方的，但是我没有自己去运行

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F2018313204944.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
