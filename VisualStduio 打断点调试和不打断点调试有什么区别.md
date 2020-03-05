# VisualStduio 打断点调试和不打断点调试有什么区别

最近小伙伴告诉我，他的代码在打断点的时候，运行到断点，之后就可以运行。如果没有断点，他的代码就无法运行，经过了一段时间的研究才发现，原来打断点和不打断点是有一些区别。

本文来告诉大家，如果在自己的软件发现打断点之后程序和不打有区别，如在打断点之后程序可以运行，不打就不能运行，那么可以从下面的方法开始查看是否程序的问题。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->

<!-- csdn -->
<div id="toc"></div>

<!-- 标签：VisualStudio，调试 -->

实际上进入断点之后的运行有一个很大的不同在于时间。对于线程调度等，因为时间的不同，进行调度的顺序可能被修改。例如有两个线程，使用相同属性，请看代码。

```csharp

int n = 0;

//线程1
if(n == 0)
{
	n ++;
}

//线程2
if(n == 1)
{
	//代码
}

```

这时如果同时运行 线程1 和 线程2 ，可能线程2的代码就不会运行，因为他判断的 n 还是 0 不是 1。在线程2的判断打断点，这时会发现，经常可以运行代码。

所以遇到打断点问题，那么请看一下使用的判断是否使用很多线程，所以遇到断点让程序可以运行或者不能的时候，先看一下是不是线程的问题。有没有属性没有加锁。

## 随机数

最近在测试框架也看到随机数的问题。

下面是一个简单的类，在构造创建一个随机数，判断相同使用的是判断随机数属性是否相同。实际上小伙伴的问题是有随机数。默认的随机数的构造方法使用的是时间，所以如果创建两个随机数，可能拿到的值是相同，请看下面的代码

```csharp
    class DhtwSwyotml : IEquatable<DhtwSwyotml>
    {
        /// <inheritdoc />
        public DhtwSwyotml()
        {
            var ran = new Random();
            HvkemkKevavvqur = ran.Next();
        }

        public int HvkemkKevavvqur { get; set; }

        /// <inheritdoc />
        public bool Equals(DhtwSwyotml other)
        {
            if (ReferenceEquals(null, other)) return false;
            if (ReferenceEquals(this, other)) return true;
            return HvkemkKevavvqur == other.HvkemkKevavvqur;
        }

        /// <inheritdoc />
        public override bool Equals(object obj)
        {
            if (ReferenceEquals(null, obj)) return false;
            if (ReferenceEquals(this, obj)) return true;
            if (obj.GetType() != this.GetType()) return false;
            return Equals((DhtwSwyotml) obj);
        }

        /// <inheritdoc />
        public override int GetHashCode()
        {
            return HvkemkKevavvqur;
        }

        public static bool operator ==(DhtwSwyotml left, DhtwSwyotml right)
        {
            return Equals(left, right);
        }

        public static bool operator !=(DhtwSwyotml left, DhtwSwyotml right)
        {
            return !Equals(left, right);
        }
    }

```

如果使用的是两个不同的对象，那么一般判断相等都是 false ，但是下面的代码在不打断点可能进入错误的代码

```csharp
        static void Main(string[] args)
        {
            var sutpSgeg = new DhtwSwyotml();
            var kzduDglirxr = new DhtwSwyotml();
            if (sutpSgeg.Equals(kzduDglirxr))
            { 
                //xx 这时不应该进入
                  Console.WriteLine("错误进入");
            }

            Console.WriteLine(sutpSgeg.HvkemkKevavvqur);
            Console.WriteLine(kzduDglirxr.HvkemkKevavvqur);

        }
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018213111657.jpg)

在 kzduDglirxr 打断点和不在他这里打断点，得到不相同的结果。来两个不同的对象希望返回 false ，结果写为 true ，而且在 kzduDglirxr 断点可以看到不会进入错误的代码。打的时候不会进入判断相等的代码，但是不打的时候可能可以进入判断相等。因为Random的构造函数默认给他是当前的时间，两个对象的创建使用的时间很短，所以创建的随机数可能就相同。但是不是所有的运行都能进入判断。我把上面代码给我小伙伴，结果他一直运行都是不会进入错误的代码。

参见：[Make GenericParameterHelper's behavior same between running and debugging by walterlv · Pull Request #362 · Microsoft/testfx](https://github.com/Microsoft/testfx/pull/362 )

所以在发现断点和没有之间出现不同的，请看一下是不是有线程问题或者随机数问题和所有时间有关的。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
