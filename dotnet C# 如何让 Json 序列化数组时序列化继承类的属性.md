# dotnet C# 如何让 Json 序列化数组时序列化继承类的属性

如果我使用的是具体的数组而我的数组是基类数组，而我传入子类的元素进行 json 序列化，可能发现 Json.NET 序列化没有包含子类元素的属性。如果要包含子类的属性或字段，可以在序列化的类数组定义为 object 数组的方式

<!--more-->
<!-- CreateTime:2020/2/21 19:05:02 -->

<!-- 发布 -->

我在用 WPF 写一个复杂的应用，我需要 ASP.NET Core 后台传输一个 AppData 类的数组，包含的属性如下

```csharp
public class Lindexi
{
	public string Name { set; get; }
}
```

然后我有 Foo 类继承 Lindexi 类


```csharp
public class Foo : Lindexi
{
	public string F1 { set; get; }
}
```

用下面代码序列化

```csharp
        static void Main(string[] args)
        {
            Console.WriteLine(ToString(new Foo()
            {
                F1 = "林德熙是逗比"
            }));
        }

        static string ToString(Lindexi lindexi)
        {
            return JsonSerializer.Serialize(new [] { lindexi });
        }
```

运行可以看到输出

```json
[{"Name":null}]
```

也就是 Foo 的属性被丢失了，在 .NET Core 3.0 可以使用 System.Text.Json 命名空间而不需要用 Newtonsoft.Json 库

此时解决方法是将数组定义为 object 数组

```csharp
        static string ToString(Lindexi lindexi)
        {
            return JsonSerializer.Serialize(new object[] { lindexi });
        }
```

刚才定义的属性都是首字符大写的，转换为首字符小写的可以添加配置 PropertyNamingPolicy 请看下面

```csharp
        static string ToString(Lindexi lindexi)
        {
            return JsonSerializer.Serialize(new object[] { lindexi }, new JsonSerializerOptions()
            {
                PropertyNamingPolicy = JsonNamingPolicy.CamelCase
            });
        }
```

另外作为 json 上传到后台需要注意添加 "application/json" 请看代码

```csharp
            var stringContent = new StringContent(json, Encoding.UTF8, "application/json");
```

[win10 uwp 客户端如何发送类到 asp dotnet core 作为参数](https://blog.lindexi.com/post/win10-uwp-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%A6%82%E4%BD%95%E5%8F%91%E9%80%81%E7%B1%BB%E5%88%B0-asp-dotnet-core-%E4%BD%9C%E4%B8%BA%E5%8F%82%E6%95%B0.html )

如果你是被这个问题坑到的，同时一开始没有找到本文解决，请告诉我你之前搜的关键字，我去优化这个博客内容

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
