# win10 UWP MvvmLight入门

## 安装MvvmLight

我们安装的是MvvmLightLib。

右击项目，管理Nuget，搜索MVVM

![](http://7xqpl8.com1.z0.glb.clouddn.com/bb831a25-a924-4819-aa80-5b273267c5a120161127165650.jpg)

安装第二个。他包含了运行的类库，我们将会使用里面的一些类。关于lib包含的库，参见：http://www.cnblogs.com/manupstairs/p/4890300.html

## 新建ViewModel文件夹

右击项目，新建文件夹ViewModel和View、Model三个文件夹。

在ViewModel文件夹新建类ViewModelLocator。

打开App.xaml

添加命名空间`xmlns:viewModel="using:项目.ViewModel"`

其中项目为你新建项目的名称

添加资源作为全局的ViewModel管理

```
    <Application.Resources>
        <ResourceDictionary>
            <viewModel:ViewModelLocator x:Key ="ViewModelLocator"></viewModel:ViewModelLocator>
        </ResourceDictionary>
    </Application.Resources>

```

接着假如我们有两个页面，一个是MainPage，一个是ChangeControlPage，那么我们就在ViewModel文件夹新建两个类MainModel和ChangeControlModel

他们继承ViewModelBase

接着在ViewModelLocator添加属性

```
 public ViewModel ViewModel => ServiceLocator.Current.GetInstance<ViewModel>();

```

注意，ViewModel 是多个ViewModel，也就是MainModel和ChangeControlModel，写法就是代换ViewModel为你的ViewModel

我们需要在ViewModelLocator的构造上用`SimpleIoc.Default.Register`

```
        public ViewModelLocator()
        {
            ServiceLocator.SetLocatorProvider(()=>SimpleIoc.Default);

            SimpleIoc.Default.Register<ViewModel>();
        }
```

接着打开MainPage.xaml，原来是在cs写我们的ViewModel实例，现在我们可以直接在xaml

```
    <Page.DataContext>
        <Binding Source="{StaticResource ViewModelLocator}" Path="ViewModel"></Binding>
    </Page.DataContext>
```

当然我们可以换个方式

```
    DataContext="{Binding Source={StaticResource ViewModelLocator} Path=ViewModel}"


```

##绑定





## 参考

http://www.cnblogs.com/manupstairs/
