# dotnet C# 基础 为什么 GetHashCode 推荐只取只读属性或字段做哈希值

在 C# 里面，所有的对象都继承 Object 类型，此类型有开放 GetHashCode 用于给开发者重写。此 GetHashCode 方法推荐是在重写 Equals 方法时也同时进行重写，要求两个对象在 Equals 返回相等时，两个对象的 GetHashCode 返回值也相等。反过来则不然，允许有两个不相等的对象的 GetHashCode 是相等的

在重写 Equals 方法时，大部分时候都是自动生成的，如将类里面的所有字段或属性都进行一一比较。那在 GetHashCode 方法里面，所输出的哈希值的计算，是否也需要使用此类型的所有字段或属性共同计算出来？如果在 GetHashCode 里面使用的字段或属性非只读，那么 ReSharper 将会警告你这是不安全的。本文将来告诉大家为什么这是不安全的

<!--more-->
<!-- CreateTime:2021/10/8 19:29:51 -->

<!-- 博客 -->

在 dotnet 里面，大部分会用到 GetHashCode 的逻辑都在于哈希容器里面，如 Dictionary 字典等。这些哈希容器在设计上都期望类型遵守以下行为：当两个对象相等的时候，那么获取 GetHashCode 的值也一定相等

假定有类型的 GetHashCode 返回值是基于非只读的属性或字段，将会导致在将对象加入哈希容器的时候，所获取到的 GetHashCode 的值是不包括未来对非只读属性或字段变更的防御的。在未来对此对象的非只读的属性或字段进行变更，也许就会影响到此对象再次获取 GetHashCode 的属性，从而让相同的一个对象，在哈希容器里面，因为 GetHashCode 返回值不同，而被认为是不同的对象

假设有如此的代码逻辑，某个 Foo2 的对象的 GetHashCode 返回值是由此对象的属性决定的，如下面代码

```csharp
    class Foo2
    {
        public int HashCode { set; get; }

        public override int GetHashCode()
        {
            return HashCode;
        }
    }
```

假定将此 Foo2 的对象加入到字典里面，接着去判断字典里面是否存在此对象。再修改 Foo2 的 HashCode 属性，再去判断字典里面是否存在此对象，代码如下

```csharp
                var foo2 = new Foo2();
                Dictionary<Foo2, object> dictionary = new();
                dictionary[foo2] = foo2;
                Console.WriteLine(dictionary.ContainsKey(foo2));
                foo2.HashCode = 2;
                Console.WriteLine(dictionary.ContainsKey(foo2));
```

有趣的逻辑是第一次返回的符合预期，就是 True 的值。然而第二次，明明没有从字典里面移除 Foo2 对象，然而字典却认为找不到此对象

其原因如上文，在字典里面，优先通过 GetHashCode 的值来进行判断。如上面代码，更改了 Foo2 的 GetHashCode 返回值，将会让字典找不到此 HashCode 对应的元素，从而让字典认为不存在此对象

大部分在设计类型的时候，都不会考虑到某个类型在未来或其他模块里面，会被存放进哈希容器里面。如果此时在 GetHashCode 里面，使用了非只读字段或属性，将会挖一个坑。也许某个逻辑变更了这些非只读字段或属性的时候，影响了 GetHashCode 的返回值从而影响了哈希容器的行为

这就是为什么 ReSharper 警告不要在 GetHashCode 里面使用非只读字段或属性进行制作哈希值的原因

不过在理解了这个行为，在某些特别的业务里面，也可以利用此行为实现有趣的功能

通过本文也可以了解到，对于 GetHashCode 的返回值也不能为了因为重写 Equals 方法而被 VS 警告而随便写此方法的实现，如下面逗比代码

```csharp
    class Foo
    {
        public Foo(string name)
        {
            Name = name;
        }

        public string Name { get; }

        public override int GetHashCode()
        {
            return _random.Next();
        }

        private readonly Random _random = new Random();
    }
```

上面的代码在 GetHashCode 随机返回一个值，这将会让所有哈希容器炸掉，如下面的代码，将在字典拿不到值

```csharp
            try
            {
                Foo[] foo = new Foo[100];
                for (int i = 0; i < 100; i++)
                {
                    foo[i] = new Foo(i.ToString());
                }

                Dictionary<Foo, string> dictionary = foo.ToDictionary(t => t, t => t.Name);

                for (int i = 0; i < foo.Length; i++)
                {
                    Console.WriteLine($"{foo[i].Name}-{dictionary[foo[i]]}"); // KeyNotFoundException
                }
            }
            catch (Exception)
            {
            }
```

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/572e405d383c69929397a583102576e2e140f1fc/BelwheaheajeachelYikaidairnay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/572e405d383c69929397a583102576e2e140f1fc/BelwheaheajeachelYikaidairnay) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 572e405d383c69929397a583102576e2e140f1fc
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 BelwheaheajeachelYikaidairnay 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
