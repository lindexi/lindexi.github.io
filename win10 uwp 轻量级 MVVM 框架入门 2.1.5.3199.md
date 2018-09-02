# win10 uwp 轻量级 MVVM 框架入门 2.1.5.3199

一个好的框架是不需要写教程大家看到就会用，但是本金鱼没有那么好的技术，所以需要写很长的博客告诉大家如何使用我的框架。

<!--more-->
<div id="toc"></div>
<!-- csdn -->
<!-- 标签：win10,uwp,mvvm -->

在本文开始之前，希望大家是有 UWP 基础而且熟悉 C#，因为本金鱼有很多认为是大家都知道的就没有在博客说。

## 安装

首先需要从 Nuget 安装两个库

 - lindexi.uwp.Framework

 - lindexi.MVVM.Framework

第一个库是使用 UWP 的封装，因为我还有 WPF 的封装，实际上在使用，用 WPF 或 UWP 是差不多的。只要存在 UWP 和 WPF 不相同的库，我就把这写封装在不同的库。

使用 WPF 项目只需要安装 lindexi.wpf.Framework 这个库。因为 Nuget 可以找到依赖库，所以只需要安装 lindexi.wpf.Framework 就会自动安装 lindexi.MVVM.Framework 。如果现在使用的是 Xarmain ，那么安装 lindexi.MVVM.Framework 就可以，这个库使用 dotnet framework 4.5 和 dotnet standard 2.0 ，所以在很多项目都可以使用。

## 项目要求

安装这个库的要求是 UWP 的最低版本是 16299 ，因为在 16299 才支持 dotnet standard 2.0，在之前的版本是不支持。

如果使用的是 WPF 项目，要求项目最低版本是 dotnet framework 4.5 

## 主界面

这个框架是适合有一个主界面和多个子页面的程序，而且适合多个子页面之间有通信，包括子页面让另一个页面跳转等的框架。

先创建一个 ViewModel 类，表示这是主界面。

```csharp
    public class ViewModel : NavigateViewModel
```

然后在 MainPage 添加 ViewModel ，因为需要做导航，所以需要在前台添加 Frame 用来做导航。

```csharp
    <Grid>
        <Frame x:Name="Frame" />
    </Grid>
```

```csharp
        public ViewModel.ViewModel ViewModel { get; set; } = new ViewModel.ViewModel();
```

不需要我说，大家也知道代码放在哪

