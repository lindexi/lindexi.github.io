# C# 标准性能测试高级用法

本文告诉大家如何在项目使用性能测试测试自己写的方法

<!--more-->
<!-- CreateTime:2019/11/29 10:13:16 -->

<div id="toc"></div>

在 [C# 标准性能测试](https://blog.lindexi.com/post/C-%E6%A0%87%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95.html ) 已经告诉大家如何使用 BenchmarkDotNet 测试性能，本文会告诉大家高级的用法。

建议是创建一个控制台项目用来做性能测试，这个项目要求是 dotnet framework 4.6 以上，建议是 4.7 的版本。使用这个项目引用需要测试的项目，然后在里面写测试的代码。

例如被测试项目有一个类 Foo 里面有一个方法是 lindexidb ，需要测试 林德熙逗比 方法的性能

最简单的测试的代码

```csharp
public class FooPerf
{
	[Benchmark]
	public void lindexidb()
	{
		new Foo().lindexidb();
	}
}
```

在 Main 函数使用下面代码

```csharp
  var boKar = BenchmarkRunner.Run<Foo>();
```

这样就可以进行测试，如果需要传入一些参数，那么就需要使用本文的方法

## 传入参数

如果需要测试的方法需要传入不同的参数，而且在使用不同的参数的性能也是不相同，就需要使用传入参数特性。

例如有底层的项目

```csharp
    public class Foo
    {
        public void Lindexidb()
        {

        }
    }
```

需要创建另一个项目测试这个项目的性能， 需要注意不要在自己的库安装 BenchmarkDotNet ，安装之后会让启动速度慢很多

在测试性能的另一个项目，安装 BenchmarkDotNet 引用库测试，所有的代码

```csharp
   class Program
    {
        static void Main(string[] args)
        {
            BenchmarkRunner.Run<FooPerf>();
            Console.Read();
        }
    }

    public class FooPerf
    {
        [Benchmark]
        public void Lindexidb()
        {
            var foo = new Foo();
            foo.Lindexidb();
        }
    }
```

需要知道，必须设置 FooPerf 的访问是 public 没有设置会出现异常

现在例如修改了 Lindexidb 需要传入参数

```csharp
    public class Foo
    {
        public void Lindexidb(int a, int b)
        {
            var foo = a + b;
            if (Arguments>a)
            {
                return;
            }
            if (foo == 2)
            {
                Arguments = foo;
            }
        }

        public int Arguments { get; set; }
    }
```

现在需要修改性能项目

```csharp
    public class FooPerf
    {
        [Benchmark(Description = "这里可以写这个方法是什么")]
        public void Lindexidb()
        {
            var foo = new Foo();
            foo.Lindexidb(2, 3);
        }
    }
```

可以看到上面写法很难写出测试很多参数

```csharp
    public class FooPerf
    {
        [Benchmark]
        [Arguments(100, 10)]
        [Arguments(2, 10)]
        [Arguments(2, 3)]
        [Arguments(10, 3)]
        [Arguments(21, 3)]
        public void Lindexidb(int a,int b)
        {
            var foo = new Foo();
            foo.Lindexidb(a, b);
        }
    }
```

通过 Arguments 可以传入不同的参数，使用这个方法可以防止初始化参数需要的时间计算为算法的

运行程序可以看到下面输出

```csharp
//   FooPerf.Lindexidb: DefaultJob [a=2, b=3]
//   FooPerf.Lindexidb: DefaultJob [a=2, b=10]
//   FooPerf.Lindexidb: DefaultJob [a=10, b=3]
//   FooPerf.Lindexidb: DefaultJob [a=21, b=3]
//   FooPerf.Lindexidb: DefaultJob [a=100, b=10]
```

在使用不同的参数可以看到不同的速度

|    Method |   a |  b |     Mean |     Error |    StdDev |
|---------- |---- |--- |---------:|----------:|----------:|
| **Lindexidb** |   **2** |  **3** | **2.037 ns** | **0.0749 ns** | **0.0833 ns** |
| **Lindexidb** |   **2** | **10** | **3.263 ns** | **0.0992 ns** | **0.2682 ns** |
| **Lindexidb** |  **10** |  **3** | **2.333 ns** | **0.0798 ns** | **0.1038 ns** |
| **Lindexidb** |  **21** |  **3** | **2.278 ns** | **0.0776 ns** | **0.0863 ns** |
| **Lindexidb** | **100** | **10** | **2.364 ns** | **0.0809 ns** | **0.2242 ns** |


可以传入不同的参数，传入的参数可以自动转换

如果传入的参数不对，就会提示，如下面代码

```csharp
        [Benchmark]
        [Arguments("123", "123")]
        [Arguments(2, 10)]
        [Arguments(2, 3)]
        [Arguments(10, 3)]
        [Arguments(21, 3)]
        public void Lindexidb(int a,int b)
        {
            var foo = new Foo();
            foo.Lindexidb(a, b);
        }
```

本来是使用 int 但是参数写 string 所以会出现下面提示

```csharp
// Build Exception: The build has failed!
CS0029: Cannot implicitly convert type 'string' to 'int'
CS0029: Cannot implicitly convert type 'string' to 'int'
```

如果需要参数是 一个，如代码，传入的参数是两个，那么会出现异常 

```csharp
    public class FooPerf
    {
        [Benchmark]
        [Arguments(1, 2)]
        [Arguments(2, 10)]
        [Arguments(2, 3)]
        [Arguments(10, 3)]
        [Arguments(21, 3)]
        public void Lindexidb(int a)
        {
            var foo = new Foo();
            var b = Arguments;
            foo.Lindexidb(a, b);
        }

        public int Arguments { get; set; }
    }
```



## 属性

属性和字段都可以修改，但是修改字段需要修改公开字段，不推荐修改字段

```csharp
        [Params(10, 2, 3)]
        public int Arguments { get; set; }
```

可以设置属性的值为 `10,2,3` 在下面代码会组合属性和传入参数

```csharp
        [Benchmark(Description = "这里可以写这个方法是什么")]
        [Arguments(1)]
        [Arguments(2)]
        [Arguments(2)]
        [Arguments(10)]
        [Arguments(21)]
        public void Lindexidb(int a)
        {
            var foo = new Foo();
            var b = Arguments;
            foo.Lindexidb(a, b);
        }

        [Params(10, 2, 3)]
        public int Arguments { get; set; }
```

运行看到有 15 个测试

```csharp
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=1]
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=10]
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=21]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=1]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=10]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=21]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=1]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=10]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=21]
```

## 传入多个值

可以看到在特性写参数是比较多的，如果需要很多参数就需要写很多代码

可以使用数组的方式把很多的代码作为数组

请看代码

```csharp
        [Benchmark(Description = "这里可以写这个方法是什么")]
        [ArgumentsSource(nameof(LeesikeasowSearjeeball))]
        public void Lindexidb(int a, int b)
        {
            var foo = new Foo();
            foo.Lindexidb(a, b);
        }

        public IEnumerable<object[]> LeesikeasowSearjeeball()
        {
            yield return new object[] {2, 3};
            yield return new object[] {10, 2};
            yield return new object[] {5, 2};
            yield return new object[] {100, 5};
            yield return new object[] {3, 100};
        }
```

上面使用 LeesikeasowSearjeeball 作为输入的参数，注意需要返回一个数组，这个数组里就是参数的列表。上面使用的参数有两个，所以数组就是包含两个参数 

```csharp
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=2, b=3]
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=3, b=100]
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=5, b=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=10, b=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=2, a=100, b=5]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=2, b=3]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=3, b=100]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=5, b=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=10, b=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=3, a=100, b=5]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=2, b=3]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=3, b=100]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=5, b=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=10, b=2]
//   FooPerf.Lindexidb: DefaultJob [Arguments=10, a=100, b=5]
```

除了可以设置方法传入，还可以设置属性

```csharp
        [Benchmark]
        public void Foo()
        {
            for (int i = 0; i < Arguments; i++)
            {
                
            }
        }

        [ParamsSource(nameof(PememasiDismikasu))]
        public int Arguments { get; set; }

        public IEnumerable<int> PememasiDismikasu => new[] { 100, 200 };
```

通过 ParamsSource 可以告诉测试使用的从哪个拿到


## 基线

基线可以用在三个不同的地方，最简单的是方法，另外可以用在分类和不同环境。

因为测试的时间在不同的设备的时间都不相同，如何判断一个方法优化之后是比原来好？方法就是把原来的方法作为基线，这样可以对比不同的方法的速度

如有三个不同的方法，选一个作为基线

```csharp
        [Benchmark]
        public void Time50() => Thread.Sleep(50);

        [Benchmark(Baseline = true)]
        public void Time100() => Thread.Sleep(100);

        [Benchmark]
        public void Time150() => Thread.Sleep(150);
```

设置基线的方法是添加 Baseline = true ，建议在原来的方法添加，然后使用不同的方法看哪个方法的速度比较快

在输出会添加一列 `Scaled` 用于表示这个方法对比基线的速度，他的时间是基线的多少。如上面代码的运行会输出

|  Method |      Mean |     Error |    StdDev | Scaled |
|-------- |----------:|----------:|----------:|-------:|
|  Time50 |  50.46 ms | 0.0779 ms | 0.0729 ms |   0.50 |
| Time100 | 100.39 ms | 0.0762 ms | 0.0713 ms |   1.00 |
| Time150 | 150.48 ms | 0.0986 ms | 0.0922 ms |   1.50 |

这里的 Scaled 就是对比基线方法的时间

如果在不同的分类下需要做不同的标准，就可以在 BenchmarkCategory 添加 Baseline 告诉在哪个分类使用哪个方法作为标准。如下面的代码，设置 Fast 类和 Slow 类使用不同的标准 

```csharp
    [GroupBenchmarksBy(BenchmarkLogicalGroupRule.ByCategory)]
    [CategoriesColumn]
    public class IntroCategoryBaseline
    {
        [BenchmarkCategory("Fast"), Benchmark(Baseline = true)]
        public void Time50() => Thread.Sleep(50);

        [BenchmarkCategory("Fast"), Benchmark]
        public void Time100() => Thread.Sleep(100);

        [BenchmarkCategory("Slow"), Benchmark(Baseline = true)]
        public void Time550() => Thread.Sleep(550);

        [BenchmarkCategory("Slow"), Benchmark]
        public void Time600() => Thread.Sleep(600);
    }
```

运行的输出，可以看到对于不同的分类用的是不同的方法

|  Method | Categories |      Mean |     Error |    StdDev | Scaled |
|-------- |----------- |----------:|----------:|----------:|-------:|
|  Time50 |       Fast |  50.46 ms | 0.0745 ms | 0.0697 ms |   1.00 |
| Time100 |       Fast | 100.47 ms | 0.0955 ms | 0.0893 ms |   1.99 |
|         |            |           |           |           |        |
| Time550 |       Slow | 550.48 ms | 0.0525 ms | 0.0492 ms |   1.00 |
| Time600 |       Slow | 600.45 ms | 0.0396 ms | 0.0331 ms |   1.09 |

基线除了可以测试方法的基线，还可以测试环境。如我的代码需要在 `Clr` `Mono` `Core` 三个不同环境运行，这时我想知道对比 Clr 环境，其他两个环境的性能。可以使用 JobBaseline 的方式。

```csharp
    [ClrJob(baseline: true)]
    [MonoJob]
    [CoreJob]
    public class IntroJobBaseline
    {
        [Benchmark]
        public int SplitJoin() 
            => string.Join(",", new string[1000]).Split(',').Length;
    }
```

这时输出可以看到 Clr 运行的是标准，在 Core 运行时间是在 Clr 运行的 0.67 通过这个方法就知道在不同的环境相同的方法的测试

|    Method | Runtime |     Mean |     Error |    StdDev | Scaled | ScaledSD |
|---------- |-------- |---------:|----------:|----------:|-------:|---------:|
| SplitJoin |     Clr | 19.42 us | 0.2447 us | 0.1910 us |   1.00 |     0.00 |
| SplitJoin |    Core | 13.00 us | 0.2183 us | 0.1935 us |   0.67 |     0.01 |
| SplitJoin |    Mono | 39.14 us | 0.7763 us | 1.3596 us |   2.02 |     0.07 |

更多关于基线请看 [Benchmark and Job Baselines ](https://benchmarkdotnet.org/articles/features/baselines.html )

## 分类

如果在一个类的测试方法有不同的类型，而只需要测试某几个类型的就需要使用本文的方法

```csharp
    [DryJob]
    [CategoriesColumn]
    [BenchmarkCategory("分类")]
    [AnyCategoriesFilter("A", "1")]
    public class FooPerf
    {
        [Benchmark]
        [BenchmarkCategory("A", "1")]
        public void A1() => Thread.Sleep(10); // Will be benchmarked

        [Benchmark]
        [BenchmarkCategory("A", "2")]
        public void A2() => Thread.Sleep(10); // Will be benchmarked

        [Benchmark]
        [BenchmarkCategory("B", "1")]
        public void B1() => Thread.Sleep(10); // Will be benchmarked

        [Benchmark]
        [BenchmarkCategory("B", "2")]
        public void B2() => Thread.Sleep(10);
    }
```

在方法表示方法属于的类型，可以标记一个方法属于多个类型，如 A1 方法属于 `A` `1` 两个类型，在类标记 AnyCategoriesFilter 表示测试某些类型，这里标记了 `A` 和 1 也就是所有包含 A 或 1 类型的方法会被测试，所以 A1 A2 B1 都会被运行。

## 包含名字

如果在一个类有很多方法，只需要名字满足某些条件的方法才可以执行，就需要进行包含名字判断。

```csharp
    [Config(typeof(Config))]
    public class FooPerf
    {
        private class Config : ManualConfig
        {
            // 只有在名字满足包含 "A" 或 "1" 和名字长度小于 3 才执行
            public Config()
            {
                Add(new DisjunctionFilter
                (
                    // 这里的是或关系
                    // 只要名字包含 "A" 或 "1" 就执行
                    new NameFilter(name => name.Contains("A")),
                    new NameFilter(name => name.Contains("1"))
                ));

                // 这里和上面是 And 关系，也就是必须要同时满足名字长度小于 3 才可以执行
                Add(new NameFilter(name => name.Length < 3));
            }
        }

        [Benchmark]
        public void A1() => Thread.Sleep(10); // Will be benchmarked

        [Benchmark]
        public void A2() => Thread.Sleep(10); // Will be benchmarked

        [Benchmark]
        public void A3() => Thread.Sleep(10); // Will be benchmarked

        [Benchmark]
        public void B1() => Thread.Sleep(10); // Will be benchmarked

        [Benchmark]
        public void B2() => Thread.Sleep(10);

        [Benchmark]
        public void B3() => Thread.Sleep(10);

        [Benchmark]
        public void C1() => Thread.Sleep(10); // Will be benchmarked

        [Benchmark]
        public void C2() => Thread.Sleep(10);

        [Benchmark]
        public void C3() => Thread.Sleep(10);

        [Benchmark]
        public void Aaa() => Thread.Sleep(10);
    }
```

在类添加特性，告诉这个类需要使用哪个配置，在配置的构造函数使用了两次的 Add 函数，在多个 Add 之间是 And 关系，也就是必须所有 Add 的条件都满足才可以执行。在一个Add使用的 DisjunctionFilter 可以使用或关系多个条件。

上面的函数使用的满足名字带有 A 或 1 而且名字的长度小于 3 才可以执行。

除了使用名字作为条件，还可以使用 AnyCategoriesFilter 表示存在任意的类型符合，AllCategoriesFilter 要求所有的类型都符合。

<!-- ## 泛型参数

有一些测试方法是需要传入泛型的，如下面方法

```csharp
    public class Foo
    {
        public void Lindexidb<T>(T a, T b) where T : struct, IComparable<T>
        {
            if (a.CompareTo(b) > 0)
            {
                Arguments = 2;
            }
        }

        public int Arguments { get; set; }
    }
```

就需要测试使用不同的泛型，方法的速度 -->

## 运行多个类

一个需要测试的类需要使用下面代码

```csharp
            BenchmarkRunner.Run<FooPerf>();

```

只能测试一个类，如果有很多类就需要写很多代码，下面告诉大家如何找到所有方法

```csharp
 BenchmarkSwitcher.FromAssembly(typeof(Program).GetTypeInfo().Assembly).Run(args);

```

在运行的时候就可以选运行哪个

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

