# WPF UncommonField 类型是什么

本文告诉大家一个黑科技，这个黑科技在.net 框架外无法使用，这就是 UncommonField 。下面将会告诉大家这个类有什么用。

<!--more-->
<!-- CreateTime:2018/3/8 16:25:02 -->

<!-- csdn -->

<!-- 标签：WPF，.net framework,.net源代码,源代码分析 -->

如果大家有反编译 UIElement 那么就会看到下面的代码

```csharp
internal static readonly UncommonField<EventHandlersStore> EventHandlersStoreField = new UncommonField<EventHandlersStore>();
```

那么这个`UncommonField`是什么？这个类是解决`DependencyObject `使用很多内存。使用这个类可以作为轻量的`DependencyObject `因为他使用很少的内存。

因为使用了`DependencyObject `就会创建很多默认的值，无论使用的是`TextBox.Text`的依赖属性还是`Grid.Row`附加的，都会有很多不需要使用的值。但是在框架，需要使用很少的内存，所以就使用`UncommonField`。

如果使用`UncommonField`就会去掉很多元数据、校验、通知，`UncommonField`会使用和`DependencyObject `相同的机制，让他可以存放在`DependencyObject `中和其他存放的属性一样，在没有改变值的时候会使用上一级、默认的值。所以可以减少一些内存。

因为现在很少人会写出和框架一样的那么多使用依赖属性，所以就不需要使用这个功能。

下面就是`UncommonField`代码，我添加一些注释

```csharp
     //这个类可以减少内存使用，比使用 DependencyObject 少的内存，这个类在框架使用，不在外面使用
  [FriendAccessAllowed] // Built into Base, used by Core and Framework
    internal class UncommonField<T>
    {
        /// <summary>
        ///     Create a new UncommonField.
        /// </summary>
        public UncommonField() : this(default(T))
        {
        }
 
        /// <summary>
        ///     Create a new UncommonField.
        /// </summary>
        /// <param name="defaultValue">The default value of the field.</param>
        public UncommonField(T defaultValue)
        {
            _defaultValue = defaultValue;
            _hasBeenSet = false;
 
            lock (DependencyProperty.Synchronized)
            {
            	//注册方法和依赖属性相同
                _globalIndex = DependencyProperty.GetUniqueGlobalIndex(null, null);
 
                DependencyProperty.RegisteredPropertyList.Add();
            }
        }
 
        /// <summary>
        ///     从下面代码可以看到，设置值代码和依赖属性相同
        ///     Write the given value onto a DependencyObject instance.
        /// </summary>
        /// <param name="instance">The DependencyObject on which to set the value.</param>
        /// <param name="value">The value to set.</param>
        public void SetValue(DependencyObject instance, T value)
        {
        	//如果传入的值是空，会有异常
            if (instance != null)
            {
                EntryIndex entryIndex = instance.LookupEntry(_globalIndex);
 
                //设置的值如果和默认的相同，那么就直接跳过
                // Set the value if it's not the default, otherwise remove the value.
                if (!object.ReferenceEquals(value, _defaultValue))
                {
                	//下面的代码进行设置值
                    instance.SetEffectiveValue(entryIndex, null /* dp */, _globalIndex, null /* metadata */, value, BaseValueSourceInternal.Local);
                    _hasBeenSet = true;
                }
                else
                {
                    instance.UnsetEffectiveValue(entryIndex, null /* dp */, null /* metadata */);
                }
            }
            else
            {
                throw new ArgumentNullException("instance");
            }
        }
 
        /// <summary>
        ///     如果没有设置值，就从默认获取，或者上一级，方法和依赖属性相同
        ///     Read the value of this field on a DependencyObject instance.
        /// </summary>
        /// <param name="instance">The DependencyObject from which to get the value.</param>
        /// <returns></returns>
        public T GetValue(DependencyObject instance)
        {
            if (instance != null)
            {
                if (_hasBeenSet)
                {
                    EntryIndex entryIndex = instance.LookupEntry(_globalIndex);
 
                    if (entryIndex.Found)
                    {
                        object value = instance.EffectiveValues[entryIndex.Index].LocalValue;
 
                        if (value != DependencyProperty.UnsetValue)
                        {
                            return (T)value;
                        }
                    }
                    return _defaultValue;
                }
                else
                {
                    return _defaultValue;
                }
            }
            else
            {
                throw new ArgumentNullException("instance");
            }
        }
 
 
        /// <summary>
        ///     Clear this field from the given DependencyObject instance.
        /// </summary>
        /// <param name="instance"></param>
        public void ClearValue(DependencyObject instance)
        {
            if (instance != null)
            {
                EntryIndex entryIndex = instance.LookupEntry(_globalIndex);
 
                instance.UnsetEffectiveValue(entryIndex, null /* dp */, null /* metadata */);
            }
            else
            {
                throw new ArgumentNullException("instance");
            }
        }
 
        internal int GlobalIndex
        {
            get
            {
                return _globalIndex;
            }
        }
 
        #region Private Fields
 
        private T _defaultValue;
        private int _globalIndex;
        private bool _hasBeenSet;
 
        #endregion
    }
```

从上面的代码可以自己定义一个和他一样的类，用来存放比较少的属性，但是使用不多，因为现在的软件很少需要减少那么少的内存。

参见：https://stackoverflow.com/a/18280136/6116637

https://referencesource.microsoft.com/#WindowsBase/Base/System/Windows/UncommonField.cs

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
