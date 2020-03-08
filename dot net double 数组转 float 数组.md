# dot net double 数组转 float 数组

本文告诉大家如果遇到 double 数组转 float 数组千万不要使用 Cast ，一般都使用 select 强转。

<!--more-->
<!-- CreateTime:2018/12/25 9:27:46 -->
 

<!-- 标签：C#,dotnet -->

最近在开发[Avalonia](https://github.com/AvaloniaUI/Avalonia ) ，有大神告诉我，下面的代码可以这样写

```csharp
dashes = pen.DashStyle.Dashes.Select(x => (float)x).ToArray();

```

修改为

```csharp
dashes = pen.DashStyle.Dashes.Cast<float>.ToArray()
```

[Improve tiny performance](https://github.com/AvaloniaUI/Avalonia/pull/1472 )

但是实际上不能这样写，因为 cast 无法转换 float 和 double 因为不存在一个类同时继承 float 和 double ，所以如果使用这个方法转换，就无法运行

```csharp
System.InvalidCastException:“Unable to cast object of type 'System.Double' to type 'System.Single'.”
```

所以建议的方法是使用 select ，在里面强转。

尝试运行下面代码

```csharp
            List<double> titHruxvrvaa = new List<double>()
            {
                1d,
                2d,
                3d
            };

            var traStqjq = titHruxvrvaa.Cast<float>().ToArray();//System.InvalidCastException:“Unable to cast object of type 'System.Double' to type 'System.Single'.”

            foreach (var temp in traStqjq)
            {
                Console.WriteLine(temp);
            }
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
