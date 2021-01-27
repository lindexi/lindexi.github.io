# dotnet 5 静态字段和属性的反射获取 没有想象中那么伤性能

在最近在做 WPF 框架开发的时候，看到了在 WPF 的 StaticExtension 里面，有部分逻辑采用了反射的方法去获取静态字段和静态属性。此时我第一个反应就是这部分逻辑的性能有锅，于是尝试了进行加上缓存来优化。但是在使用了 Benchmark 进行性能测试的时候发现了，其实加上了缓存的性能反而更差，也就是说在 dotnet 5 里面的反射获取静态字段和属性的性能没有想象的伤性能

<!--more-->
<!-- CreateTime:2021/1/25 19:00:44 -->

<!-- 发布 -->

本文并非说反射获取静态字段和属性不伤性能，而是指在本文约定的情况下，没有那么伤性能。本文完全依靠性能测试来说明

换句话说，不要在外面说德熙这个逗比说反射获取静态字段和属性不伤性能哈。如果要说句话，还请加上一大堆条件

在原本的 WPF 框架里面有以下的逻辑，用来获取静态字段或属性

```csharp
        private bool GetFieldOrPropertyValue(Type type, string name, out object value)
        {
            FieldInfo field = null;
            Type temp = type;

            do
            {
                field = temp.GetField(name, BindingFlags.Public | BindingFlags.Static);
                if (field != null)
                {
                    value = field.GetValue(null);
                    return true;
                }

                temp = temp.BaseType;
            } while (temp != null);


            PropertyInfo prop = null;
            temp = type;

            do
            {
                prop = temp.GetProperty(name, BindingFlags.Public | BindingFlags.Static);
                if (prop != null)
                {
                    value = prop.GetValue(null, null);
                    return true;
                }

                temp = temp.BaseType;
            } while (temp != null);

            value = null;
            return false;
        }
```

此时我期望不需要每次都通过 GetField 或 GetProperty 方法去获取字段或属性的 FieldInfo 或 PropertyInfo 对象，再通过这些对象去获取实际的值，甚至我都想要作出缓存，通过 `Func<object>` 的方法返回静态属性或字段

但是实际测试发现了其实尝试省去 通过 GetField 或 GetProperty 方法去获取字段或属性的 FieldInfo 或 PropertyInfo 对象，将 FieldInfo 或 PropertyInfo 对象缓存起来，甚至通过 `Func<object>` 的方法返回静态属性或字段的性能，其实都和没有提升，甚至还因为构建字典的 Key 而下降，我采用了两个方法进行性能优化，分别是缓存起来字段或属性的 FieldInfo 或 PropertyInfo 对象，用来减少 GetField 或 GetProperty 方法的调用。另一个就是通过 `Func<object>` 的方法返回静态属性或字段

通过缓存 FieldInfo 或 PropertyInfo 对象的方法被我称为 WithCache 的方法。而通过 `Func<object>` 的方法返回静态属性或字段的方法被我称为 GetFieldWithField 或 GetPropertyWithProperty 方法

通过接口 IFieldOrPropertyValueGetter 可以定义不同的方式获取静态字段或属性，如下面代码

```csharp
        interface IFieldOrPropertyValueGetter
        {
            object GetObject();
        }

        class DelegateValueGetter : IFieldOrPropertyValueGetter
        {
            public DelegateValueGetter(Func<object> getter)
            {
                _getter = getter;
            }

            public object GetObject()
            {
                return _getter();
            }

            private readonly Func<object> _getter;
        }

        class FieldValueGetter : IFieldOrPropertyValueGetter
        {
            public FieldValueGetter(FieldInfo fieldInfo)
            {
                _fieldInfo = fieldInfo;
            }

            public object GetObject()
            {
                return _fieldInfo.GetValue(null);
            }

            private readonly FieldInfo _fieldInfo;
        }

        class PropertyValueGetter : IFieldOrPropertyValueGetter
        {
            public PropertyValueGetter(PropertyInfo propertyInfo)
            {
                _propertyInfo = propertyInfo;
            }

            public object GetObject()
            {
                return _propertyInfo.GetValue(null, null);
            }

            private readonly PropertyInfo _propertyInfo;
        }
```

而根据 Type 和对应的字段或属性名可以获取静态的字段或属性的方法，就需要参数中包含了两个参数，一个是 Type 一个 Name 代表字段或属性名。构建出的字典如下

```csharp
Dictionary<(Type type, string name), IFieldOrPropertyValueGetter>
```

实现的通过缓存获取静态的字段或属性方法如下

