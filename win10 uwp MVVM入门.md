# win10 uwp MVVM入门

MVVM是一个强大的架构，基本从WPF，wr就提倡使用MVVM。他可以将界面和后台分离，让开发人员可以不关心界面是怎样，投入到后台代码编写。

本文主要：如何在UWP使用MVVM，如何做一个自己的框架

<!--more-->

MVVM是View、model、ViewModel合起来叫MVVM。View就是界面，我们看到的，一般是Page等。

我们写界面很多用的xaml和cs合起来。他可以做出好看的效果。

ViewModel是界面的抽象，我们不知道界面有什么，但是我们提供给界面什么，这就是我们可以不管界面，抽象出来，model是不知道View。ViewModel可以简单单元测试，我们不需要打开界面。

Model是核心逻辑，有些大神说，model只定义数据结构，有些大神说model写核心逻辑，这个我不知道哪个对，我是把model写核心逻辑，如果错了，请告诉我。

如何让ViewModel抽象View，之后我们可以简单把写好的界面联系，我们是使用binding，这个是WPF强大的地方，我们的UWP也有。

我们下面说下绑定。

## 绑定

我们有多种方式绑定ViewModel，最简单的方法，是在xaml.cs写一个ViewModel，假如我们的ViewModel叫LinModel，我们可以在xaml.cs写
		
```
        public MainPage()
        {
            ViewModel = new LinModel();
            this.InitializeComponent();
            DataContext = ViewModel;
        }

        private LinModel ViewModel
        {
            set;
            get;
        }

```

我们也可以把ViewModel换成其他名字，遇到需要什么名称就使用最好的。

我们的ViewModel的new，写在构造或直接写
		
```
        private LinModel ViewModel
        {
            set;
            get;
        }=new LinModel();

```

这个方式是6之后才有的，因为我们需要的ViewModel几乎不会修改，所以我们还可如下面
		
```
        private LinModel ViewModel
        {
            get;
        }

```

因为我们不需要使用public，我们就可以这样简单写ViewModel，记得我们的ViewModel new需要在`InitializeComponent`之前，DataContent需要在`InitializeComponent`之后

他是初始化控件，如果我们的ViewModel的new写在初始化之后，那么就可能让我们控件找不到，当然DataContent在初始化之后才有，不然我们设置dc在初始化后还是会被重新刷。

DataContent可以写在xaml，很简单 我们要修改ViewModel public
		
```
DataContext="{Binding RelativeSource={RelativeSource Self},Path=ViewModel}"

```

这是一个简单的方法。

ViewModel我们可以写在xaml，xaml.cs不写代码，ViewModel需要有static的，也就是ViewModel可以实现的只有一个
		
```
        <Grid DataContext="{Binding Source={StaticResource LinModel},Path=ViewModel}">

        </Grid>

    public class LinModel
    {
        public LinModel()
        {
            ViewModel = this;
        }

        public static LinModel ViewModel
        {
            set;
            get;
        }
    }

```

注意我们不能写在Page，如果写在Page，运行`Cannot find a Resource with the Name/Key `

我们用到staticResource，我们为了可以在页面使用DataContent，我们可以把静态写在app.xaml
		
```
<Application
    x:Class="JiHuangUWP.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:JiHuangUWP"
    xmlns:view="using:JiHuangUWP.ViewModel"
    RequestedTheme="Light">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary>
                    <local:LinModel x:Key="LinModel"></local:LinModel>
                </ResourceDictionary>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>


<Page
    x:Class="JiHuangUWP.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:JiHuangUWP"
    xmlns:view="using:JiHuangUWP.ViewModel"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    DataContext="{Binding Source= {StaticResource LinModel},Path=ViewModel}"
    mc:Ignorable="d">
```

我们这个写法可以让cs不写代码，如果我们有多个相同页面，那么我们不可以使用这个办法。

我们要把static去掉
		
```
    public class LinModel
    {
        public LinModel()
        {
            ViewModel = this;
        }

        public /*static*/ LinModel ViewModel
        {
            set;
            get;
        }
    }

```

我们要做cs使用
		
```
            InitializeComponent();
            ViewModel = (LinModel) DataContext;

```

这是一个简单方法，其实有一些比较难做，我将和大家说去做一个自己的框架

<!-- 我们可以写一个类，这个类保存所有的ViewModel，我们可以通过这个方式去让我们有多个Page使用相同的ViewModel不同的实现。

		
```


```

locater是我在MVVMLight学的，大家可以使用这个方式。 -->

## 自己的框架

我们开始说如何做一个自己的框架。

在上面使用绑定的方法，我们可以看到，我们需要一个类来存放page和ViewModel，我们的ViewModel之间的通信比较难做，于是我们为了让开发简单，我们做一个简单的ViewModel，这个是核心，在程序运行就存在一个。

我们写一个类，这个类是保存ViewModel和View

这个类有`Type Page`页面，`ViewModelBase ViewModel`，如果我们有多个页使用相同ViewModel，我们需要使用key来知道

我们这个类就需要下面很少的
		
```
       public string Key
        {
            set;
            get;
        }


        public ViewModelBase ViewModel
        {
            set;
            get;
        }

        public Type Page
        {
            set;
            get;
        }

```

但是大家也看到，这个需要在使用就实现ViewModel，如果我们想要在使用ViewModel才实现，那么我们需要`Type _viewModel`，从type构造可以去看 http://lindexi.oschina.io/lindexi/post/win10-uwp-%E4%BB%8EType%E4%BD%BF%E7%94%A8%E6%9E%84%E9%80%A0/

我们在这个类写方法Navigate
		
```
        public async Task Navigate(Frame content, object paramter)
        {
            if (ViewModel == null)
            {
                ViewModel = (ViewModelBase) _viewModel.GetConstructor(Type.EmptyTypes).Invoke(null);
            }
            ViewModel.OnNavigatedTo(paramter);
#if NOGUI
            return;
#endif
            await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal,
                () =>
                {
                    content.Navigate(Page);
                });
        }

```
我们在测试是没有UI，我们就不跳转，我们可能在后台，所以需要`await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal`

我们这个类需要`ViewModelBase viewModel, Type page`输入或
		
```
        public ViewModelPage(Type viewModel, Type page)
        {
            _viewModel = viewModel;
            Page = page;
        }

```

		
```
        public ViewModelPage(Type viewModel, Type page,string key=null)
        {
            _viewModel = viewModel;
            Page = page;
        }

```

## MasterDetail

我们用我们上面写的来做一个MasterDetail，我之前做了一个简单