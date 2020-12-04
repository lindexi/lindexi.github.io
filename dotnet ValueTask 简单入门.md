# dotnet ValueTask 简单入门

通过 ValueTask 可以用来提升软件整体的性能，使用方法也非常简单，本文将带大家简单的入门使用这个 ValueTask 的功能

<!--more-->
<!-- CreateTime:2020/11/20 9:25:04 -->



为什么使用 ValueTask 可以用来提升软件整体的性能？回答这个文件的博客有很多，请看以下这几篇博客

- [深入理解 ValueTask - 沉睡的木木夕 - 博客园](https://www.cnblogs.com/ms27946/p/understanding-task-valuetask.html )
- [如何使用 C# 中的 ValueTask - 技术译民 - 博客园](https://www.cnblogs.com/ittranslator/p/13703279.html )

简单的使用方法就是在方法的返回值里面，将 Task 替换为 ValueTask 就可以了

如以下代码

```csharp
    public async Task<T> GetDataAsync()
    {
        var value = default(T);
        await Task.Delay(100);
        return value;
    }
```

使用 ValueTask 时，只需要更改返回值为 Task 就可以了

```csharp
    public async ValueTask<T> GetDataAsync()
    {
        var value = default(T);
        await Task.Delay(100);
        return value;
    }
```

这样就完成了，十分简单

但是使用的时候依然是有限制的

只要你确定能遵循以下简单的规则，那使用 ValueTask 替换是安全的，也是可以提升性能的

规则是使用对应的方法是通过 `await` 读取返回值或者等待执行完成，如以下代码

```csharp
var foo = await GetDataAsync();
```

而如果是期望获取整个任务本身，只是就 **不能** 使用 ValueTask 代替（强行代替也可以，只是坑多，请确定你了解实现细节） 如以下代码

```csharp
var task = GetDataAsync(); // 不要使用 ValueTask 返回值
```

不要尝试转换为同步获取返回值，如以下代码

```csharp
var foo = await GetDataAsync().GetAwaiter().GetResult(); // 切记不要这样写，如果需要这样写就不适合使用 ValueTask 代替
```

当然了，本文只是简单的入门的博客，很多细节都没有说到。但只要遵循本文告诉大家的规则，此时的使用将会是安全的，而其他的情况，还请小心。另外，在考虑使用 ValueTask 之前，还是需要进行一定的性能分析。在使用之后，依然需要做一些测试

在 .NET Framework 里面暂时还没有加入 ValueTask 的支持，因为 ValueTask 需要最低是 .NET Standard 2.1 才能支持，因此需要加上一点兼容的代码。如下面代码，放在文件的开始，就能支持。但下面代码本质上就是使用 Task 而已，因此不会带来任何的优化

```csharp
#if !NETCOREAPP
using ValueTask = System.Threading.Tasks.Task;
#endif
```

想要在 dotnet framework 里面使用 ValueTask 同时有真的提升，可以使用官方给兼容包 [System.Threading.Tasks.Extensions](https://www.nuget.org/packages/System.Threading.Tasks.Extensions ) 来实现，这个 NuGet 包可以用来兼容旧版本，如 .NET Framework 4.5 等，让这些能支持 ValueTask 的功能。感谢 Sagilio 的提醒

官方文档请看[Understanding the Whys, Whats, and Whens of ValueTask](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
