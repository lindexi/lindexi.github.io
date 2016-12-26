# win10 uwp MVVM入门

MVVM 是一个强大的架构，基本从 WPF ，wr就提倡使用 MVVM。他可以将界面和后台分离，让开发人员可以不关心界面是怎样，投入到后台代码编写。

本文主要：如何在 UWP 使用 MVVM，如何做一个自己的框架

<!--more-->



MVVM 是 View、Model、 ViewModel 合起来叫MVVM。 View 就是界面，我们看到的，一般是 page 等。

我们写界面很多用的 xaml 和 cs 合起来。他可以做出好看的效果。

 ViewModel 是界面的抽象，我们不知道界面有什么，但是我们提供给界面什么，这就是我们可以不管界面，抽象出来， Model 是不知道 view 。 ViewModel 可以简单单元测试，我们不需要打开界面。

 model 是核心逻辑，有些大神说， Model 只定义数据结构，有些大神说 model 写核心逻辑，这个我不知道哪个对，我是把 model 写核心逻辑，如果错了，请告诉我。

如何让 ViewModel 抽象 view ，之后我们可以简单把写好的界面联系，我们是使用 binding ，这个是 WPF 强大的地方，我们的 UWP 也有。

![](http://7xqpl8.com1.z0.glb.clouddn.com/2639f44f-463b-4fd1-b9e9-c01652649f28201612268535.jpg)

如果希望知道如何要写 MVVM ，可以去看 http://www.cnblogs.com/indream/p/3602348.html

我们下面说下绑定。

## 绑定

我们有多种方式绑定 ViewModel ，最简单的方法，是在xaml.cs 写一个 ViewModel ，假如我们的 ViewModel 叫 Linmodel ，我们可以在 xaml.cs 写类似下面的
		
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

注意我们的ViewModel new的地方

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
我们不需要去set，我们就改变一次。

因为我们不需要使用public，我们就可以这样简单写ViewModel，记得我们的ViewModel new需要在`InitializeComponent`之前，DataContent需要在`InitializeComponent`之后

他是初始化控件，如果我们的ViewModel的new写在初始化之后，那么就可能让我们控件找不到，当然DataContent在初始化之后才有，不然我们设置dc在初始化后还是会被重新刷。

DataContent可以写在xaml，很简单 我们要修改ViewModel public
		
```
DataContext="{Binding RelativeSource={RelativeSource Self},Path=ViewModel}"

```

这是一个简单的方法。

我建议大家把DataContext写在xaml，为何这样，自己试试，需要你在DataContent写在xaml才有提示补全属性，这个好功能

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

注意我们不能把DC写在Page，如果写在Page，运行`Cannot find a Resource with the Name/Key `

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
    x:Class="Framework.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Framework"
    xmlns:view="using:Framework.ViewModel"
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

我们要做cs使用ViewModel，可以简单用转换，因为我们从DataContext绑定，注意DC写的地方，千万不要在一开始写，如果发现你的DC是Null，那么你写的肯定不对
		
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

![](http://7xqpl8.com1.z0.glb.clouddn.com/6f20fca0-5961-468c-b5b4-682f3ef6f7882016122691528.jpg)

好啦，我把这个做出模板，大家可以去下载 http://download.csdn.net/detail/lindexi_gd/9716003

上面的模板适合于只有一个主界面，然后其他页面都是没有跳转。那么我们可以做一个静态的ViewModel，其他页面都直接从ViewModel中拿。

假如我们有个页面APage，AModel，那么把AModel写在ViewModel

![](http://7xqpl8.com1.z0.glb.clouddn.com/6f20fca0-5961-468c-b5b4-682f3ef6f7882016122694227.jpg)

我们可以使用在xaml DataContent绑定拿到，于是xaml.cs也简单可以拿到

        
```
<Page
    x:Class="Framework.View.APage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Framework.View"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    DataContext="{Binding Source={StaticResource ViewModel},Path=AModel}"
    mc:Ignorable="d">



        public APage()
        {
            this.InitializeComponent();
            ViewModel = (AModel) DataContext;
        }

        private AModel ViewModel
        {
            get;
        }

```
每个页面直接通信都是主页面传进来，而页面直接是没有通信，只有一个主页面，主页面可以跳转多个页面。

这是简单的汉堡。在我的应用，图床 https://www.microsoft.com/store/apps/9nblggh562r2  用到

![](http://7xqpl8.com1.z0.glb.clouddn.com/a7e7aea0-a434-41b7-82fd-a213384f4d62201612269471.jpg)

开始是进入主页面，主页面有图床、信息、设置三个页面，于是这个三个页面都在主页面，而这三个页面都没有跳转页面，所以他们可以从MainViewModel拿到自己的ViewModel。他们的通信都是跳转主页面传给他们，三个页面没有传输信息。对于设置页面，我们是放在一个存储数据类，所以我们不需要传参数，直接从存储拿。

![](http://7xqpl8.com1.z0.glb.clouddn.com/a7e7aea0-a434-41b7-82fd-a213384f4d62201612269517.jpg)


但是这个还是没解决在一个ViewModel里面，存在多个ViewModel之间的通信。

在我的私密密码本 
https://www.microsoft.com/store/apps/9nblggh5cc3g

我的创建密码页面需要和密码本联系，在创建密码创建一个密码，就把密码放到密码本

所以我们上面的不能做到，我们需要添加一些新的。我们不可以让两个页面直接联系，我们需要让一个页面和他的上层联系，让上层发给他要联系页面。

![](http://7xqpl8.com1.z0.glb.clouddn.com/a7e7aea0-a434-41b7-82fd-a213384f4d622016122695536.jpg)

关于这个是如何做，大家可以看下面的MasterDetail，这个我放在后面，后面的才是好的。

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

我们需要做的：如何让两个页面通信

![](http://7xqpl8.com1.z0.glb.clouddn.com/a7e7aea0-a434-41b7-82fd-a213384f4d6220161226101426.jpg)

我们的B页面要和A通信，我们让B发送信息到上一级页面，由上一级页面传给A。

我们需要一个信息，他是有发送者，目标、发送内容，发送了什么

        
```
    public class Message
    {
        public Message()
        {

        }
        /// <summary>
        /// 发送者
        /// </summary>
        public ViewModelBase Source
        {
            set;
            get;
        }
        /// <summary>
        /// 目标
        /// </summary>
        public string Goal
        {
            set;
            get;
        }

        public object Content
        {
            set;
            get;
        }
        /// <summary>
        /// 发送什么信息
        /// </summary>
        public string Key
        {
            set;
            get;
        }
    }

```

我们还需要ISendMessage、IReceiveMessage

到时我们的MasterModel就会有一个ISendMessage属性，我们会在DetailMasterModel中给他，当然我们总是把DetailMasterModel作为属性，所以我们可能在使用他的类给MasterModel的ISendMessage一个值，这个就是IOC。

这样做的原因，可以去看：http://blog.csdn.net/linux7985/article/details/44782623

我们来写这两个，很简单
        
```
    interface ISendMessage
    {
        void SendMessage(Message message);
    }

    interface IReceiveMessage
    {
        void ReceiveMessage(Message message);
    }

```

我们使用的发送具体的是使用Master的，所以我们写MasterSendMessage
        
```
    public class MasterSendMessage : ISendMessage
    {
        public MasterSendMessage(Action<Message> sendMessage)
        {
            _sendMessage = sendMessage;
        }

        public void SendMessage(Message message)
        {
            _sendMessage?.Invoke(message);
        }
        private Action<Message> _sendMessage;
    }

```

到时我们在DetailMaster中实现MasterSendMessage传给MasterModel

我们以我的密码本来说，我们有一个是左边是一列密码，右边点击是显示内容。

![](http://7xqpl8.com1.z0.glb.clouddn.com/e7f29c20-4d6b-4864-9af9-f58c3f045b77Framework.gif)

那么我们是使用一个ListModel和ContentModel，我们的数据是
        
```
    public class KeySecret : NotifyProperty
    {
        public KeySecret()
        {

        }

        public string Name
        {
            set
            {
                _name = value;
                OnPropertyChanged();
            }
            get
            {
                return _name;
            }
        }

        public string Key
        {
            set
            {
                _key = value;
                OnPropertyChanged();
            }
            get
            {
                return _key;
            }
        }



        private string _key;

        private string _name;
    }

```

在ListModel有一个
        
```
        public ObservableCollection<KeySecret> KeySecret
        {
            set
            {
                _keySecret = value;
                OnPropertyChanged();
            }
            get
            {
                return _keySecret;
            }
        }

        public ISendMessage SendMessage
        {
            set;
            get;
        }

```

在ContentModel有一个`public KeySecret Key`和接收。

CodeStorageModel有DetailMaster、ContentModel ListModel

其中DetailMaster控制界面，他的功能大家可以直接复制到自己的项目，不过还需要复制MasterDetailPage，复制好了，那么需要修改的是`<Frame x:Name="List" SourcePageType="local:ListPage"></Frame>` `<Frame x:Name="Content" SourcePageType="local:ContentPage"></Frame>`把一个换为自己的列表页，一个换为详情。如何使用，我会在后面说。

在CodeStorageModel跳转需要设置ListModel跳转，我们一开始就显示，于是他也要，我们需要把MasterSendMessage实现，给list，这样就是一个IOC。

        
```
            DetailMaster.Narrow();
            MasterSendMessage temp=new MasterSendMessage(ReceiveMessage);
            ListModel = new ListModel()
            {
                SendMessage = temp
            };
            ListModel.OnNavigatedTo(null);
            ContentModel = new ContentModel();

```

大神说除了foreach，不能使用temp，我这时也用了temp，是想告诉大家不要在使用。


这样我们需要在CodeStorageModel写一个接收，还记得DetailMasterModel在点击需要使用函数，我们接收有时有很多，我们需要判断他的key,如果是"点击列表"，那么我们需要布局显示。
        
```
        public void ReceiveMessage(Message message)
        {
            if (message.Key == "点击列表")
            {
                DetailMaster.MasterClick();

            }
            if (message.Goal == nameof(ContentModel))
            {
                ContentModel.ReceiveMessage(message);
            }
        }

```

ContentModel.ReceiveMessage可以把key改为点击列表
        
```
        public void ReceiveMessage(Message message)
        {
            if (message.Key == "点击列表")
            {
                Key=message.Content as KeySecret;
            }
        }

```

我们界面就不说了，直接去 https://github.com/lindexi/UWP/tree/cd1637bf31eb22a230390c205da93f840070c49d/uwp/src/Framework/Framework

我要讲下修改，我们发现我们现在写的两个页面通信在MasterDetail有用，但是要确定我们的页面，这样不好，在上面我们说可以加功能不需要去修改写好的，我们需要做的是接收信息，不使用上面的。

大家去看代码注意我是在新的master代码和现在的不同，注意链接

如何使用我的MasterDetail框架，我下面和大家说。

首先是复制

        
```
    public class DetailMasterModel : NotifyProperty
    {
        public DetailMasterModel()
        {
            SystemNavigationManager.GetForCurrentView().BackRequested += BackRequested;
            Narrow();
        }

        public int GridInt
        {
            set
            {
                _gridInt = value;
                OnPropertyChanged();
            }
            get
            {
                return _gridInt;
            }
        }

        public int ZFrame
        {
            set
            {
                _zFrame = value;
                OnPropertyChanged();
            }
            get
            {
                return _zFrame;
            }
        }

        public GridLength MasterGrid
        {
            set
            {
                _masterGrid = value;
                OnPropertyChanged();
            }
            get
            {
                return _masterGrid;
            }
        }

        public GridLength DetailGrid
        {
            set
            {
                _detailGrid = value;
                OnPropertyChanged();
            }
            get
            {
                return _detailGrid;
            }
        }

        public int ZListView
        {
            set
            {
                _zListView = value;
                OnPropertyChanged();
            }
            get
            {
                return _zListView;
            }
        }

        public bool HasFrame
        {
            set;
            get;
        }

        public Visibility Visibility
        {
            set
            {
                _visibility = value;
                OnPropertyChanged();
            }
            get
            {
                return _visibility;
            }
        }

        public void MasterClick()
        {
            HasFrame = true;
            Visibility = Visibility.Visible;
            Narrow();
        }

        public void Narrow()
        {
            if (Window.Current.Bounds.Width < 720)
            {
                MasterGrid = new GridLength(1, GridUnitType.Star);
                DetailGrid = GridLength.Auto;
                GridInt = 0;
                if (HasFrame)
                {
                    ZListView = 0;
                }
                else
                {
                    ZListView = 2;
                }
            }
            else
            {
                MasterGrid = GridLength.Auto;
                DetailGrid = new GridLength(1, GridUnitType.Star);
                GridInt = 1;
            }
        }

        private GridLength _detailGrid;

        private int _gridInt;

        private GridLength _masterGrid;

        private Visibility _visibility = Visibility.Collapsed;

        private int _zFrame;

        private int _zListView;

        private void BackRequested(object sender, BackRequestedEventArgs e)
        {
            HasFrame = false;
            Visibility = Visibility.Collapsed;
            Narrow();
        }
    }

```

然后把它放到ViewModel

        
```
        public DetailMasterModel DetailMaster
        {
            set;
            get;
        }

```
在ViewModel构造
        
```
            DetailMaster = new DetailMasterModel();
            SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility =
                AppViewBackButtonVisibility.Visible;
            DetailMaster.Narrow();


```

然后在界面

        
```
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <VisualStateManager.VisualStateGroups >
            <VisualStateGroup CurrentStateChanged="{x:Bind View.DetailMaster.Narrow}">
                <VisualState>
                    <VisualState.StateTriggers>
                        <AdaptiveTrigger MinWindowWidth="720"/>
                    </VisualState.StateTriggers>
                    <VisualState.Setters >
                        <!--<Setter Target="Img.Visibility" Value="Collapsed"></Setter>-->
                    </VisualState.Setters>
                </VisualState>
                <VisualState>
                    <VisualState.StateTriggers>
                        <AdaptiveTrigger MinWindowHeight="200">

                        </AdaptiveTrigger>

                    </VisualState.StateTriggers>
                    <VisualState.Setters >

                    </VisualState.Setters>
                </VisualState>
            </VisualStateGroup>
        </VisualStateManager.VisualStateGroups>

        <Grid>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="{x:Bind View.DetailMaster.MasterGrid,Mode=OneWay}"></ColumnDefinition>
                <ColumnDefinition Width="{x:Bind View.DetailMaster.DetailGrid,Mode=OneWay}"></ColumnDefinition>
            </Grid.ColumnDefinitions>

            <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"
                  Canvas.ZIndex="{x:Bind View.DetailMaster.ZListView,Mode=OneWay}">
                <!--<Grid Background="Black"></Grid>-->
                <TextBlock Text="List" HorizontalAlignment="Center"></TextBlock>
                <Frame x:Name="List" SourcePageType="local:ListPage"></Frame>
            </Grid>
            <Grid Grid.Column="{x:Bind View.DetailMaster.GridInt,Mode=OneWay}" Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"
                  Canvas.ZIndex="{x:Bind View.DetailMaster.ZFrame}">
                <Image Source="ms-appx:///Assets/Strawberry_Adult_content_easyicon.net.png"></Image>
                <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}" Visibility="{x:Bind View.DetailMaster.Visibility,Mode=OneWay}">
                    <TextBlock Text="content" HorizontalAlignment="Center"></TextBlock>
                    <!--<Grid Background="#FF565500"></Grid>-->
                    <Frame x:Name="Content" SourcePageType="local:ContentPage"></Frame>
                </Grid>
            </Grid>
        </Grid>
    </Grid>

```

注意把`<Frame x:Name="List" SourcePageType="local:ListPage"></Frame>`换为列表页面，和`<Frame x:Name="Content" SourcePageType="local:ContentPage"></Frame>`换为内容，` <Image Source="ms-appx:///Assets/Strawberry_Adult_content_easyicon.net.png"></Image>`换为自己的图片

需要在xaml.cs写ViewModel为view，如果不是，那么自己换名。

页面的联系使用`ISendMessage`，和接收，他向MasterDetailViewModel发信息，让ContentModel接收。

我们需要和上面写的一样，传入MasterSendMessage给他，让他可以发送信息。

然后判断发送信息，发给内容，具体可以去看代码，如果有不懂请发邮件或在评论，这很简单


我们写CodeStorageAttribute，这个是我们一个页面，他包含的ViewModel。

        
```
    [AttributeUsage(AttributeTargets.Class, Inherited = false, AllowMultiple = true)]
    sealed class CodeStorageAttribute : Attribute
    {
       
    }

```

我们在ListModel和ContentModel写CodeStorageAttribute

        
然后我们可以在CodeStorageModel 
        
```
           var applacationAssembly = Application.Current.GetType().GetTypeInfo().Assembly;
            foreach (var temp in applacationAssembly.DefinedTypes
                .Where(temp => temp.CustomAttributes.Any(t => t.AttributeType == typeof(CodeStorageAttribute))))
            {
                var viewmodel = temp.AsType().GetConstructor(Type.EmptyTypes).Invoke(null);
                Type page = null;
                try
                {
                    page =
                        applacationAssembly.DefinedTypes.First(
                            t => t.Name.Replace("Page", "") == temp.Name.Replace("Model", "")).AsType();
                }
                catch
                {
                    //InvalidOperationException
                    //提醒没有page
                    //throw new Exception("没有"+temp.Name.Replace("Model","")+"Page");
                }

                ViewModel.Add(new ViewModelPage( viewmodel as ViewModelBase,page));
            }

```

我修改ISendMessage
        
```
    public interface ISendMessage
    {
        EventHandler<Message> SendMessageHandler
        {
            set;
            get;
        }
    }

```

判断我们的ViewModel是不是ISendMessage，页面是先上一级发送，所以我们把SendMessageHandler添加
        
```
            foreach (var temp in ViewModel.Where(temp => temp.ViewModel is ISendMessage))
            {
                ((ISendMessage)temp.ViewModel).SendMessageHandler += (s, e) =>
               {
                   ReceiveMessage(e);
               };
            }

```

我们删除`public ContentModel ContentModel` `public ListModel ListModel`在ListPage和Content，我们直接使用索引

在CodeStorageModel
        
```
        public ViewModelBase this[string str]
        {
            get
            {
                foreach (var temp in ViewModel)
                {
                    if (temp.Key == str)
                    {
                        return temp.ViewModel;
                    }
                }
                return null;
            }
        }

```

修改ListPage dateContent

```
DataContext="{Binding Source={StaticResource ViewModel},Path=CodeStorageModel[ListModel]}"

```

ContentPage 的dateContent
        
```
    DataContext="{Binding Source={StaticResource ViewModel},Path=CodeStorageModel[ContentModel]}"


```

在CodeStorageModel OnNavigatedTo
        
```
        public override void OnNavigatedTo(object obj)
        {
            DetailMaster.Narrow();
            foreach (var temp in ViewModel)
            {
                temp.ViewModel.OnNavigatedTo(null);
            }
        }

```

这样我们就不需要去写一个ListModel在我们写CodeStorageModel，我们也不知道哪个页面会发送，不知哪个页面接收，我们直接在接收看信息发送的哪个，找出，使用他的接收
        
```
        public void ReceiveMessage(Message message)
        {
            if (message.Key == "点击列表")
            {
                DetailMaster.MasterClick();
            }
            foreach (var temp in ViewModel)
            {
                if (temp.Key == message.Goal)
                {
                    var receive = temp.ViewModel as IReceiveMessage;
                    receive?.ReceiveMessage(message);
                }
            }
        }

```

我们可以做的页面的联系，我们不知道我们有哪些页面，如果看到我写错请评论

全部源代码

https://github.com/lindexi/UWP/tree/master/uwp/src/Framework/Framework

参考：
http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html

