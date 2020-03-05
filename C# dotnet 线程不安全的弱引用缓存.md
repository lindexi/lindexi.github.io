# C# dotnet 线程不安全的弱引用缓存

很多逻辑都会使用内存做缓存，这样可以提高运行效率。但是有一些逻辑很少会执行，但是如果有执行就是频繁调用。如我写了文本编辑器，在我打开文件的逻辑，将会不断调用正则判断逻辑，而平时编辑很少会调用。如果将这部分的正则逻辑缓存了，那么可以提升打开文件速度，但是在打开文件之后这部分就成为内存垃圾了。本文给大家一个弱引用缓存，也就是在频繁使用时从内存获取，在不使用时会被回收，这样可以提升性能也能减少内存使用

<!--more-->
<!-- CreateTime:2019/11/7 9:45:05 -->

<!-- csdn -->

因为作为缓存，如果需要考虑线程安全，那么这部分的逻辑就复杂了。在不考虑线程安全下，开发一个弱引用缓存还是很简单

首先是创建一个字典，这个字典包含弱引用，这样在获取之前可以先从字典获取

```csharp
        private readonly Dictionary<object, WeakReference<object>> _cacheList =
            new Dictionary<object, WeakReference<object>>();
```

在用户获取之前，需要知道，可能内存回收了。所以使用方法是获取或创建，也就是这个方法不能保证只有第一次获取时才创建，而是看内存回收

```csharp
 public T GetOrCreate<T>(object key, Func<T> createFunc)
```

如果此时可以从内存获取，那么直接返回

```csharp
            if (_cacheList.TryGetValue(key, out var weakReference))
            {
                if (weakReference.TryGetTarget(out var value))
                {
                    return (T) value;
                }
            }
```

如果不能从内存获取，就需要调用方法创建

```csharp
            var t = createFunc();
            weakReference = new WeakReference<object>(t);
            _cacheList[key] = weakReference;
            return t;
```

所以获取方法如下

```csharp
        /// <summary>
        /// 从缓存获取或在没有获取到创建
        /// </summary>
        public T GetOrCreate<T>(object key, Func<T> createFunc)
        {
            if (_cacheList.TryGetValue(key, out var weakReference))
            {
                if (weakReference.TryGetTarget(out var value))
                {
                    return (T) value;
                }
            }

            var t = createFunc();
            weakReference = new WeakReference<object>(t);
            _cacheList[key] = weakReference;
            return t;
        }
```

因为每次给一个 key 也不好用，有一些对象只需要一个类只有存在一个，可以使用类型作为 key 可以再写另一个方法

```csharp
        /// <summary>
        /// 从缓存获取或在没有获取到创建
        /// </summary>
        public T GetOrCreate<T>(Func<T> createFunc)
        {
            var type = typeof(T);
            return GetOrCreate(type, createFunc);
        }
```

这个线程不安全的弱引用缓存所有代码很少，可以直接复制在项目使用

```csharp
    /// <summary>
    /// 弱引用缓存
    /// </summary>
    public class WeakReferenceCache
    {
        /// <summary>
        /// 从缓存获取或在没有获取到创建
        /// </summary>
        public T GetOrCreate<T>(object key, Func<T> createFunc)
        {
            if (_cacheList.TryGetValue(key, out var weakReference))
            {
                if (weakReference.TryGetTarget(out var value))
                {
                    return (T) value;
                }
            }

            var t = createFunc();
            weakReference = new WeakReference<object>(t);
            _cacheList[key] = weakReference;
            return t;
        }

        /// <summary>
        /// 从缓存获取或在没有获取到创建
        /// </summary>
        public T GetOrCreate<T>(Func<T> createFunc)
        {
            var type = typeof(T);
            return GetOrCreate(type, createFunc);
        }

        private readonly Dictionary<object, WeakReference<object>> _cacheList =
            new Dictionary<object, WeakReference<object>>();
    }
```

此方法是线程不安全的，请不要在多线程下使用此方法，可以通过 [线程静态字段](https://blog.lindexi.com/post/dotnet-%E7%BA%BF%E7%A8%8B%E9%9D%99%E6%80%81%E5%AD%97%E6%AE%B5.html ) 让一个线程有一个实例

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0f639d7a6334acf3c502065182c441113f24eb05/NaycekihallembeaDiwalkailedecer) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
