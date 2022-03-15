
# dotnet 读 WPF 源代码笔记 为什么加上 BooleanBoxes 类

在 WPF 框架，为什么需要定义一个 BooleanBoxes 类。为什么在 D3DImage 的 Callback 方法里面，传入的是 object 对象，却能被转换为布尔。本文将告诉大家为什么需要这样设计

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

大家都知道，在 dotnet 里面，如果将一个结构体通过 object 的方式传输，将需要进行装箱。而装箱将会创建一个新的对象。在 WPF 这个框架里面，有很多逻辑，例如消息，都是非常快速在调用的。如果每次调用，例如传输布尔值，由于需要进入很多框架逻辑，而让参数只能使用 object 类型，那么每次都使用结构体将需要多次的装箱，从而创建大量的对象

创建大量的对象将会让界面逻辑需要不断进行内存回收，自然性能就降低了

那为什么不设计一个泛形呢？因为代码将不好写，同时由于泛形类型的静态属性将不相同，从而再次让逻辑更加复杂。而且对于大多数逻辑来说，确实传输的只是结构体。在 WPF 框架，为了解决此问题，于是就创建了 KnownBoxes 系列类型。包括 NullableBooleanBoxes 和 BooleanBoxes 类型。这两个类型将预先将布尔装箱，当成 object 对象。接下来，所有需要对布尔装箱的逻辑，都将使用 BooleanBoxes 的对象代替

以下代码是 BooleanBoxes 的逻辑

```csharp
    internal static class BooleanBoxes
    {
        internal static object TrueBox = true;
        internal static object FalseBox = false;

        internal static object Box(bool value)
        {
            if (value)
            {
                return TrueBox;
            }
            else
            {
                return FalseBox;
            }
        }
    }
```

可以看到 BooleanBoxes 的 TrueBox 和 FalseBox 属性都是由布尔装箱创建的。为什么创建的方法是需要使用布尔装箱，而不是随便拿两个对象？原因是如此方便重新转换为布尔值

使用 BooleanBoxes 的性能如何？请看 [https://github.com/dotnet/runtime/issues/7079#issuecomment-264500921](https://github.com/dotnet/runtime/issues/7079#issuecomment-264500921)

|                  Method |      Mean |    StdDev |    Median | Scaled |
|------------------------ |---------- |---------- |---------- |------- |
|      BoolUncachedBoxing | 7.3923 ns | 0.0391 ns | 7.3866 ns |   1.00 |
|        BoolCachedBoxing | 4.5859 ns | 0.0310 ns | 4.5954 ns |   0.62 |

那为什么在 dotnet 里面，不默认加上此优化呢？原因是如文档，每次在 dotnet 的装箱，都是生成新的对象。没错，新的对象。因此如果做此优化，将修改行为

那这和 D3DImage 的 Callback 方法里面，有什么关系呢？其实在此方法里面，调用到 SetIsFrontBufferAvailable 方法，先来看看此方法做了什么

```csharp
        private object SetIsFrontBufferAvailable(object isAvailableVersionPair)
        {
            Pair pair = (Pair)isAvailableVersionPair;
            uint version = (uint)pair.Second;

            if (version == _version)
            {
                bool isFrontBufferAvailable = (bool)pair.First;
                SetValue(IsFrontBufferAvailablePropertyKey, isFrontBufferAvailable);
            }

            // ...just because DispatcherOperationCallback requires returning an object
            return null;
        }
```

此方法的参数能拿到一个 Pair 类型的对象，然而此对象的两个值都是 object 类型，需要进行一次转换。然而在 Callback 方法里面，代码如下

```csharp
        private void Callback(bool isFrontBufferAvailable, uint version)
        {
            Dispatcher.BeginInvoke(
                DispatcherPriority.Normal,
                new DispatcherOperationCallback(SetIsFrontBufferAvailable),
                new Pair(BooleanBoxes.Box(isFrontBufferAvailable), version)
                );
        }
```

可以看到在传入的参数，拿到的 Pair 的第一个参数，是用 BooleanBoxes 创建的

那为什么一个 object 对象，在 SetIsFrontBufferAvailable 能被转换为布尔呢？这就是 BooleanBoxes 的属性都是由布尔装箱创建的原因。因为本来是通过布尔装箱创建的，也因此能被转换为布尔值

以上就是 WPF 为什么加上 BooleanBoxes 类的原因，以及在 D3DImage 里，使用布尔强转一个 object 可以符合预期

更多逻辑，还请阅读 WPF 源代码

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。