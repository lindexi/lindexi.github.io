# WPF 已知问题 清空 CollectionView 的 SortDescriptions 可能抛出空异常

本文记录一个 WPF 的已知问题，在通过 CollectionViewSource 获取到 CollectionView 之后，如果 CollectionViewSource 对象已被 GC 回收，将可能在调用 CollectionView 的 SortDescriptions 属性进行清空或者移除项时，也就是使用 SortDescriptionCollection 类型的清空或者移除项时，在 WPF 框架里面抛出空异常

<!--more-->

<!-- 发布 -->
<!-- 博客 -->

此问题已经报告给 WPF 官方，请看 [https://github.com/dotnet/wpf/issues/7389](https://github.com/dotnet/wpf/issues/7389 )

我现在是一个成熟的开发者了，自己报告的 BUG 就要自己修。此问题已修复，请看 [https://github.com/dotnet/wpf/pull/7390](https://github.com/dotnet/wpf/pull/7390)

此问题的复现步骤如下

在一个 WPF 项目里面，构建出一个 CollectionViewSource 对象，接着只获取存放此 CollectionViewSource 对象的 View 属性，此 View 属性就是 CollectionView 类型的一个对象，将 CollectionView 存放到字段里面。等待 CollectionViewSource 被回收之后，调用 CollectionView 的 SortDescriptions 属性进行清空 SortDescriptionCollection 的内容。代码如下

```csharp
    public MainWindow()
    {
        InitializeComponent();

        var collectionViewSource = new CollectionViewSource()
        {
            Source = new List<Foo>(),
            IsLiveSortingRequested = true,
        };

        var collectionView = collectionViewSource.View;
        _collectionView = collectionView;

        collectionView.SortDescriptions.Add(new SortDescription("Name", ListSortDirection.Descending));

        Loaded += MainWindow_Loaded;
    }

    private readonly ICollectionView _collectionView;

    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        GC.Collect();
        GC.WaitForFullGCComplete();
        GC.Collect();
    }

    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        _collectionView.SortDescriptions.Clear();
    }
```

以上的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/c7556d7b92605000011425f82793f9e4063e5a00/LechelaneHenayfucee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c7556d7b92605000011425f82793f9e4063e5a00/LechelaneHenayfucee) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c7556d7b92605000011425f82793f9e4063e5a00
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c7556d7b92605000011425f82793f9e4063e5a00
```

获取代码之后，进入 LechelaneHenayfucee 文件夹

运行代码，然后点击按钮，就可以看到在 WPF 框架里面抛出空异常


异常的调用堆栈大概如下


```
>	PresentationFramework.dll!System.Windows.Data.ListCollectionView.PrepareLocalArray() 
 	PresentationFramework.dll!System.Windows.Data.ListCollectionView.RefreshOverride() 
 	PresentationFramework.dll!System.Windows.Data.CollectionView.RefreshInternal() 
 	PresentationFramework.dll!System.Windows.Data.CollectionView.RefreshOrDefer() 
 	PresentationFramework.dll!System.Windows.Data.ListCollectionView.SortDescriptionsChanged(object sender, System.Collections.Specialized.NotifyCollectionChangedEventArgs e) 
 	WindowsBase.dll!System.ComponentModel.SortDescriptionCollection.RemoveItem(int index)
 	System.Private.CoreLib.dll!System.Collections.ObjectModel.Collection<System.ComponentModel.SortDescription>.RemoveAt(int index)
 	App.dll!MyClass.Foo();
```

阅读 WPF 框架的源代码，可以了解到原因就是因为 CollectionViewSource 对象没有被引用，从而被 GC 回收。在 CollectionViewSource 回收之后，将会让其 View 属性，也就是 CollectionView 类型，被 WPF 框架触发 DetachFromSourceCollection 方法进行回收。这个 DetachFromSourceCollection 方法代码如下


```csharp
 public virtual void DetachFromSourceCollection() 
 { 
     INotifyCollectionChanged incc = _sourceCollection as INotifyCollectionChanged; 
     if (incc != null) 
     { 
         IBindingList ibl; 
         if (!(this is BindingListCollectionView) || 
             ((ibl = _sourceCollection as IBindingList) != null && !ibl.SupportsChangeNotification)) 
         { 
             incc.CollectionChanged -= new NotifyCollectionChangedEventHandler(OnCollectionChanged); 
         } 
     } 
  
     _sourceCollection = null; 
 }
```

在 DetachFromSourceCollection 方法里面，将 `_sourceCollection` 设置为空，这就导致了在清空 SortDescriptionCollection 内容的时候，尝试获取 `_sourceCollection` 的属性时，抛出空异常