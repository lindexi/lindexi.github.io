# win10 uwp 通知列表

经常看到小伙伴问，问已经绑定列表，在进行修改时，不会通知界面添加或删除。这时问题就在，一般使用的列表不会在添加时通知界面，因为他们没有通知。

本文：知道什么是通知的列表，如何去写一个通知列表。

<!--more-->
<!-- CreateTime:2018/12/25 18:12:33 -->


在 C# 很少直接使用数组，因为数组难以指定类型，需要指定类型的，一般都会使用 `List<T>` 。而 List 我就叫他列表，继承 ICollection 的类，一般就可以叫列表。

但是在使用 ListView  直接给 List 作为 Source ，那么只会在初次显示，之后无论对 List 进行什么修改，都不会导致界面的列表项修改。

当然如果列表使用的类的属性有通知，对他进行修改，还是可以看到界面修改。

如果希望对列表修改时，界面也变化，那么简单方法是使用 `ObservableCollection`。

不需要对 ObservableCollection 的属性进行通知，也就是 下面代码实际是不需要的


```csharp
        public ObservableCollection<string> ObservableCollection
        {
            set
            {
                _observableCollection = value;
                OnPropertyChanged();
            }
            get
            {
                return _observableCollection;
            }
        }

        private ObservableCollection<string> _observableCollection;
```

如果需要修改项，只需要进行和List一样的添加或移除元素就可以。

如果想问，为何使用 ObservableCollection 就可以通知界面修改了元素，而使用 List 就不可以。

其实因为 ObservableCollection 继承了 INotifyCollectionChanged ，他可以通知 ListView  说修改了项。

如果对于上面的说法觉得还是不行，那么深一点，来解释一下 ListView 是如何知道 ObservableCollection 修改了。

首先在给 Source 值的时候，会自动判断是不是有 INotifyCollectionChanged ，如果是的话，自动监听。

因为 ListView 的 Source 大概就是这样，下面代码和真的 Source 是不一样，但是大概也是这样的


```csharp
         /// <summary>
        /// 标识 <see cref="Source"/> 的依赖项属性。
        /// </summary>
        public static readonly DependencyProperty SourceProperty = DependencyProperty.Register(
            "Source", typeof(object), typeof(MainPage), new PropertyMetadata(default(object), (s, e) =>
            {
                var c = s as INotifyCollectionChanged;
                if (c != null)
                {
                    c.CollectionChanged += (sender, args) =>
                    {
                        switch (args.Action)
                        {
                            case NotifyCollectionChangedAction.Add:
                                break;
                            case NotifyCollectionChangedAction.Move:
                                break;
                            case NotifyCollectionChangedAction.Remove:
                                break;
                            case NotifyCollectionChangedAction.Replace:
                                break;
                            case NotifyCollectionChangedAction.Reset:
                                break;
                            default:
                                throw new ArgumentOutOfRangeException();
                        }
                    };
                }
            }));

        /// <summary>
        /// 获取或设置
        /// </summary>
        public object Source
        {
            get { return (object) GetValue(SourceProperty); }
            set { SetValue(SourceProperty, value); }
        }
```


判断传入的是不 INotifyCollectionChanged ，如果是的话，获取他修改时，是什么，添加或删除，修改的元素是哪个。

于是这样就可以绑定时，进行修改 列表就可以让界面得到。

那么 ObservableCollection 缺少了很多东西，如添加多个元素，需要每次都进行 遍历，是不是可以自己写一个和 ObservableCollection 差不多的？

于是我就来写一个，很简单的代码

首先需要给他一个名字，这里是随意给的。

但是命名是需要时间，经过很久，我想到诡异的名字。

需要做一个泛型，然后继承 Collection 和通知。继承 Collection 可以少写代码，因为基本的添加他做了。


```csharp
    public class AvaloniaCol<T> : Collection<T>, INotifyCollectionChanged
```


那么继承了 Collection 不可以直接写添加函数，如何做？实际他可以直接 InsertItem 就是添加会调用。


```csharp
         protected override void InsertItem(int index, T item)
        {
            base.InsertItem(index, item);
            CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(NotifyCollectionChangedAction.Add, item, index));
        }
```
需要做的就是使用 原来的方法，但是加一个通知，通知时需要告诉当前是添加和添加的元素

写了添加自然需要写删除


```csharp
         protected override void RemoveItem(int index)
        {
            var temp = this[index];
            base.RemoveItem(index);
            CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(NotifyCollectionChangedAction.Remove, temp, index));
        }
```

代码就这么简单，在界面写个按钮，用于添加或移除

可以看到界面就是进行变换，就这么简单写通知列表。

代码：http://download.csdn.net/detail/lindexi_gd/9826807

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F%25E5%2588%2597%25E8%25A1%25A8.gif)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 