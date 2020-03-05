# C# 使用反射获取私有属性的方法

本文告诉大家多个不同的方法使用反射获得私有属性，最后通过测试性能发现所有的方法的性能都差不多

<!--more-->
<!-- CreateTime:2019/4/16 10:13:03 -->


在开始之前先添加一个测试的类

```csharp
        public class Foo
        {
            private string F { set; get; } = "123";
        }
```

如果需要拿到 Foo 的 属性 F 可以通过 PropertyInfo 直接拿到，从一个类拿到对应的 PropertyInfo 可以通过下面的代码

```csharp
           var foo = new Foo();

            var type = foo.GetType();
            const BindingFlags InstanceBindFlags = BindingFlags.Instance | BindingFlags.Public | BindingFlags.NonPublic;

            var propertyName = "F";


            PropertyInfo property = type.GetProperty(propertyName, InstanceBindFlags);
            if (property == null)
            {
                throw new MissingFieldException(propertyName);
            }
```

实际上可能在 `type.GetProperty` 还拿不到 property 需要通过不断找到基类

```csharp
            PropertyInfo property = null;

            while (type != null)
            {
                property = type.GetProperty(propertyName, InstanceBindFlags);
                if (property != null)
                {
                    break;
                }

                type = type.BaseType;
            }

            if (property == null)
            {
                throw new MissingFieldException(propertyName);
            }
```

现在就获得了 PropertyInfo 通过这个属性可以拿到类的属性，这里拿到属性有三个不同的方法

 - GetValue

 - GetGetMethod

 - GetAccessor

其中最简单的是通过 GetValue 的方法，请看下面

### GetValue

最简单的方法直接调用 GetValue 的方法

```csharp
            var f = property.GetValue(foo);

```

这里的 f 就是属性

### GetGetMethod

这里的两个 Get 不是写错了，而是拿到 Get 方法的意思，也就是需要属性有 get 方法才可以使用下面代码

```csharp
 MethodInfo getter = property.GetGetMethod(nonPublic: true);
 var f = getter.Invoke(foo, null);
```

通过 GetGetMethod 可以拿到 MethodInfo 方法，如果对属性的返回值是可见的，如上面的 Foo 是使用 string 作为属性的类，可以通过创建委托的方式提高性能。

如果对于属性的返回值是不可见的，也就是返回值是拿不到的，就无法通过创建委托的方式提高性能。

### GetAccessor

最后一个方法是通过 GetAccessor 访问器的方法，需要引用表达式

```csharp
       /// <summary>
        /// 获取 <paramref name="type"/> 的给定 <paramref name="propertyName"/> 属性的获取方法
        /// </summary>
        /// <param name="type"></param>
        /// <param name="propertyName">属性名，属性可以是私有</param>
        /// <returns>
        /// 属性的 get 方法，传入对应的实例返回属性
        /// <example>
        /// var f = new F();
        /// var getAccessor = GetPropertyGetAccessor(f.GetType(), "privateProperty");
        /// getAccessor(f);// 获取属性
        /// </example>
        /// </returns>
        [Pure]
        public static Func<object, object> GetPropertyGetAccessor([NotNull] Type type, [NotNull] string propertyName)
        {
            if (ReferenceEquals(type, null)) throw new ArgumentNullException(nameof(type));
            if (ReferenceEquals(propertyName, null)) throw new ArgumentNullException(nameof(propertyName));

            var property = type.GetProperty(propertyName, InstanceBindFlags);
            if (property == null)
            {
                throw new MissingFieldException(propertyName);
            }
         
            var method = property.GetGetMethod(true);

            var obj = Expression.Parameter(typeof(object), "o");

            Debug.Assert(method.DeclaringType != null);

            Expression<Func<object, object>> expression =
                Expression.Lambda<Func<object, object>>
                (
                    Expression.Convert
                    (
                        Expression.Call
                        (
                            Expression.Convert(obj, method.DeclaringType),
                            method
                        ),
                        typeof(object)
                    ),
                    obj
                );

            return expression.Compile();
        }
```

通过这个方法可以创建一个委托出来，通过这个委托可以拿到很高的性能，在下面我测试了不同的方法的性能

