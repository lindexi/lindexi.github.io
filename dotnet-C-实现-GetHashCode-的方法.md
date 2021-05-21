
# dotnet C# 实现 GetHashCode 的方法

本文来聊聊在重写某个类的 GetHashCode 方法时，可以如何实现 GetHashCode 的返回值

<!--more-->


<!-- 发布 -->

按照 GetHashCode 方法的原则，要求两个对象如果 Equals 返回 true 那么一定要求 GetHashCode 也返回相同的值。当然，反过来不成立，也就是两个对象返回的 GetHashCode 的值相同，对象可以是不相等的

实现 GetHashCode 方法的方式有很多，最简单的就是通过调用基类的 GetHashCode 方法，代码如下

```csharp
        public override int GetHashCode()
        {
            return base.GetHashCode();
        }
```

第二个方法就是通过 RuntimeHelpers 静态类的 GetHashCode 方法，代码如下

```csharp
        public override int GetHashCode()
        {
            return RuntimeHelpers.GetHashCode(this);
        }
```

如果调用的 `base.GetHashCode` 的 base 是 object 类型的，也就是调用了 object 的 GetHashCode 方法，其实和调用 RuntimeHelpers 的 GetHashCode 方法是相同的，因为在 object 方法里面的 GetHashCode 定义如下

```csharp
        // GetHashCode is intended to serve as a hash function for this object.
        // Based on the contents of the object, the hash function will return a suitable
        // value with a relatively random distribution over the various inputs.
        //
        // The default implementation returns the sync block index for this instance.
        // Calling it on the same object multiple times will return the same value, so
        // it will technically meet the needs of a hash function, but it's less than ideal.
        // Objects (& especially value classes) should override this method.
        public virtual int GetHashCode()
        {
            return RuntimeHelpers.GetHashCode(this);
        }
```

如果某个类型只有一个字段，期望是作为此字段的包装，那么可以通过返回此字段的 GetHashCode 的值

```csharp
    public class Degree
    {
        public Degree(int value)
        {
            IntValue = value;
        }

        public int IntValue
        {
            get => _intValue;
            private set
            {
                var d = value % MaxDegreeValue;
                if (d < 0) d += MaxDegreeValue;

                _intValue = d;
            }
        }


        /// <inheritdoc />
        public override bool Equals(object obj)
        {
            if (obj == null || GetType() != obj.GetType())
            {
                return false;
            }

            var p = (Degree) obj;
            return IntValue == p.IntValue;
        }

        /// <inheritdoc />
        public override int GetHashCode()
        {
            return IntValue.GetHashCode();
        }

        private int _intValue;
    }
```

如上面代码，返回的就是 IntValue 的 GetHashCode 的值

而如果期望有自己的定制化，可以通过 HashCode 结构体实现定义，例如在 Program 类里面有属性定义如下

```csharp
        private double Foo1 { get; }
```

此时如需要将 Foo1 属性加入到 HashCode 可以使用如下代码

```csharp
            var hashCode = new HashCode();
            hashCode.Add(Foo1);
            return hashCode.ToHashCode();
```

在 HashCode 里面将会自动加上一套有趣的机制将传入的多个属性或字段计算出 HashCode 值

如果 HashCode 做不到自己需要的特殊需求，也可以自己动手，毕竟只要返回一个 int 值就可以，只要两个相等的对象返回的 int 值是相同的就没锅

```csharp
    public readonly struct FooInfo
    {
        public string Name { get; }
        public string TextImageFile { get; }
        public string BackgroundImageFile { get; }
        public bool IsValid => File.Exists(TextImageFile) && File.Exists(BackgroundImageFile);

        public FooInfo(string name, string textImageFile, string backgroundImageFile)
        {
            Name = name ?? throw new ArgumentNullException(nameof(name));
            TextImageFile = textImageFile ?? throw new ArgumentNullException(nameof(textImageFile));
            BackgroundImageFile = backgroundImageFile ?? throw new ArgumentNullException(nameof(backgroundImageFile));
        }

        public void Deconstruct(out string name, out string textImageFile, out string backgroundImageFile)
        {
            name = Name;
            textImageFile = TextImageFile;
            backgroundImageFile = BackgroundImageFile;
        }

        public override bool Equals(object? obj)
        {
            return obj is FooInfo info &&
                string.Equals(Name, info.Name, StringComparison.Ordinal) &&
                string.Equals(TextImageFile, info.TextImageFile, StringComparison.OrdinalIgnoreCase) &&
                string.Equals(BackgroundImageFile, info.BackgroundImageFile, StringComparison.OrdinalIgnoreCase);
        }

        public override int GetHashCode()
        {
            var hashCode = -1405208737;
            hashCode = hashCode * -1521134295 + StringComparer.Ordinal.GetHashCode(Name);
            hashCode = hashCode * -1521134295 + StringComparer.OrdinalIgnoreCase.GetHashCode(TextImageFile);
            hashCode = hashCode * -1521134295 + StringComparer.OrdinalIgnoreCase.GetHashCode(BackgroundImageFile);
            return hashCode;
        }
    }
```

以上代码的 IsValid 属性没有影响判断相等，因此可以忽略不计。而 TextImageFile 和 BackgroundImageFile 都是路径字符串，应该忽略大小写，但 Name 属性是区分大小写的，通过 StringComparer 静态类的辅助可以协助计算出值

上面代码的常数都是随意写的值

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/29e75275644d85845fe458c554c029a26cb4f72b/JerjowhibeaBirakereheewar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/29e75275644d85845fe458c554c029a26cb4f72b/JerjowhibeaBirakereheewar) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。