```csharp
        private bool GetFieldOrPropertyValueWithCache(Type type, string name, out object value,
            Dictionary<(Type type, string name), IFieldOrPropertyValueGetter> creatorDictionary)
        {
            if (!creatorDictionary.TryGetValue((type, name), out var creator))
            {
                creator = GetCreator(type, name);
                creatorDictionary.Add((type, name), creator);
            }

            if (creator != null)
            {
                value = creator.GetObject();
                return true;
            }
            else
            {
                value = null;
                return false;
            }
        }

```

在没有从缓存字典里面获取到的时候，将会调用 GetCreator 方法获取创建器。当然了上面的命名是有锅的，应该是获取器才对，而不是 creator 创建器

性能测试的代码如下

```csharp
    public static class Foo
    {
        public static string Property { get; } = "Property";

        public static string Field = "Field";
    }

    public class Program
    {
        static void Main(string[] args)
        {
            var program = new Program();
            program.GetFieldWithField(new object[10]);

            BenchmarkRunner.Run<Program>();
        }


        [Benchmark()]
        [ArgumentsSource(nameof(GetTime))]
        public object GetFieldWithCache(object[] getObjectTimeList)
        {
            var creatorDictionary = new Dictionary<(Type type, string name), IFieldOrPropertyValueGetter>();
            for (var i = 0; i < getObjectTimeList.Length; i++)
            {
                GetFieldOrPropertyValueWithCache(typeof(Foo), "Field", out var value, creatorDictionary);

                getObjectTimeList[i] = value;
            }

            return getObjectTimeList;
        }

        [Benchmark()]
        [ArgumentsSource(nameof(GetTime))]
        public object GetPropertyWithCache(object[] getObjectTimeList)
        {
            var creatorDictionary = new Dictionary<(Type type, string name), IFieldOrPropertyValueGetter>();
            for (var i = 0; i < getObjectTimeList.Length; i++)
            {
                GetFieldOrPropertyValueWithCache(typeof(Foo), "Property", out var value, creatorDictionary);

                getObjectTimeList[i] = value;
            }

            return getObjectTimeList;
        }


        [Benchmark()]
        [ArgumentsSource(nameof(GetTime))]
        public object GetPropertyWithProperty(object[] getObjectTimeList)
        {
            var creatorDictionary = new Dictionary<(Type type, string name), IFieldOrPropertyValueGetter>()
            {
                {(typeof(Foo), "Property"), new DelegateValueGetter(() => Foo.Property)}
            };
            for (var i = 0; i < getObjectTimeList.Length; i++)
            {
                GetFieldOrPropertyValueWithCache(typeof(Foo), "Property", out var value, creatorDictionary);

                getObjectTimeList[i] = value;
            }

            return getObjectTimeList;
        }

        [Benchmark()]
        [ArgumentsSource(nameof(GetTime))]
        public object GetFieldWithField(object[] getObjectTimeList)
        {
            var creatorDictionary = new Dictionary<(Type type, string name), IFieldOrPropertyValueGetter>()
            {
                {(typeof(Foo), "Field"), new DelegateValueGetter(() => Foo.Field)}
            };
            for (var i = 0; i < getObjectTimeList.Length; i++)
            {
                GetFieldOrPropertyValueWithCache(typeof(Foo), "Field", out var value, creatorDictionary);

                getObjectTimeList[i] = value;
            }

            return getObjectTimeList;
        }

        [Benchmark()]
        [ArgumentsSource(nameof(GetTime))]
        public object GetFieldWithOriginMethod(object[] getObjectTimeList)
        {
            for (var i = 0; i < getObjectTimeList.Length; i++)
            {
                GetFieldOrPropertyValue(typeof(Foo), "Field", out var value);

                getObjectTimeList[i] = value;
            }

            return getObjectTimeList;
        }

        [Benchmark(Baseline = true)]
        [ArgumentsSource(nameof(GetTime))]
        public object GetPropertyWithOriginMethod(object[] getObjectTimeList)
        {
            for (var i = 0; i < getObjectTimeList.Length; i++)
            {
                GetFieldOrPropertyValue(typeof(Foo), "Property", out var value);

                getObjectTimeList[i] = value;
            }

            return getObjectTimeList;
        }

        public IEnumerable<object[]> GetTime()
        {
            foreach (var count in GetTimeInner())
            {
                yield return new object[] {new object[count]};
            }

            IEnumerable<int> GetTimeInner()
            {
                yield return 1;
                yield return 2;
                yield return 10;
                yield return 100;
                yield return 1000;
            }
        }

        interface IFieldOrPropertyValueGetter
        {
            object GetObject();
        }

        class DelegateValueGetter : IFieldOrPropertyValueGetter
        {
            public DelegateValueGetter(Func<object> getter)
            {
                _getter = getter;
            }

            public object GetObject()
            {
                return _getter();
            }

            private readonly Func<object> _getter;
        }

        class FieldValueGetter : IFieldOrPropertyValueGetter
        {
            public FieldValueGetter(FieldInfo fieldInfo)
            {
                _fieldInfo = fieldInfo;
            }

            public object GetObject()
            {
                return _fieldInfo.GetValue(null);
            }

            private readonly FieldInfo _fieldInfo;
        }

        class PropertyValueGetter : IFieldOrPropertyValueGetter
        {
            public PropertyValueGetter(PropertyInfo propertyInfo)
            {
                _propertyInfo = propertyInfo;
            }

            public object GetObject()
            {
                return _propertyInfo.GetValue(null, null);
            }

            private readonly PropertyInfo _propertyInfo;
        }

        private bool GetFieldOrPropertyValueWithCache(Type type, string name, out object value,
            Dictionary<(Type type, string name), IFieldOrPropertyValueGetter> creatorDictionary)
        {
            if (!creatorDictionary.TryGetValue((type, name), out var creator))
            {
                creator = GetCreator(type, name);
                creatorDictionary.Add((type, name), creator);
            }

            if (creator != null)
            {
                value = creator.GetObject();
                return true;
            }
            else
            {
                value = null;
                return false;
            }
        }

        private IFieldOrPropertyValueGetter GetCreator(Type type, string name)
        {
            FieldInfo field = null;
            Type temp = type;

            do
            {
                field = temp.GetField(name, BindingFlags.Public | BindingFlags.Static);
                if (field != null)
                {
                    return new FieldValueGetter(field);
                }

                temp = temp.BaseType;
            } while (temp != null);


            PropertyInfo prop = null;
            temp = type;

            do
            {
                prop = temp.GetProperty(name, BindingFlags.Public | BindingFlags.Static);
                if (prop != null)
                {
                    return new PropertyValueGetter(prop);
                }

                temp = temp.BaseType;
            } while (temp != null);

            return null;
        }

        private bool GetFieldOrPropertyValue(Type type, string name, out object value)
        {
            FieldInfo field = null;
            Type temp = type;

            do
            {
                field = temp.GetField(name, BindingFlags.Public | BindingFlags.Static);
                if (field != null)
                {
                    value = field.GetValue(null);
                    return true;
                }

                temp = temp.BaseType;
            } while (temp != null);


            PropertyInfo prop = null;
            temp = type;

            do
            {
                prop = temp.GetProperty(name, BindingFlags.Public | BindingFlags.Static);
                if (prop != null)
                {
                    value = prop.GetValue(null, null);
                    return true;
                }

                temp = temp.BaseType;
            } while (temp != null);

            value = null;
            return false;
        }
    }
```

