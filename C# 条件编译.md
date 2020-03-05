# C# 条件编译

有一部分代码只是用来调试使用，不期望在发布的时候执行。也有一些代码只是用来测试性能，也不期望在其他时候使用。在做源代码包的时候，我需要对不同的平台使用不同的代码。此时就可以用到条件编译符，在不同的条件下编译不同的代码

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


和 C++ 差不多，在 C# 里面也有宏的概念，只是在 C# 里面的专业名词是条件编译符

通过 `#if` `#else` 这些预处理器指令，可以指定使用不同的代码参加编译

用法是在 `#if` 后面跟上条件判断逻辑，只要条件判断逻辑返回 true 那么在 `#if` 包含的范围内的代码将会参加编译

在 `#if` 包含的范围内指的是在 `#if` 和下一个 `#else` 或 `#elif` 或 `#endif` 指令之间的范围，和普通的条件判断逻辑相同

```csharp
#if NET45

// 添加代码
// 如果定义了 NET45 这个宏，那么在这个范围内的代码将会参加编译

#elif NET46

// 如果没有定义 NET45 这个宏，那么将会进入这个分支的判断，如果定义了 NET46 这个宏那么在这个范围内的代码将会参加编译

#else 

// 在上面的判断都不成立的时候，在这个范围内的代码将会参加编译

#endif
```

上面是使用最长的判断代码，而更多的只是其中的组合，如 `#if xx` 和 `#endif` 的代码

例如我指定了在 DEBUG 模式，也就是调试模式下执行和发布模式不同的输出

```csharp
public void Foo()
{
#if DEBUG
    Console.WriteLine("Debug version");
#endif
}
```

在有定义 DEBUG 宏的编译时候，也就是一般在调试的时候，将会编译下面代码

```csharp
public void Foo()
{
    Console.WriteLine("Debug version");
}
```

而在没有定义 DEBUG 条件编译符的时候，将会编译下面的代码

```csharp
public void Foo()
{
    
}
```

可以注意到 `Console.WriteLine("Debug version");` 没有在没有定义 DEBUG 的时候参加编译，这段代码将被忽略

这样就是预处理器指令命名的原因，表示在编译之前做的指令

在进行判断是否进行编译的时候，支持使用复杂的条件判断，包括使用运算符 `==`（相等）和 `!=`（不相等）判断逻辑，在使用运算符的左边是对应的宏，右边是 true 和 false 两个值，其中 true 表示存在这个宏的定义，如下代码

```csharp
#if DEBUG == true
#endif
```

上面代码的 `#if DEBUG == true` 和 `#if DEBUG` 是等价写法

另外还支持运算符 `&&` (and) `||` (or) 和 `!` (not) 连接多个判断

此时的 `#if DEBUG == false` 和 `#if !DEBUG` 是等价判断

在使用连接符号的时候，支持添加 `==` 等判断运行符，也支持直接写条件编译符，如下代码

```csharp
#if NET45 || DEBUG == true
            // 在 NET45 定义或 DEBUG 定义的时候，这个范围内的代码可以执行
#endif
```

多个条件同时判断可以使用括号包含判断，请看代码

```csharp
#if NET46 || (DEBUG == true && NET47)
#endif
```

在定义了 NET46 或同时定义了 DEBUG 和 NET47 编译范围代码

更多预定义宏请看[dotnet 新项目格式与对应框架预定义的宏](https://blog.lindexi.com/post/dotnet-%E6%96%B0%E9%A1%B9%E7%9B%AE%E6%A0%BC%E5%BC%8F%E4%B8%8E%E5%AF%B9%E5%BA%94%E6%A1%86%E6%9E%B6%E9%A2%84%E5%AE%9A%E4%B9%89%E7%9A%84%E5%AE%8F.html )

[#if 预处理器指令](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
