# VisualStudio 打断点和不打断点的区别

因为小伙伴告诉我他的代码在打断点的时候可以运行，但是在不打的时候出现异常。我去他那里看到，真的是这样，最后发现原来是代码写错了。本文来告诉大家，如果遇到了进入断点和没有进入有区别，可能的问题。

<!--more-->
<!-- CreateTime:2020/3/5 9:26:16 -->

<!-- csdn -->
<!-- 标签：VisualStudio，调试 -->

<div id="toc"></div>

如果发现打断点和不打软件运行不同，那么可能的原因就是时间，例如有两个线程，使用相同属性，请看代码

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

可以看到线程2依赖线程1先运行，所以可能在运行的时候，有时可以进入代码，有时无法。只要这时存在线程2等待一下，那么一般都可以进入。所以遇到断点让程序可以运行或者不能的时候，先看一下是不是线程的问题。有没有属性没有加锁。

## 随机数

实际上小伙伴的问题是有随机数。默认的随机数的构造方法使用的是时间，所以如果创建两个随机数，可能拿到的值是相同，请看下面的代码

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
            return Equals((DhtwSwyotml)obj);
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

使用下面的代码的时候出现了错误访问

```csharp
        static void Main(string[] args)
        {
            var sutpSgeg = new DhtwSwyotml();
            var kzduDglirxr = new DhtwSwyotml();
            if (sutpSgeg.Equals(kzduDglirxr))
            {
                Console.WriteLine("错误进入");
            }
         
        }
```

本来两个不同的对象希望返回 false ，结果写为 true ，而且在 kzduDglirxr 断点可以看到不会进入错误的代码。

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018213174010.jpg)

所以如果遇到这个问题，首先需要想是不是时间问题，如果遇到时间的问题，那么需要解决代码才可以。

参见：[Make GenericParameterHelper's behavior same between running and debugging by walterlv · Pull Request #362 · Microsoft/testfx](https://github.com/Microsoft/testfx/pull/362 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
