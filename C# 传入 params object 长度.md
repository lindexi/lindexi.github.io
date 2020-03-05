# C# 传入 params object 长度

刚刚 [LiesAuer](www.liesauer.net) 大神问了一个问题，如果在 params object 传入 object 数组，那么拿到的值是的长度是多少

我做了测试在传入不同的值可能拿到不同的长度

<!--more-->
<!-- CreateTime:2018/9/30 18:33:20 -->


先来说总结

 - 传入一个数组进入 `params object[]` 会解析数组，传入的数组有多少元素就解析出多少元素

 - 传入一个数组和另一个元素，就会将数组解析为一个元素

 - 传入一个数组强转为 object ，就会解析为传入一个元素

先来创建一个控制台项目，在项目添加一个空白的方法，这个方法输出参数长度

```csharp
        private static void TracinozeSallraymear(params object[] jearsawelSekerdis)
        {
            Console.WriteLine(jearsawelSekerdis.Length);
        }
```

尝试传入不同的值来测试长度

```csharp
            var foo = new object[3] {"lindexi", "123", "csdn"};

            TracinozeSallraymear(foo); 
```

这里可以说传入的是 一个 数组，也就是参数是 1 ，也可以说是传入 3 个元素，我尝试运行了一下，输出的是 3 也就是拿到 3 个元素。

我又尝试使用 string 数组，请看下面

```csharp
            TracinozeSallraymear(new string[]
            {
                "lindexi", "123", "csdn"
            }); 
```

上面的代码也是拿到 3 个元素。

但是如何将数组转为 object 就会输出 1 个元素

```csharp
            TracinozeSallraymear((object) foo); 
            //输出 1 个元素
```	

如果我在一个数组和一个元素放在一起，那么就会自动将数组转换为元素，请看下面代码

```csharp

            TracinozeSallraymear(new string[]
            {
                "lindexi", "123", "csdn"
            }, "lindexi.gitee.io"); 
            //输出 2 个元素
            // 一个是字符串数组，一个是字符串
```

如果我不使用数组，我使用 List 呢？

```csharp
            TracinozeSallraymear(new List<string>
            {
                "lindexi",
                "123",
                "csdn"
            }); 
            //输出 1 个元素
```

所以需要将一个数组传入`params object[]`作为一个元素，简单的方法是将数组强转 `object` 或者再添加一个参数

参见：
[c# - How to pass a single object[] to a params object[] - Stack Overflow](https://stackoverflow.com/questions/36350/how-to-pass-a-single-object-to-a-params-object )

![](https://i.loli.net/2018/08/20/5b7aa70fd1f1a.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
