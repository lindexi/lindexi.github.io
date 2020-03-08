# Moq基础 判断方法被执行

如果想知道注入的类的某个方法被使用了几次，就可以通过 mock 提供的方法进行判断方法有没被执行或被使用多少次

<!--more-->
<!-- CreateTime:2019/1/29 16:29:57 -->

<!-- 标签：mock，单元测试 -->

本文是一个系列，具体请看

 - [Moq基础（一） 为什么需要单元测试框架](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%B8%80.html )

 - [Moq基础（二） 快速使用 Mock 写代码，区分stub和mock是什么](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%BA%8C.html )

 - [Moq基础（三） 伪造特定方法](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%B8%89.html )

 - [Moq基础（四） 伪造属性和事件](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E5%9B%9B.html )

 - [Moq基础（五） 参数匹配，回调，和验证](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%BA%94.html )

 - [Moq基础（六） 对Moq使用的评价](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E5%85%AD.html )

本文是在Moq基础（五）的基础上做的补充

例如有方法 Foo 支持注入接口 IFoo 需要判断在 Foo 方法里面调用 IFoo 的 Foo 方法几次

```csharp
   public class Foo
    {
        /// <inheritdoc />
        public Foo(IFoo foo)
        {
            _foo = foo;
        }

        public void A()
        {
            _foo.Foo();
        }

        private readonly IFoo _foo;
    }

    public interface IFoo
    {
        void Foo();
    }
```

此时就可以通过先做一个虚拟的方法

```csharp

                    var mock = new Mock<IFoo>();
                    var foo = new Foo(mock.Object);
                    // 创建一个虚拟方法
                    mock.Setup(fake => fake.Foo());
```

然后调用 foo 的需要被测试方法

```csharp
                    foo.A();
```

接着判断这个 IFoo 的 Foo 被调用多少次

```csharp
                    // 判断在调用 A 之后调用了 IFoo 的 Foo 方法多少次
                    mock.Verify(fake => fake.Foo(), Times.Once);
```

这是整个测试方法的代码


```csharp
    [TestClass]
    public class FT
    {
        [ContractTestCase]
        public void A()
        {
            "调用 Foo 的 A 方法时会调用 IFoo 的 Foo 方法一次".Test(() =>
            {
                // Arrange

                var mock = new Mock<IFoo>();
                var foo = new Foo(mock.Object);
                // 创建一个虚拟方法
                mock.Setup(fake => fake.Foo());

                // Action

                foo.A();
                foo.A();

                // Assert

                // 判断在调用 A 之后调用了 IFoo 的 Foo 方法多少次
                mock.Verify(fake => fake.Foo(), Times.Once);
            });
        }
    }

```

如果觉得上面的代码写的不错，可以清晰看到每个测试方法，而不是去写小伙伴都看不懂的英文，那么请让使用 MSTestEnhancer 这个测试框架的使用方法十分简单，具体请看[MSTestEnhancer 的使用](https://github.com/dotnet-campus/MSTestEnhancer/blob/master/README.zh-chs.md )

使用了之后就可以在运行单元测试的时候看到有哪些方法可以运行，有哪些不通过

<!-- ![](image/Moq基础 判断方法被执行/Moq基础 判断方法被执行0.png) -->

![](http://image.acmx.xyz/lindexi%2F2019117144456616)

下面将会详细告诉大家如何使用方法验证

在 Mock 可以通过 Setup 做出虚拟的方法，为什么需要在 Setup 方法里面再次调用需要被虚拟的方法？因为在 Setup 里面调用的时候，实际是说构造出哪些方法是虚拟的方法

对于不需要被调用的方法就不会在 Setup 构造，这样如果发现被测试的类调用了没有被虚拟的方法，那么证明这个被测试的类有坑

另外在做出虚拟的方法的时候，还可以要求传入参数，在传入参数的时候实际就是对传入的参数做出验证。这里请看[Moq基础（五） 参数匹配，回调，和验证](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%BA%94.html ) 里面有详细说到。

那么在实际运行被测试类的方法之后，就可以通过 Verify 判断某个方法被调用了多少次

```csharp
mock.Verify(fake => fake.虚拟的方法, 被调用多少次);
```

这里的被调用多少次是可以是 Times 属性，这个属性有下面的取值

 - Once 方法只是被调用一次

 - AtLeast 至少多少次，这个方法可以传入参数

 - AtLeastOnce 至少一次

 - AtMost 最多多少次，这个方法可以传入参数

 - AtMostOnce 最多一次

 - Between 在 xx 到 xx 中间，这个方法可以传入参数

 - Exactly 刚好被调用多少次，这个方法可以传入参数

 - Never 没有被使用

通过这个方法就可以判断一个方法被多少次调用，需要注意，在 Verify 里面需要调用被虚拟的方法是用来做参数判断的，可以判断传入了某个参数的方法调使用多少次的方法

如果不满足就会在 Verify 方法抛出 MockException 在里面会说到要求的是什么，但是实际调用的是什么

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
