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

然后我们需要让ViewModel继承的类
		
```
public abstract class ViewModelBase
{

}

```

我们基本的ViewModel需要在属性更改通知，我之前写了一个类 https://github.com/lindexi/UWP/blob/master/uwp/src/ViewModel/NotifyProperty.cs

我们需要继承这个

原来跳转页面的参数是写在Page的OnNavigatedTo，但我们想让ViewModel知道我们跳转，我们的ViewModel通信需要INavigable
		
```
    public interface INavigable
    {
        /// <summary>
        ///     不使用这个页面
        ///     清理页面
        /// </summary>
        /// <param name="obj"></param>
        void OnNavigatedFrom(object obj);

        /// <summary>
        ///     跳转到
        /// </summary>
        /// <param name="obj"></param>
        void OnNavigatedTo(object obj);
    }

```

所有的ViewModel继承这个，为何让ViewModel继承他，是因为我们不想每次离开、使用都new 一个，我们使用的是一个，一旦我们不使用这个页面，使用From，这样让页面清理。可以提高我们的使用，在MasterDetail，总是切换页面，可以不需要实现那么多的ViewModel。我们还可以使用他来保存我们当前的使用，我们所输入，但是一旦输入多了，这个并不是很好用，主要看你是需要什么。

如果我们的ViewModel有页面，可以跳转，我们要继承
		
```
    public interface INavigato
    {
        Frame Content
        {
            set;
            get;
        }

        void Navigateto(Type viewModel, object parameter);
    }

```
Content 就是ViewModel可以跳转页面，我们的Navigateto提供viewmodel的type或key，输入参数。这是在一个页面里可以有跳转使用，假如我们使用的页面是一个MasterDetail，我们就需要两个页面，一个是列表，一个是内容，于是我们就可以使用他来跳转。



我们在ViewModelBase把ViewModel包含的页面ViewModel数组
		
```
        public List<ViewModelPage> ViewModel
        {
            set;
            get;
        } = new List<ViewModelPage>();

```
如果我们的页面LinModel存在多个可以跳转的页面AModel、BModel，我们就把他放进base.ViewModel，需要跳转，就遍历ViewModel，拿出和输入相同type、key的ViewModel，使用他的跳转，因为我们把ViewModel和View都放一个类，我们直接使用类的跳转就好。


```
    public abstract class ViewModelBase : NotifyProperty, INavigable, INavigato
    {
        public List<ViewModelPage> ViewModel
        {
            set;
            get;
        } = new List<ViewModelPage>();

        public Frame Content
        {
            set;
            get;
        }

        public abstract void OnNavigatedFrom(object obj);
        public abstract void OnNavigatedTo(object obj);

        public async void Navigateto(Type viewModel, object paramter)
        {
            _viewModel?.OnNavigatedFrom(null);
            ViewModelPage view = ViewModel.Find(temp => temp.ViewModel.GetType() == viewModel);
            await view.Navigate(Content, paramter);
            _viewModel = view.ViewModel;
        }
        
        //当前ViewModel
        private ViewModelBase _viewModel;
    }

```

我们这样写如何绑定，我们可以通过跳转页面传入ViewModel，我们需要在ViewModelPage的Navigate，传入对应的ViewModel
		
```
            await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal,
                () =>
                {
                    content.Navigate(Page,ViewModel);
                });

```

然后在页面
		
```
        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);
            ViewModel = (LinModel) e.Parameter;
        }

```
这时，我们需要DataContent就写在ViewModel的后面

## 反射获取所有类

我们如果使用的ViewModel是Main的，我们有跳转很页面，那么我们加一个功能就需要加一个ViewModel，我们使用一个已经做好的ViewModel还需要在添加功能时修改，这样在我们添加一个新功能需要修改很多地方，我们可以使用反射，在添加新功能不需要做对已经做好的ViewModel修改太多。

我们需要一个识别类是属于我们某个ViewModel的方法，很简单，假如我们的ViewModel是LinModel，我们里面有了AModel和BModel
        
```
    public class LinModelAttribute : Attribute
    {
        
    }



    [LinModelAttribute]
    public class AModel:ViewModelBase
    {
        public override void OnNavigatedFrom(object obj)
        {
            throw new NotImplementedException();
        }

        public override void OnNavigatedTo(object obj)
        {
            throw new NotImplementedException();
        }
    }

    public class APage : Page
    {
        
    }

    public class LinModel:ViewModelBase
    {
        public LinModel()
        {
            var applacationAssembly = Application.Current.GetType().GetTypeInfo().Assembly;
            foreach (var temp in applacationAssembly.DefinedTypes
             .Where(temp => temp.CustomAttributes.Any(t => t.AttributeType == typeof(LinModelAttribute))))
            {
                var viewmodel = temp.AsType().GetConstructor(Type.EmptyTypes).Invoke(null);
                Type page=null;
                try
                {
                    page= applacationAssembly.DefinedTypes.First(t => t.Name.Replace("Page", "") == temp.Name.Replace("Model", "")).AsType();
                }
                catch 
                {
                    //InvalidOperationException
                    //提醒没有page
                    //throw new Exception("没有"+temp.Name.Replace("Model","")+"Page");
                }

                ViewModel.Add(new ViewModelPage()
                {
                    ViewModel = viewmodel as ViewModelBase,
                    Page = page
                });
            }
        }

        public override void OnNavigatedFrom(object obj)
        {
            
        }

        public override void OnNavigatedTo(object obj)
        {
        }
    }

```

我们可以使用
        
```
            var applacationAssembly = Application.Current.GetType().GetTypeInfo().Assembly;
            foreach (var temp in applacationAssembly.DefinedTypes
             .Where(temp => temp.CustomAttributes.Any(t => t.AttributeType == typeof(LinModelAttribute))))
            {
                var viewmodel = temp.AsType().GetConstructor(Type.EmptyTypes).Invoke(null);
            }

```
实现ViewModel，但是我们需要得到Page，那么简单是命名，我们把AModel和APage命名好，从temp拿名称`page= applacationAssembly.DefinedTypes.First(t => t.Name.Replace("Page", "") == temp.Name.Replace("Model", "")).AsType();`就可以得到page，我们判断InvalidOperationException，如果是这个，那么用户命名错或没有Page

如果你需要把ViewModel的命名后缀ViewModel，那么替换`temp.Name.Replace("ViewModel", "")`,如果没有这些，不需修改。这样我们添加新功能修改好少



http://lindexi.oschina.io/lindexi/post/win10-uwp-%E5%8F%8D%E5%B0%84/


## MasterDetail

我们用我们上面写的来做一个MasterDetail，我之前做了一个简单 http://lindexi.oschina.io/lindexi/post/win10-uwp-%E7%AE%80%E5%8D%95MasterDetail/

