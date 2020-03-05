# win10 UWP MvvmLight入门
<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>
<!-- csdn -->

## 安装MvvmLight

我们安装的是MvvmLightLib。

右击项目，管理Nuget，搜索MVVM

![](http://image.acmx.xyz/bb831a25-a924-4819-aa80-5b273267c5a120161127165650.jpg)

安装第二个。他包含了运行的类库，我们将会使用里面的一些类。关于lib包含的库，参见：http://www.cnblogs.com/manupstairs/p/4890300.html

## 新建ViewModel文件夹

右击项目，新建文件夹ViewModel和View、Model三个文件夹。

在ViewModel文件夹新建类ViewModelLocator。

打开App.xaml

添加命名空间`xmlns:viewModel="using:项目.ViewModel"`

其中项目为你新建项目的名称

添加资源作为全局的ViewModel管理

```xml
    <Application.Resources>
        <ResourceDictionary>
            <viewModel:ViewModelLocator x:Key ="ViewModelLocator"></viewModel:ViewModelLocator>
        </ResourceDictionary>
    </Application.Resources>

```

接着假如我们有两个页面，一个是MainPage，一个是ChangeControlPage，那么我们就在ViewModel文件夹新建两个类MainModel和ChangeControlModel

他们继承ViewModelBase

接着在ViewModelLocator添加属性

```csharp
 public ViewModel ViewModel => ServiceLocator.Current.GetInstance<ViewModel>();

```

注意，ViewModel 是多个ViewModel，也就是MainModel和ChangeControlModel，写法就是代换ViewModel为你的ViewModel

我们需要在ViewModelLocator的构造上用`SimpleIoc.Default.Register`

```csharp
        public ViewModelLocator()
        {
            ServiceLocator.SetLocatorProvider(()=>SimpleIoc.Default);

            SimpleIoc.Default.Register<ViewModel>();
        }
```

接着打开MainPage.xaml，原来是在cs写我们的ViewModel实例，现在我们可以直接在xaml

```xml
    <Page.DataContext>
        <Binding Source="{StaticResource ViewModelLocator}" Path="ViewModel"></Binding>
    </Page.DataContext>
```

当然我们可以换个方式

```csharp
    DataContext="{Binding Source={StaticResource ViewModelLocator}，Path=ViewModel}"


```

## 绑定

我们先在ViewModel中新建字段`_str`，我们绑定到xaml

```csharp
        public string Str
        {
            set
            {
                Set(ref _str, value);
            }
            get
            {
                return _str;
            }
        }

        private string _str;

```

```xml
        <TextBlock Text="{Binding Path=Str,Mode=OneWay}"
                   HorizontalAlignment="Center" VerticalAlignment="Center"></TextBlock>

```

我们可以把命令绑定到ViewModel，写法垃圾

```csharp
        public ViewModel(INavigationService navigationService)
        {
            _navigationService = navigationService;
            Navigate = new RelayCommand(NavigateHigPage);
        }

        public ICommand Navigate
        {
            set;
            get;
        }

        private void NavigateHigPage()
        {
            
        }

```

我们继续做从Main跳到第二页面，第二页面叫HigPage

首先打开ViewModelLocator，在构造写一个帮我们传页面信息NavigationService

```csharp
            var navigationService = new NavigationService();
            navigationService.Configure("main",typeof(MainPage));
            navigationService.Configure("hig",typeof(HightClipPage));
            SimpleIoc.Default.Register<INavigationService>(() => navigationService);


```

在需要跳转的函数

```csharp
        private void NavigateHigPage()
        {
            var navigateService = ServiceLocator.Current.GetInstance<INavigationService>();
            navigateService.NavigateTo("hig","hellow");
        }

```

我跳转到第二页面，然后给他参数hellow






## 参考

http://www.cnblogs.com/manupstairs/


