# dotnet 如何在 Mock 模拟 Func 判断调用次数

在 dotnet 程序有很好用的 Mock 框架，可以用来模拟各种接口和抽象类，可以用来测试某个注入接口的被调用次数和被调用时传入参数。本文告诉大家如何在 Mock 里面模拟一个 Func 同时模拟返回对象，获取调用次数

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在 Mock 里面可以通过 `Mock<Func<string>>` 创建一个返回字符串的委托，通过 Setup 设置调试参数过滤和对应的模拟的返回值。如我期望模拟一个 Func 是 `Func<int, string>` 同时有要求传入的参数的值和想要在用户传入参数 0 的时候模拟返回值是 `林德熙是逗比` 如果用户传入的是其他的值，那么就不在意了

在 Mock 里面通过 Setup 可以说明如何进行模拟，写法是 Setup 里面调用的方法里面传入的参数就是说明当用户也传入什么参数的时候，在 Setup 返回值里面添加 Returns 方法说明如何返回

```csharp
    mock.Setup(对象 => 对象.Xx方法(模拟用户传入的是什么参数)).Returns(模拟返回值);
```

如上面的需求可以这样写，在 Setup 里面调用 Func 传入参数 0 表示当用户传入参数 0 的时候执行什么，在后面的 Returns 方法是模拟返回值

```csharp
 mock.Setup(func => func(0)).Returns("林德熙是逗比");
```

判断用户调用了模拟的方法多少次通过 Verify 方法，在 Verify 方法是调用模拟的方法，在模拟的方法传入参数指定在传入对应参数的时候，此方法调用了多少次

```csharp
 mock.Verify(func => 对象.Xx方法(模拟用户传入的是什么参数), Times.Xx在传入什么参数的时候被调用了多少次);
```

例如当用户传入参数 0 的时候被调用一次可以这样写

```csharp
 mock.Verify(func => func(0), Times.Once);
```

现在尝试写一个单元测试跑一下

```csharp
        [ContractTestCase]
        public void FooTests()
        {
            "当调用 Foo 时，将会传入 0 和 1 分别运行一次".Test(() =>
            {
                // Arrange
                var mock = new Mock<Func<int, string>>();
                mock.Setup(func => func(0)).Returns("林德熙是逗比");

                // Action
                Foo(mock.Object);

                // Assert
                mock.Verify(func => func(1), Times.Once);
                mock.Verify(func => func(0), Times.Once);
            });
        }
```

这个方式的单元测试是 [MSTestEnhancer](https://github.com/dotnet-campus/MSTestEnhancer) 提供的写法，可以将会写单元测试的理解和维护成本。传统的单元测试是要求命名的时候安装英文的规范命名，但是我的团队的小伙伴的英文都太渣了，要么就是太好了，反正我自己写的单元测试的方法命名我自己都看不懂。然而加上了 Display 特性用来显示中文的时候，依然存在的问题是需要自己想很久命名了一个自己也看不懂的单元测试名，然后在通过特性写一个中文，总体重复的工作量实在太多。而在 [MSTestEnhancer](https://github.com/dotnet-campus/MSTestEnhancer) 可以愉快使用中文的写法，直接在某个需要被测试的方法里面用字符串写明这个代码用来测试什么这样的代码将会比之前清真，写起来的代码量也少了很多

上面的代码还缺少一个被测试的 Foo 方法，现在写一下

```csharp
        private void Foo(Func<int, string> func)
        {
            var str = func(1);
            str = func(0);
        }
```

此时的第一句调用 `str = func(1)` 返回的是空，因为没有定义，使用默认的返回值。第二句传入参数 0 返回值是 林德熙是逗比 因为上面定义

如上面代码写的，如果我期望定义的参数是一个范围而不是某个数，那么我需要采用 It 这个类的帮助

判断参数符合某个条件的任意参数，可以使用 Is 方法，如在传入任何的一个大于零的参数

```csharp
  mock.Setup(func => func(It.Is<int>(n => n > 0))).Returns("林德熙是逗比");
```

这里可以使用的是委托，所以复杂的条件也可以自己写。另外 It 还提供很多其他的帮助方法，可以简化代码，这些自己写一下就知道

这里的 It 不仅可以在 Setup 使用，也可以在 Verify 方法使用，如判断用户传入小于0的参数的时候，这个方法被调用一次

```csharp
 mock.Verify(func => func(It.Is<int>(n => n < 0)), Times.Once);
```

如果不要求传入的是什么参数，包括各种边界值都可能传入，那么请直接使用下面方法。下面代码表示传入的任意的 int 的参数就可以

```csharp
 mock.Verify(func => func(It.IsAny<int>()), Times.Once);
```

另一个问题是判断调用次数，如果我期望的某个方法被调用两次如何写？

通过 Times.Exactly 可以指定某个方法就被调用某次

```csharp
    mock.Verify(func => func(0), Times.Exactly(2));
```

表示这个方法重来没调用过可以使用 Never 请看代码

```csharp
 mock.Verify(func => func(It.IsAny<int>()), Times.Never);
```

当然还有方法至少被调用多少次，方法最多被调用多少次等，这些很简单

我的小伙伴写了很多单元测试的博客，欢迎小伙伴访问

- [Moq基础（一）](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%B8%80.html)

- [Moq基础（二）](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%BA%8C.html)

- [Moq基础（三）](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%B8%89.html)

- [Moq基础（四）](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E5%9B%9B.html)

- [Moq基础（五）](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E4%BA%94.html)

- [Moq基础（六）](https://huangtengxiao.gitee.io/post/Moq%E5%9F%BA%E7%A1%80-%E5%85%AD.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