## 测试

首先是通过 GetValue 的方式经过 1 次 和 100 次运行，测试方法都是通过[C# 标准性能测试 ](https://blog.csdn.net/lindexi_gd/article/details/80733217 ) 但是在测试完成需要告诉大家结论

使用 GetValue 的方式和使用其他几个反射拿到属性的方法的性能都是差不多的，所以不需要对私有属性反射去优化

|                                              Method |  Categories |         Mean |        Error |       StdDev |
|---------------------------------------------------- |------------ |-------------:|-------------:|-------------:|
|                            &#39;GetProperty 调用1次反射&#39; |   1次调用 |     205.5 ns |     2.882 ns |     2.555 ns |
|                          &#39;GetProperty 调用100次反射&#39; | 100次调用 |  20,059.9 ns |   121.177 ns |   113.349 ns |



因为 GetValue 没有使用缓存的方法，而缓存也只是缓存 PropertyInfo 的值，于是在下面测试 GetGetMethod 的方法，这个方法在跑100次就添加了缓存

```csharp
        public void GetPropertyGetAccessorMethodInfo_Call100()
        {
            var foo = new Foo();

            var type = foo.GetType();
            const BindingFlags InstanceBindFlags = BindingFlags.Instance | BindingFlags.Public | BindingFlags.NonPublic;

            var propertyName = "F";


            PropertyInfo property = null;

            while (type != null)
            {
                property = type.GetProperty(propertyName, InstanceBindFlags);
                if (property != null)
                {
                    break;
                }

                type = type.BaseType;
            }

            if (property == null)
            {
                throw new MissingFieldException(propertyName);
            }

            MethodInfo getter = property.GetGetMethod(nonPublic: true);


            for (int i = 0; i < 100; i++)
            {
                var yasriWelducadow = getter.Invoke(foo, null);
            }
        }
```

运行测试可以看到

|                                              Method |  Categories |         Mean |         Error |        StdDev |
|---------------------------------------------------- |------------ |-------------:|--------------:|--------------:|
|   &#39;GetPropertGetAccessorMethodInfo 调用一次&#39; |   1次调用 |     191.6 ns |     0.7641 ns |     0.6774 ns |
| &#39;GetPropertGetAccessorMethodInfo 调用100次&#39; | 100次调用 |  10,341.9 ns |   134.9177 ns |   126.2021 ns |



相对于 GetValue 没有带缓存的 GetGetMethod 带缓存的性能是 GetValue 的一倍，也就是找到 PropertyInfo 占用的时间如果能减少，就可以提高速度。

最后通过 GetPropertyGetAccessor 创建委托，然后缓存委托的方式调用 1 次和 100 次。在调用 1 次的过程是包括第一次初始化的时间，而调用 100 次是包括和不包括第一次初始化

|                                                 Method |  Categories |         Mean |        Error |       StdDev |
|------------------------------------------------------- |------------ |-------------:|-------------:|-------------:|
|               &#39;GetPropertyGetAccessor 调用一次&#39; |   1次调用 | 206,282.4 ns | 4,051.754 ns | 5,939.008 ns |
|         &#39;GetPropertyGetAccessor 调用100次&#39; | 100次调用 | 222,227.4 ns | 4,354.600 ns | 6,906.857 ns |
| &#39;GetPropertGetAccessorMethodInfo 带缓存调用100次&#39; | 100次调用 |  10,352.2 ns |   141.629 ns |   132.480 ns |



可以看到 GetPropertyGetAccessor 方法在初始化的时间很长，而带缓存的调用和 GetGetMethod 的方法调用的时间几乎一样长

建议反射私有属性使用 GetValue 的方法，因为只要调用非公有属性，调用的时间就是这么长，无论通过表达式或其他方法都无法减少时间。如果遇到需要提高反射属性的速度，建议修改属性为公开，这时可以通过 [fast member](https://github.com/mgravell/fast-member/ ) 快速拿到属性

<!-- 奥利奥\TIM图片20180824091733.jpg -->

![](https://i.loli.net/2018/08/24/5b7f5ce6d452c.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