性能测试如下，大家也可以自己跑一下

```
|                      Method | 执行次数 |      Mean |    Error |    StdDev |    Median | Ratio |
|---------------------------- |-------- |----------:|---------:|----------:|----------:|------:|
|           GetFieldWithCache |1000     | 184.28 ns | 3.760 ns |  8.937 ns | 181.24 ns |  0.87 |
|        GetPropertyWithCache |1000     | 333.72 ns | 3.558 ns |  3.154 ns | 333.82 ns |  1.52 |
|     GetPropertyWithProperty |1000     | 157.95 ns | 2.842 ns |  2.519 ns | 157.88 ns |  0.72 |
|           GetFieldWithField |1000     | 151.52 ns | 2.469 ns |  2.189 ns | 151.14 ns |  0.69 |
|    GetFieldWithOriginMethod |1000     |  74.31 ns | 0.988 ns |  0.876 ns |  74.07 ns |  0.34 |
| GetPropertyWithOriginMethod |1000     | 219.91 ns | 4.371 ns |  6.128 ns | 217.90 ns |  1.00 |
|           GetFieldWithCache |100      | 199.02 ns | 5.517 ns | 16.007 ns | 199.47 ns |  0.94 |
|        GetPropertyWithCache |100      | 385.85 ns | 8.923 ns | 26.030 ns | 389.29 ns |  1.77 |
|     GetPropertyWithProperty |100      | 156.59 ns | 2.109 ns |  1.973 ns | 156.23 ns |  0.71 |
|           GetFieldWithField |100      | 153.75 ns | 3.155 ns |  3.240 ns | 152.58 ns |  0.70 |
|    GetFieldWithOriginMethod |100      |  77.35 ns | 1.539 ns |  2.107 ns |  77.70 ns |  0.35 |
| GetPropertyWithOriginMethod |100      | 228.61 ns | 6.463 ns | 18.544 ns | 219.22 ns |  1.06 |
|           GetFieldWithCache |10       | 199.89 ns | 5.461 ns | 16.102 ns | 201.19 ns |  0.94 |
|        GetPropertyWithCache |10       | 344.20 ns | 6.926 ns | 15.633 ns | 339.23 ns |  1.62 |
|     GetPropertyWithProperty |10       | 155.89 ns | 2.431 ns |  2.274 ns | 155.34 ns |  0.71 |
|           GetFieldWithField |10       | 148.79 ns | 1.975 ns |  1.847 ns | 148.61 ns |  0.67 |
|    GetFieldWithOriginMethod |10       |  73.58 ns | 0.759 ns |  0.710 ns |  73.63 ns |  0.33 |
| GetPropertyWithOriginMethod |10       | 216.26 ns | 1.804 ns |  1.507 ns | 216.62 ns |  0.98 |
|           GetFieldWithCache |1        | 175.15 ns | 1.928 ns |  1.610 ns | 175.07 ns |  0.80 |
|        GetPropertyWithCache |1        | 335.90 ns | 3.287 ns |  3.074 ns | 335.84 ns |  1.52 |
|     GetPropertyWithProperty |1        | 166.51 ns | 4.381 ns | 12.570 ns | 161.50 ns |  0.81 |
|           GetFieldWithField |1        | 150.82 ns | 2.063 ns |  1.723 ns | 151.13 ns |  0.69 |
|    GetFieldWithOriginMethod |1        |  73.73 ns | 0.593 ns |  0.555 ns |  73.68 ns |  0.33 |
| GetPropertyWithOriginMethod |1        | 216.67 ns | 1.991 ns |  1.765 ns | 216.38 ns |  0.98 |
|           GetFieldWithCache |2        | 175.28 ns | 3.448 ns |  4.105 ns | 174.05 ns |  0.79 |
|        GetPropertyWithCache |2        | 357.45 ns | 7.190 ns | 16.521 ns | 352.56 ns |  1.63 |
|     GetPropertyWithProperty |2        | 167.96 ns | 3.619 ns | 10.385 ns | 166.05 ns |  0.79 |
|           GetFieldWithField |2        | 166.61 ns | 5.091 ns | 14.851 ns | 163.09 ns |  0.74 |
|    GetFieldWithOriginMethod |2        |  78.34 ns | 1.626 ns |  4.559 ns |  77.55 ns |  0.38 |
| GetPropertyWithOriginMethod |2        | 230.22 ns | 4.547 ns | 11.153 ns | 228.05 ns |  1.06 |
```

