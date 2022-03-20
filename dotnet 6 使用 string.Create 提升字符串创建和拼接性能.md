# dotnet 6 使用 string.Create 提升字符串创建和拼接性能

本文告诉大家，在 dotnet 6 或更高版本的 dotnet 里，如何使用 string.Create 提升字符串创建和拼接的性能，减少拼接字符串时，需要额外申请的内存，从而减少内存回收压力

<!--more-->

<!-- 标签：dotnet，性能优化 -->
<!-- 发布 -->
<!-- 博客 -->

本文也是跟着 [Stephen Toub](https://github.com/stephentoub) 大佬学性能优化系列博客之一。这是 [Stephen Toub](https://github.com/stephentoub) 大佬在给 WPF 做的性能优化里面其中的一个小点。只是刚好这个优化点，是 [Stephen Toub](https://github.com/stephentoub) 大佬参与设计（预计是主导）和进行开发的。此优化点需要修改 Roslyn 内核，编写分析器，以及在 dotnet runtime 层进行支持才可以做到的优化。在过去完成了从 Roslyn 到分析器到 runtime 的支持之后，就到了应用框架层的支持了，这就是 [Stephen Toub](https://github.com/stephentoub) 大佬会在 WPF 仓库活跃的其中一个原因了

歪个楼，大家知道 dotnet 的各个层之间的关系吧。在 dotnet 里面，各个部分的角色是：

- Roslyn: 编译器内核层
- Runtime: 提供运行时的支持，广义的运行时，包括了执行引擎和基础库
- WPF: 应用代码框架层

在 WPF 上方就是业务代码逻辑了

在 WPF 仓库里 [Stephen Toub](https://github.com/stephentoub) 大佬的改动代码可以从 [Remove some unnecessary StringBuilders by stephentoub · Pull Request #6275 · dotnet/wpf](https://github.com/dotnet/wpf/pull/6275) 找到。这就是本文的例子代码了

在 dotnet 6 里面，新提供了 string.Create 方法的两个新重载方法，此两个重载方法签名分别如下

第一个重载方法：

```csharp
public static string Create (IFormatProvider? provider, Span<char> initialBuffer, ref System.Runtime.CompilerServices.DefaultInterpolatedStringHandler handler);
```

以上的三个参数的说明如下：

- provider: 一个提供区域性特定的格式设置信息的对象。
- initialBuffer: 初始缓冲区，可用作格式设置操作的一部分的临时空间。 此缓冲区的内容可能会被覆盖。
- handler: 通过引用传递的内插字符串。

第二个重载方法：

```csharp
public static string Create (IFormatProvider? provider, ref System.Runtime.CompilerServices.DefaultInterpolatedStringHandler handler);
```

第二个重载方法只是将第一个方法的 `Span<char> initialBuffer` 干掉而已

本文核心和大家聊的就是第一个重载方法

为什么这两个方法只有在 dotnet 6 或更高版本才能使用？为什么低版本的不能使用？如本文开始所说，这是因为这两个方法需要从 Roslyn 改到 dotnet runtime 才能支持。那为什么需要改那么多才能支持呢？因为这两个方法别看起来简单，实际上用到了 Roslyn 的黑科技。当然了用上了 Roslyn 黑科技，就可以让你告诉老师们，你的知识又需要更新了

敲黑板，第一个知识更新点是内插字符串。有趣的是在 C# 6.0 提出的内插字符串的知识点，刚好在 dotnet 6 的时候进行更新。别混了哦，这里说的 C# 版本和 dotnet 的版本可是两回事哦。如以下的内插字符串，你猜猜这是什么

```csharp
  $"lindexi is {doubi}"
```

在 dotnet 6 或更低的版本，你可以听从老师的话，说这是一个 `string.Format` 的语法优化而已，和以下的代码是完全等价的

```csharp
 string.Format("lindexi is {0}", doubi);
```

当然了，这么简单的代码我可没有开IDE来写，如果语法写错了，还请大家忽略吧

但是在 dotnet 6 或更高的版本，这些知识就需要更新了哈。看到了内插字符串，可不一定是 `string.Format` 的语法优化，还可以是 `System.Runtime.CompilerServices.DefaultInterpolatedStringHandler` 类型的创建哦

官方有一篇博客，嗯，又是 [Stephen Toub](https://github.com/stephentoub) 大佬写的，来告诉大家，这个 DefaultInterpolatedStringHandler 类型的来源以及是如何工作的，详细请看 [String Interpolation in C# 10 and .NET 6 - .NET Blog](https://devblogs.microsoft.com/dotnet/string-interpolation-in-c-10-and-net-6/)

简单来说就是使用内插字符串时，在 C# 10 和 dotnet 6 之前，将会额外创建一些对象，这些对象将会造成内存回收的压力。嗯，只是造成压力而已，不用担心，咱996都不怕。一点压力，没多少

如下面的代码，就是一个标准的内插字符串的用法

```csharp
public static string FormatVersion(int major, int minor, int build, int revision) =>
    $"{major}.{minor}.{build}.{revision}";
```

在 C# 10 和 dotnet 6 之前，经过了构建的代码，将会拆分以上的语法优化大概为如下代码

```csharp
public static string FormatVersion(int major, int minor, int build, int revision)
{
    var array = new object[4];
    array[0] = major;
    array[1] = minor;
    array[2] = build;
    array[3] = revision;
    return string.Format("{0}.{1}.{2}.{3}", array);
}
```

可以看到，其实这将需要额外多创建了一个 object 数组，同时在 `string.Format` 方法里面，还有很多其他的损耗

在 C# 10 和 dotnet 6 同时满足时，将在构建时，修改为如下结果等价的代码

```csharp
public static string FormatVersion(int major, int minor, int build, int revision)
{
    var handler = new DefaultInterpolatedStringHandler(literalLength: 3, formattedCount: 4);
    handler.AppendFormatted(major);
    handler.AppendLiteral(".");
    handler.AppendFormatted(minor);
    handler.AppendLiteral(".");
    handler.AppendFormatted(build);
    handler.AppendLiteral(".");
    handler.AppendFormatted(revision);
    return handler.ToStringAndClear();
}
```

这个 `DefaultInterpolatedStringHandler` 是一个结构体对象。根据一个完全不对的知识，结构体是在栈上分配的，以上的代码将除了返回的字符串之外，不会需要额外的内存申请。虽然知识完全是错的，不过结果是对的哈。辟谣时间：结构体可以是在栈上分配，也可以是在堆上分配的。对于大部分的局部变量创建的结构体来说，此结构体就是在栈上分配的。至少，以上的代码就是在栈上分配了一个 DefaultInterpolatedStringHandler 结构体对象。由于栈的内存是固定且明确的，可以认为用到 栈 上的内存就不属于额外申请的内存，再因为栈的空间，将会在方法执行完成之后，自动栈回收，也就没有了内存回收压力。相当于此方法执行完成之后，此方法内用到的栈空间，都会抹掉，自然就不需要算内存回收了。当然了，本文的主角可不是栈内存，细聊下去，我预计还能吹很久。还是回到本文主题吧，大家就只需要记得，以上的代码超级超级省内存分配资源

以上的代码，分配的对象，只有一个字符串，没错，就是返回值的字符串

也就是说在 dotnet 6 以及更高的版本，可以让构建时，将 `$` 内插字符串，构建成为 DefaultInterpolatedStringHandler 结构体对象，而不需要走 `string.Format` 方法的逻辑。这是一个很大的优势。可以让内插的字符串，不需要创建额外的数组存放参数列表，不需要在 `string.Format` 方法里面解析字符串

但大家又有另外一个疑惑，在使用 `DefaultInterpolatedStringHandler` 的 ToStringAndClear 方法的时候，难道底层不需要一个缓存使用的数组么？实际上还是有用到的，要不然，还要本文的主角做啥。在 ToStringAndClear 方法里面，实际上是需要用到一个数组进行缓存的，不然的话，代码还是有点坑。用到了数组缓存，为什么在本文上面还说没有额外的内存分配？别忘了数组池哦

默认在 DefaultInterpolatedStringHandler 里，将申请 `ArrayPool<char>.Shared` 一个数组池的数组空间来作为缓存。在大部分情况下，可以认为这是一个无伤的过程。然而数组池也不见得每次都有那么空闲。而且，借和还是需要算利息的哦

为了减少利息，减少 CPU 计算的耗时，就到了本文的主角，也就是 `string.Create` 新加入的重载方法出场的时候

如上文，调用 DefaultInterpolatedStringHandler 里，也需要一个缓存数组。那这个数组，如果也是从栈上过来的呢，是不是就更省一些了？没错。那如何将从栈上的数组给到 DefaultInterpolatedStringHandler 结构体，这就需要用到本文的主角了

先通过 stackalloc 申请一定的数组空间，再将数组空间给到 DefaultInterpolatedStringHandler 结构体，即可实现几乎所有内存的分配逻辑都是在栈上分配的。将随着方法的结束，自动清理垃圾

用法如下：

```csharp
public static string FormatVersion(int major, int minor, int build, int revision) =>
    string.Create(null, stackalloc char[64], $"{major}.{minor}.{build}.{revision}");
```

以上的用法属于高级用法部分。在构建的时候，将自动拆分内插字符串为 DefaultInterpolatedStringHandler 结构体，提示将传入的 `stackalloc char[64]` 作为缓冲的数组传入使用。如此即可实现，除了返回值的字符串，就不需要从堆上额外申请空间。而且在传入的缓冲数组够用的情况下，也不用数组池里申请缓存数组空间，减少了一借一还的时间损耗，从而达到极高的性能

但，这是高级的用法，还是要需要小心的事项的。第一个就是，咱使用 stackalloc 是在栈上分配内存空间，分配的大小可要小心哦，如果将栈上的空间玩爆了，那就只能再见了。默认分配 512 一下，可以认为是安全的。不过，分配越小越好，刚刚好够用就好哦。千万别多打了几个 0 哦

第二个就是如果传入的缓存空间不足了，那依然会需要从数组池里申请内存空间。而不是进行栈空间越界炸掉你的应用。更进一步的说明，有时，咱是无法预估此内插字符串所使用的缓存大小需要多大的。如果真的难以预估的话，而且实际业务预期也会超过预估的大小，那么使用以上的方法，相当于白申请一段栈空间，不如不要

如果实际所需要的字符串拼接的缓存空间比传入的 stackalloc 的空间还要更大。那么在 runtime 底层，将抛弃传入的数组空间，改用从数组池申请的空间。因此，传入 stackalloc 申请的预估的固定大小的数组，在开发中是安全的。预估的固定大小，如果小了，是不会有逻辑上的问题的

例如使用的内插字符串的拼接需要 5000 的 char 数组空间大小作为缓存空间，然而传入的 `stackalloc` 申请的空间是 `stackalloc char[64]` 那显然不够用。这是没有问题的，在底层将重新和数组池借足够的空间。不会强行在你的栈上分配空间越界的

对于字符串来说，还有一个很重要的就是语言文化。例如对于日期来说，美国和中国的文化的日期的字符串表示是不相同的。自然在格式化输出字符串时，最好是带上日期。咱上面的例子只是为了简单，将 IFormatProvider 传入空值而已。实际上可以传入符合你预期的格式化方法，例如无视语言文化的格式化

```csharp
public static string FormatVersion(int major, int minor, int build, int revision) =>
    string.Create(CultureInfo.InvariantCulture, stackalloc char[64], $"{major}.{minor}.{build}.{revision}");
```

以上的 `CultureInfo.InvariantCulture` 将对后续的内插字符串进行对应的格式化，如此可以解决很多语言文化的坑

对于咱的应用代码，如果需要给用户展示的，最好是根据当地的语言文化进行展示。而对于咱应用里层的计算逻辑，最好是做语言文化无关的。如此才能保持逻辑的符合预期，毕竟诡异的语言格式化还是很多的，采用语言文化无关，可以保持咱应用内计算逻辑符合预期

在 dotnet 6 下，如有使用 `string.Create` 这两个新的重载方法进行拼接字符串，性能上是比 `StringBuilder` 更高的

如以下的代码，是采用 StringBuilder 进行拼接创建字符串

```csharp
StringBuilder stringBuilder = new StringBuilder(64);
stringBuilder.Append(cr.TopLeft.ToString(cultureInfo));
stringBuilder.Append(listSeparator);
stringBuilder.Append(cr.TopRight.ToString(cultureInfo));
stringBuilder.Append(listSeparator);
stringBuilder.Append(cr.BottomRight.ToString(cultureInfo));
stringBuilder.Append(listSeparator);
stringBuilder.Append(cr.BottomLeft.ToString(cultureInfo));
return sb.ToString();
```

以上代码是需要多在栈上分配一个 StringBuilder 对象的，而且还需要为此对象申请至少一个 64 长度的数组。而在优化之后，采用 `string.Create` 的方式，如以下代码则几乎除了返回值的字符串之外，就不需要再申请任何的空间

```csharp
return string.Create(cultureInfo, stackalloc char[128], $"{cr.TopLeft}{listSeparator}{cr.TopRight}{listSeparator}{cr.BottomRight}{listSeparator}{cr.BottomLeft}");
```

实际上，也不是所有在使用字符串拼接的地方，都使用 StringBuilder 都能提升性能。如果字符串拼接只是很简单的两个字符串相加，那么大多数的时候，使用两个字符串相加的性能是大于采用 StringBuilder 拼接的

这就是本文和大家聊的性能优化点，采用 C# 10 和 dotnet 6 配合的字符串内插优化方法

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
