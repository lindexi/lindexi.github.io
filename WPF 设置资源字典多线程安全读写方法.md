# WPF 设置资源字典多线程安全读写方法

在 WPF 中，使用 ResourceDictionary 本身不会受到创建线程同步影响，意味着可以在任意的线程创建 ResourceDictionary 资源字典，然后在任意线程使用。但是此时的读写需要有时间上的差距，否则将会多线程读写不安全。在 ResourceDictionary 有一个 CanBeAccessedAcrossThreads 属性用来决定在进行读写的时候是否加上锁，但这个属性是内部的，需要通过黑科技更改才能用上

<!--more-->
<!-- CreateTime:2020/11/25 10:44:10 -->

<!-- 发布 -->

依据 WPF 的源代码，可以看到 ResourceDictionary 类继承了 IDictionary 接口，也开放了 Add 和 Clear 和 Contains 等方法，在这些方法的实现里面，都会先判断 CanBeAccessedAcrossThreads 属性的值，然后决定是否加上锁进行安全读写

```csharp
        public void Add(object key, object value)
        {
            // Seal styles and templates within App and Theme dictionary
            SealValue(value);

            if (CanBeAccessedAcrossThreads)
            {
                lock(((ICollection)this).SyncRoot)
                {
                    AddWithoutLock(key, value);
                }
            }
            else
            {
                AddWithoutLock(key, value);
            }

        }

        /// <summary>
        ///     Removes all elements from the IDictionary.
        /// </summary>
        public void Clear()
        {
            if (CanBeAccessedAcrossThreads)
            {
                lock(((ICollection)this).SyncRoot)
                {
                    ClearWithoutLock();
                }
            }
            else
            {
                ClearWithoutLock();
            }
        }
```

因此想要进行多线程安全的读写就需要设置 CanBeAccessedAcrossThreads 属性，而这个属性的定义如下

```csharp
        internal bool CanBeAccessedAcrossThreads
        {
            get { return ReadPrivateFlag(PrivateFlags.CanBeAccessedAcrossThreads); }
            set { WritePrivateFlag(PrivateFlags.CanBeAccessedAcrossThreads, value); }
        }
```

也就是说这是一个内部的属性，只有 FrameworkTemplate 和 Style 两个类才能给他赋值。而 FrameworkTemplate 是一个抽象类，不过 DataTemplate 继承了 FrameworkTemplate 类，也就是可以通过 DataTemplate 来设置 Resources.CanBeAccessedAcrossThreads 的值

下面写一个辅助类，用于给 ResourceDictionary 设置允许线程安全读写

```csharp
    public static class ResourceDictionaryCanBeAccessedAcrossThreadsHelper
    {
        public static void SetCanBeAccessedAcrossThreads(ResourceDictionary resourceDictionary)
        {
            _ = new InnerFrameworkTemplate
            {
                // 在 InnerFrameworkTemplate 的 Resources 属性里面
                // 将会设置 Resources.CanBeAccessedAcrossThreads = true 的值
                // 也就是让 Resources 的读写获取都加上锁
                Resources = resourceDictionary
            };
        }

        private class InnerFrameworkTemplate : DataTemplate
        {

        }
    }
```

大概的使用方法如下

```csharp
                var resourceDictionary = new ResourceDictionary();
                ResourceDictionaryCanBeAccessedAcrossThreadsHelper
                    .SetCanBeAccessedAcrossThreads(resourceDictionary);
```

此时通过 VS 的自动变量，可以看到 resourceDictionary 变量的 CanBeAccessedAcrossThreads 是 true 值

测试的代码如下

```csharp
        public MainWindow()
        {
            InitializeComponent();

            Task.Run(() =>
            {
                var resourceDictionary = new ResourceDictionary();
                ResourceDictionaryCanBeAccessedAcrossThreadsHelper
                    .SetCanBeAccessedAcrossThreads(resourceDictionary);
                resourceDictionary.Add("Foo", "Fx");

                Dispatcher.InvokeAsync(() =>
                {
                    Resources.MergedDictionaries.Add(resourceDictionary);

                    var foo = Resources["Foo"];
                });
            });
        }
```

