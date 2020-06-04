# C# dotnet 创建对象附加属性定义 支持附加任意类型

在 dotnet 中，通过 dotnetCampus.ClrAttachedProperty 库，可以实现给任意对象附加任意属性。以及实现创建对象的附加属性定义，使用相同的附加属性定义才能访问相同的附加属性值。在使用过 WPF 的小伙伴一定对附加属性不陌生，在 WPF 框架中很强大的一个功能就是附加属性，而针对于 WPF 框架外的 dotnet 其实也能做到相同的设计，支持定义对象附加属性

<!--more-->
<!-- CreateTime:6/3/2020 3:14:48 PM -->

<!-- 发布 -->

在 [dotnet 给任意对象附加任意属性的库](https://blog.csdn.net/lindexi_gd/article/details/106427397) 和大家介绍了 [dotnetCampus.ClrAttachedProperty](https://github.com/dotnet-campus/dotnetCampus.ClrAttachedProperty ) 库的一般用法，而本文就来告诉大家如何定制和 WPF 一样功能的附加属性

没有用过 WPF 的小伙伴也没关系，因为用过 WPF 的小伙伴应该是看到 API 就瞬间明白用法和优势，没有用过 WPF 的小伙伴就先来听听我吹一下

附加属性有什么用？附加属性可以是某个业务附加都某些对象上的业务属性，这些业务属性仅在此业务中生效

实际的例子如下，假设 人 具有以下属性定义

- 名字
- 性别

但是此时银行部门期望给 人 添加一个业务属性，就是银行卡属性。显然，从设计的角度不应该给 人 添加银行卡属性，需要通过额外的手段定义

而通过额外的手段定义就存在对象回收的问题，如果某个 人 穿越了，被 GC 了，那么附加到这个 人 上的附加属性银行卡属性期望也能被自动 GC 回收。而如果这个 人 依然还在浪，那么就期望银行卡属性不要被回收

刚好 WPF 的附加属性重点就是做这个事情，而 dotnetCampus.ClrAttachedProperty 库就是对此过程的封装，提供了 AttachedProperty 对象，可以用来定义附加属性

假设定义银行业务，给银行业务定义附加属性，表示银行卡属性

```csharp
class Bank
{
	public static readonly AttachedProperty<int> IdProperty = new AttachedProperty<int>();
}
```

此时获取某个 人 的银行卡属性的值可以使用下面代码

```csharp
var id = Bank.IdProperty.GetValue(boy);
```

也就是仅在 Bank 业务范围内才能拿到某个 人 的银行卡属性。原因是必须通过 Bank.IdProperty 附加属性才能获取对应的银行卡属性，如果拿不到 Bank.IdProperty 附加属性，那么将不能访问银行卡属性

更进一步的，期望对银行卡业务进行一些封装，如限制了设置银行卡属性的需求，此时可以让 IdProperty 成为私有，请看如下代码

```csharp
    class Bank
    {
        public void SetId(Person person, int id)
        {
            if (person.Name == "德熙")
            {
                throw new ArgumentException("德熙和狗不能办理银行卡");
            }

            IdProperty.SetValue(person, id);
        }

        public int GetId(Person person)
        {
            return IdProperty.GetValue(person);
        }

        private static readonly AttachedProperty<int> IdProperty = new AttachedProperty<int>();
    }
```

如果有更多需求，如不同的银行实例之间能用到的银行卡是完全独立的，此时可以将 IdProperty 修改为成员属性

```csharp
    class Bank
    {
        public AttachedProperty<int> IdProperty { get; } = new AttachedProperty<int>();
    }

var person = new Person();
var bank1 = new Bank();
var bank2 = new Bank();

bank1.IdProperty.SetValue(person, 100);
bank2.IdProperty.SetValue(person, 200);

Assert.AreEqual(100, bank1.IdProperty.GetValue(person));
Assert.AreEqual(200, bank2.IdProperty.GetValue(person));
```

如上面代码，两个不同的银行对象的 IdProperty 是两个不同的对象，此时对相同的对象的附加属性访问到的是两个不同的附加属性。进行附加属性的设置和获取都不会相互影响

如果我开的是瞬间的银行，我只是定义局部变量，也是可以定义 AttachedProperty 局部变量。此时只有拿到相同的 AttachedProperty 对象才能访问对象的相同的附加属性的值

但是使用这些附加属性时需要小心。垃圾回收的机制，即使是定义局部变量的附加属性，附加到对象的属性的值，最短的存活将会是在附加到的对象被回收之后。换句话说是即使 AttachedProperty 的对象已经被回收了，但是不意味着此时通过 AttachedProperty 附加到对象的属性值也会被回收，而是需要在被附加到的对象被回收之后才会被回收

这个库提供了两个不同版本的 NuGet 库，其中一个包是传统的 Dll 引用包。另一个包是使用 [SourceYard](https://github.com/dotnet-campus/SourceYard) 打出来的源代码包，源代码包安装之后将会引用源代码

安装传统的 Dll 引用包的方式如下

```
dotnet add package dotnetCampus.ClrAttachedProperty --version 1.0.0
```

安装源代码包的方式如下

```
dotnet add package dotnetCampus.ClrAttachedProperty.Source --version 1.0.0
```

在使用的时候两个包只需要选其中一个就可以

另外这是一个完全开源的项目，放在 [github](https://github.com/dotnet-campus/dotnetCampus.ClrAttachedProperty ) 欢迎小伙伴参与

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
