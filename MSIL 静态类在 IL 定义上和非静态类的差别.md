# MSIL 静态类在 IL 定义上和非静态类的差别

本文来聊聊 MSIL 的基础知识，给一个 C# 的类标记了 static 之后和标记 static 之前，生成这个类的 IL 代码有什么不同

<!--more-->
<!-- CreateTime:2021/7/26 14:20:54 -->

<!-- 发布 -->

如以下的代码是一个默认的控制台程序

```csharp
    class Program
    {
        static void Main(string[] args)
        {
        }
    }
```

此时生成的 IL 代码，大概如下

```IL
.class private auto ansi beforefieldinit KakawbaijairKacheberelere.Program
        extends [System.Runtime]System.Object
```

而如果给 Program 加上静态，如以下代码，生成的 IL 代码是和之前不相同的

```csharp
    static class Program
    {
        static void Main(string[] args)
        {
        }
    }
```

生成的 IL 代码如下

```IL
.class private abstract sealed auto ansi beforefieldinit
  KakawbaijairKacheberelere.Program
    extends [System.Runtime]System.Object
```

复习一下 IL 代码的知识

在 MSIL 里，采用 `.class` 表示这是类型的定义，类型定义的格式大概如下

```csharp
.class [访问权限] [其他修饰] [命名空间].[类名] extends [继承的基类]
```

可以看到上下两个 IL 代码的不同在于，如果标记了 static 那 IL 将加上 `abstract sealed` 修饰。和 C# 代码的含义相同，通过 `abstract` 表示此类型不能被实例化，通过 `sealed` 表示此类型不能被继承。因此这就构成了静态类的特点，不能被创建实例，也不能被继承

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