![](http://image.acmx.xyz/lindexi%2F20186121956549284.jpg)

很多程序在启动的是否都需要读取配置，这时就需要先显示一个初始页面，在这个页面显示的过程，加载很多数据

在 Main 构造函数使用 LoadAsync 方法，这个方法先跳转到 SplashPage 然后再调用 ViewModel 的读取数据

```csharp
        public MainPage()
        {
            this.InitializeComponent();
            LoadAsync();
        }
```

在 LoadAsync 方法使用 ViewModel.Read 读取数据，而且组合对应的启动的页面和 ViewModel 。这里还写有两个页面 MeetokaCutusaiPage 和 WastounowMearhallworcelPage ，使用下面代码组合

```csharp
        private async void LoadAsync()
        {
            Frame.Navigate(typeof(SplashPage));
            await ViewModel.Read();

            var viewModelPage = new List<ViewModelPage>()
            {
                new ViewModelPage(new NavigatableViewModel<WastounowMearhallworcelModel>(),new NavigatablePage(typeof(WastounowMearhallworcelPage))),
                new ViewModelPage(new NavigatableViewModel<MeetokaCutusaiModel>(),
                    new NavigatablePage(typeof(MeetokaCutusaiPage)))
            };
            ViewModel.ViewModelPage = viewModelPage;
            ViewModel.NavigatedTo(this, (NavigateFrame)Frame);
        }
```

这里创建 ViewModelPage 需要 INavigatableViewModel 和 INavigatablePage 的原因是，我这个还有在 WPF 使用，大家都知道 WPF 的 Frame 跳转和 UWP 的相同，所以需要传入不同的类

这里大家还看到我使用了`(NavigateFrame)Frame`，因为在 ViewModel.NavigatedTo 使用的是 INavigateFrame 来作为跳转，如果传入其他的参数就需要自己写的 ViewModel 进行处理。

这个方式是代码进行组合多个页面和 ViewModel ，如果页面和 ViewModel 比较少，使用这个方法还是可以。如果页面比较多，那么就建议使用反射或其他方法组合，不要自己写。

## 读取文件

在软件启动的过程，需要先使用 ViewModel 读取配置信息，读取到的配置信息放在 ViewModel 的属性，在页面跳转，ViewModel 可以把信息传给跳转的 ViewModel 这样就可以让被跳转的 ViewModel 知道信息。

最上面的页面的 ViewModel 是不做功能的，就做跳转，实际上他的跳转逻辑也不需要写，因为底层已经做了跳转的逻辑。

为了模拟读取数据，使用 Task.Delay 假装是在读取数据

```csharp
         /// <summary>
        /// 读取数据
        /// </summary>
        /// <returns></returns>
        public async Task Read()
        {
            await Task.Delay(1000);
        }
``` 

## 页面传参

如果没有使用框架，那么在开发的时候有一个问题，ViewModel 是写在 页面进行创建还是从外面创建然后传进来。

如果写在页面有一个问题是如何把其他页面跳转的信息发送到 ViewModel ，这个框架使用的是在外面创建。

在 UWP 的页面参数是在 OnNavigatedTo 函数拿到。

在刚才的ViewModel 就在读取完信息，就把页面跳转到 WastounowMearhallworcelModel ，这是一个随意的名字

```csharp
        public override void OnNavigatedTo(object sender, object obj)
        {
            base.OnNavigatedTo(sender, obj);
           
            Navigate(typeof(WastounowMearhallworcelModel));
        }
```

在 WastounowMearhallworcelModel 对应的页面使用 OnNavigatedTo 就可以拿到这个 ViewModel ，需要强转，我之前想使用泛型的方法让页面指定 ViewModel ，但是存在一个文件是 xaml 对泛型支持不好，所以不在 UWP 使用这个方法

```csharp
        /// <inheritdoc />
        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            ViewModel = (WastounowMearhallworcelModel) e.Parameter;
            DataContext = e.Parameter;
            base.OnNavigatedTo(e);
        }

        public WastounowMearhallworcelModel ViewModel { get; set; }
```

在页面使用泛型请看 [win10 uwp 如何让 Page 继承泛型类](https://lindexi.gitee.io/post/win10-uwp-%E5%A6%82%E4%BD%95%E8%AE%A9-Page-%E7%BB%A7%E6%89%BF%E6%B3%9B%E5%9E%8B%E7%B1%BB.html )

现在就可以使用 WastounowMearhallworcelModel ，在 WastounowMearhallworcelModel 里面也可以获得主界面传过来的参数

```csharp
        /// <inheritdoc />
        public override void OnNavigatedTo(object sender, object obj)
        {
            
        }
```

这里的 obj 就是页面导航传过来的参数，也就是原来的 Page 传过来的就是 ViewModel ，在 ViewModel 跳转的就是另一个 ViewModel 传过来的参数。

这样可以去掉 Page 进行调试和测试，因为这时的 ViewModel 完全使用 ViewModel 就可以做到。

但是对于一些交互细节要求比较高的地方，那么就不建议使用 MVVM 来做，如手势移动这些交互。

在进入一个 ViewModel 的时候，需要让他发送消息给其他的 ViewModel ，在继承 ViewModelMessage 就可以使用 Send 函数，发送的消息可以是消息也可以是告诉指定ViewModel如何处理。

发送的消息先会发送到这个 ViewModel 的上一级，如果这个消息指定的 ViewModel 不是上一级的 ViewModel 就会在上一级寻找同级的 ViewModel 。如果找到 消息指定的 ViewModel 再寻找消息对应的处理，把消息交给处理。如果发送的消息是自带处理，就调用消息本身的处理。

所以通过这个方式就可以让 ViewModel 发送消息到另一个 ViewModel ，下面的代码就是 WastounowMearhallworcelModel 发送消息，让主页面跳转到 MeetokaCutusaiModel 的页面

```csharp
        public void NavigatedMeetokaCutusaiModel()
        {
            Send(new NavigateMessage(this,typeof(MeetokaCutusaiModel).Name));
        }
```

这样做的设计是解耦，在 WastounowMearhallworcelModel 是完全不知道跳转的逻辑，他只需要知道发送这个消息，就会切换页面。而且页面被切换到 MeetokaCutusaiModel ，轻量框架是可以用来减少 ViewModel 的相关。

这个框架的设计参考了 MVVMCross 和 MVVMLight 只是减少了里面部分功能

参见：

[win10 uwp MVVM入门](https://lindexi.gitee.io/post/win10-uwp-MVVM%E5%85%A5%E9%97%A8.html )

[win10 uwp MVVM 轻量框架](https://lindexi.gitee.io/post/win10-uwp-MVVM-%E8%BD%BB%E9%87%8F%E6%A1%86%E6%9E%B6.html )

[win10 uwp MVVM 语义耦合](https://lindexi.gitee.io/post/win10-uwp-MVVM-%E8%AF%AD%E4%B9%89%E8%80%A6%E5%90%88.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
