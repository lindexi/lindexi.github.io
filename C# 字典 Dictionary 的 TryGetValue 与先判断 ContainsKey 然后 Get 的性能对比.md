# C# 字典 Dictionary 的 TryGetValue 与先判断 ContainsKey 然后 Get 的性能对比

本文使用 [benchmarkdotnet ](https://blog.lindexi.com/post/C-%E6%A0%87%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95.html ) 测试字典的性能，在使用字典获取一个可能存在的值的时候可以使用两个不同的写法，于是本文分析两个写法的性能。

<!--more-->
<!-- CreateTime:2019/11/29 10:13:19 -->


判断值存在，如果值存在就获取值，可以使用下面两个不同的方法

一个方法是使用 TryGetValue 请看下面代码

```csharp
            if (Dictionary.TryGetValue(xx, out var foo))
            {
            }
```

另一个方法是先判断是否存在然后再获取，请看下面代码

```csharp
if(Dictionary.ContainsKey(xx))
{
	var foo = Dictionary[xx];
}
```

于是本文就使用[benchmarkdotnet ](https://blog.lindexi.com/post/C-%E6%A0%87%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95.html )测试两个方法的性能

下面是进行测试的数据，测试的代码放在本文的最后。这里的 TryGetExist 方法就是尝试获取一个值，这个值是存在的。而 ContainGetExist 是先判断值是否存在，如果存在就尝试获取这个值。

``` ini

BenchmarkDotNet=v0.10.14, OS=Windows 10.0.17134
Intel Core i7-6700 CPU 3.40GHz (Skylake), 1 CPU, 8 logical and 4 physical cores
Frequency=3328130 Hz, Resolution=300.4690 ns, Timer=TSC
  [Host]     : .NET Framework 4.7 (CLR 4.0.30319.42000), 64bit RyuJIT-v4.7.3132.0  [AttachedDebugger]
  DefaultJob : .NET Framework 4.7 (CLR 4.0.30319.42000), 64bit RyuJIT-v4.7.3132.0


```

|            Method |     Mean |     Error |    StdDev |   Median |
|------------------ |---------:|----------:|----------:|---------:|
|       TryGetExist | 30.26 ns | 0.6057 ns | 0.5949 ns | 30.11 ns |
|   ContainGetExist | 46.36 ns | 1.0883 ns | 3.1919 ns | 44.90 ns |
|     TryGetNoExist | 20.23 ns | 0.4661 ns | 0.7658 ns | 19.93 ns |
| ContainGetNoExist | 18.68 ns | 0.2569 ns | 0.2403 ns | 18.66 ns |


同样对比 ConcurrentDictionary 线程安全的类的性能，也就是将会上面的 Foo 测试类的字典替换为 ConcurrentDictionary 其他代码都不修改，下面是测试的数据，可以看到使用 TryGetValue 的性能依然比较好


``` ini

BenchmarkDotNet=v0.10.14, OS=Windows 10.0.17134
Intel Core i7-6700 CPU 3.40GHz (Skylake), 1 CPU, 8 logical and 4 physical cores
Frequency=3328130 Hz, Resolution=300.4690 ns, Timer=TSC
  [Host]     : .NET Framework 4.7 (CLR 4.0.30319.42000), 64bit RyuJIT-v4.7.3132.0  [AttachedDebugger]
  DefaultJob : .NET Framework 4.7 (CLR 4.0.30319.42000), 64bit RyuJIT-v4.7.3132.0


```

|            Method |     Mean |     Error |    StdDev |   Median |
|------------------ |---------:|----------:|----------:|---------:|
|       TryGetExist | 31.20 ns | 0.4644 ns | 0.3625 ns | 31.17 ns |
|   ContainGetExist | 66.80 ns | 2.4692 ns | 7.2806 ns | 63.84 ns |
|     TryGetNoExist | 20.07 ns | 0.1254 ns | 0.1112 ns | 20.04 ns |
| ContainGetNoExist | 27.63 ns | 0.4230 ns | 0.3956 ns | 27.65 ns |


所有代码

```csharp
    public class Foo
    {
        /// <inheritdoc />
        public Foo()
        {
            var ran = new Random();
            bool set = false;
            for (int i = 0; i < 100000; i++)
            {
                LazyDictionary[ran.Next().ToString() + "-" + i.ToString()] = ran.Next().ToString();
                if (!set)
                {
                    if (ran.Next() < i)
                    {
                        set = true;
                        LazyDictionary["lindexi"] = "逗比";
                    }
                }
            }
        }

        [Benchmark]
        public void TryGetExist()
        {
            if (LazyDictionary.TryGetValue("lindexi", out var foo))
            {
                _foo = foo;
            }
        }

        [Benchmark]
        public void ContainGetExist()
        {
            if (LazyDictionary.ContainsKey("lindexi"))
            {
                _foo = LazyDictionary["lindexi"];
            }
        }


        [Benchmark]
        public void TryGetNoExist()
        {
            if (LazyDictionary.TryGetValue("lindexi123", out var foo))
            {
                _foo = foo;
            }
        }

        [Benchmark]
        public void ContainGetNoExist()
        {
            if (LazyDictionary.ContainsKey("lindexi123"))
            {
                _foo = LazyDictionary["lindexi123"];
            }
        }

        private object _foo;

        private Dictionary<string, object> LazyDictionary { get; } = new Dictionary<string, object>();

    }

```

我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=19bm8i8js1ezb

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