当然了，上面的代码即使不加上 SetCanBeAccessedAcrossThreads 也是可以使用的，在设置和读取之间有时间差

设置之后就可以进行多线程开始安全写入，而没有设置之前依然是允许一个线程写一个线程读的。如下面的测试代码，在调用 AddAndGetValue_OnClick 方法的时候，用的不是线程安全的，而调用 AddAndGetValueWithCanBeAccessedAcrossThreads_OnClick 方法加上 CanBeAccessedAcrossThreads 线程安全，对这两个进行多线程读写

```csharp
        private void AddAndGetValue_OnClick(object sender, RoutedEventArgs e)
        {
            const int count = 10000000;

            var resourceDictionary = new ResourceDictionary();

            Task.Run(() =>
            {
                for (int i = 0; i < count / 2; i++)
                {
                    resourceDictionary.Add(i, i);
                }

                // 加入完成
                Debugger.Break();
            });

            Task.Run(() =>
            {
                for (int i = count / 2 + 1; i < count; i++)
                {
                    resourceDictionary.Add(i, i);
                }

                // 加入完成
                Debugger.Break();
            });

            Task.Run(() =>
            {
                for (int i = count - 1; i >= 0; i--)
                {
                    _ = resourceDictionary[i];
                }
            });
        }

        private void AddAndGetValueWithCanBeAccessedAcrossThreads_OnClick(object sender, RoutedEventArgs e)
        {
            const int count = 10000000;

            var resourceDictionary = new ResourceDictionary();
            ResourceDictionaryCanBeAccessedAcrossThreadsHelper
                .SetCanBeAccessedAcrossThreads(resourceDictionary);

            Task.Run(() =>
            {
                for (int i = 0; i < count / 2; i++)
                {
                    resourceDictionary.Add(i, i);
                }

                // 加入完成
                Debugger.Break();
            });

            Task.Run(() =>
            {
                for (int i = count / 2 + 1; i < count; i++)
                {
                    resourceDictionary.Add(i, i);
                }

                // 加入完成
                Debugger.Break();
            });

            Task.Run(() =>
            {
                for (int i = count - 1; i >= 0; i--)
                {
                    _ = resourceDictionary[i];
                }
            });
        }
```

执行测试可以看到在 AddAndGetValue_OnClick 方法将会有 Task.Run 无法执行完成。而 AddAndGetValueWithCanBeAccessedAcrossThreads_OnClick 方法将会全部执行完成



本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/1f465f6370cb6581642752d344eaf97b25740bf6/NurkagucemNairrebulaleki)欢迎小伙伴访问



在 WPF 里面挖了一个坑，在 Contains 方法里面没有加上锁，因此在 XAML 内的使用还请小心，也许会存在字典出错

```csharp
        public bool Contains(object key)
        {
        	// 这里缺少了 if (CanBeAccessedAcrossThreads) 这样的代码

            bool result = _baseDictionary.Contains(key);

            if (result)
            {
                KeyRecord keyRecord = _baseDictionary[key] as KeyRecord;
                if (keyRecord != null && _deferredLocationList.Contains(keyRecord))
                {
                    return false;
                }
            }

            //Search for the value in the Merged Dictionaries
            if (_mergedDictionaries != null)
            {
                for (int i = MergedDictionaries.Count - 1; (i > -1) && !result; i--)
                {
                    // Note that MergedDictionaries collection can also contain null values
                    ResourceDictionary mergedDictionary = MergedDictionaries[i];
                    if (mergedDictionary != null)
                    {
                        result = mergedDictionary.Contains(key);
                    }
                }
            }
            return result;
        }
```

在 XAML 里面使用 StaticResourceExtension 也就是 `{StaticResource xx}` 的方法获取的时候，将会调用到 Contains 方法。但是在使用的时候还请放心，因为理论上对字典以及 WPF 的 Hashtable 进行一个线程写一个线程读是不会有异常的。本文提供的方法只是为了

通过多线程创建资源字典的方法可以用来提升启动性能

当前整个 WPF 源代码都是开源的，请看 [https://github.com/dotnet/wpf/](https://github.com/dotnet/wpf/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