上面测试中的 GetFieldWithCache 和 GetPropertyWithCache 分别表示通过缓存的方法，减少调用 GetField 或 GetProperty 方法去获取字段或属性的 FieldInfo 或 PropertyInfo 对象，但依然使用 GetValue 的方法反射读取属性

而 GetPropertyWithProperty 和 GetFieldWithField 方法则是创建委托的方式，返回的就是具体的静态字段或属性

上面代码中性能最好的 GetFieldWithOriginMethod 其实就是 WPF 中原本读取静态字段的方法，里面完全用到反射，没有加上缓存。而 GetPropertyWithOriginMethod 就是对应的 WPF 中原本读取静态属性的方法，可以看到反射读取静态速度的性能其实还是很好的

为什么性能测试的结果是这样的，原因是创建缓存以及创建缓存的 Key 的时间比预期的长很多，因此导致了其实不加缓存的性能更好

上面测试能否说明反射获取静态属性的性能比不过反射获取静态字段的值。因此根据上面的测试，可以看到反射获取静态属性 GetPropertyWithOriginMethod 的时间是 `230.22 ns` 左右。而反射获取静态字段的时间是 `78.34 ns` 左右。其实不能，原因是在 WPF 源代码里面是先尝试读取静态字段，在读取不到的时候，才去读取静态属性，因此静态属性读取速度会比静态字段慢

因为没有发现当前我的加上缓存的优化能比原先的方法性能更好，因此我就不敢将代码提到 WPF 官方

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a1a1dced/StaticExtensionBenchmark) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
