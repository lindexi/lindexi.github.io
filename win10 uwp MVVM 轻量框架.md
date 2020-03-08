# win10 uwp MVVM 轻量框架

如果在开发过程，遇到多个页面之间，需要传输信息，那么可能遇到设计的问题。如果因为一个页面内包含多个子页面和多个子页面之间的通信问题找不到一个好的解决方法，那么请看本文。如果因为ViewModel代码越来越多烦恼，请试试本文提供的框架。

本文介绍我做的框架，这是一个轻量的框架，可以同时使用其它的框架，用于多个页面之间，多个 ViewModel 之间的通信。

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->


<div id="toc"></div>

一般的通信是一个页面内存在多个子页面，而且经常需要对多个页面进行通信，为了降低多个页面的耦合度，于是我就做了自己的框架。这个框架比较简单，很多地方都抄袭了MVVMLight，所以是他的轻量版。

暂时我把框架放在 Nuget ，提供 UWP 和 WPF 的下载。

UWP：[lindexi.uwp.Framework 1.0.15512](https://www.nuget.org/packages/lindexi.uwp.Framework/1.0.15512 )

wpf: [lindexi.wpf.Framework 1.1.1155](https://www.nuget.org/packages/lindexi.wpf.Framework/ )

## 多页面存在的问题

这里所说的页面包括用户控件，很多情况，可以使用用户控件代替页面。在存在一个页面创建之后，就不需要替换，那么使用用户控件也可以，但是页面的等级是比用户控件更高，所以在比较大的功能，建议使用页面。另一个是存在一个功能，需要替换多个页面，这时使用用户控件就不太好，建议使用页面。接下来，我将会分析两个情况来告诉大家。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017730155441.jpg)

第一个情况如上图的设计，左侧的选项页面就是在程序运行中不会改变的，即使改变，也只是某一些选项，所以这个左侧页面，就可以尝试使用用户控件，或者连用户控件也不要，直接写就可以了。

第二个情况就是上图功能页面，在点击不同的选项，显示不同的页面，那么这时建议使用的就是Frame和页面，因为这样比较容易导航。虽然使用用户控件同样可以做到，并且加上 DataTemplate 也可以做出来。

假如左侧的页面和功能页面是两个不同的页面，那么这两个页面如何进行通信？简单的方法是，直接使用 MainPage 传给左侧页面一个 Frame ，于是左侧页面就可以通过 Frame 进行跳转到需要显示的功能页面。看起来这是一个不错的方法，但是如果有一天，需要把功能页面修改为用户控件，那么这个动作就会很大。当然，程序员是最大的，所以不会修改，好吧，我相信了。那么如果存在另一个按钮，用于跳转到某个特定的功能页面，这时怎么办？

如果说，那就让他跳，反正我不关心，这时需要想一下，左侧页面，是否对当前显示的页面做出不同的颜色，如果功能页面修改了，如何知道？

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201773016113.jpg)

所以多页面通信之间还是有坑的，最好让他们耦合降低，这样就不会出现修改时，准备拿刀砍人。如果耦合很低，点击左侧页面，只是向一个类A发送信息，那么之后这个类A不会改变，那么左侧页面就不会进行修改，如果需要修改功能页面。如果这时有一个按钮可以控制功能页面，那么这个按钮同样对类A发送消息就可以，不需要去关心里面的逻辑，而功能页面通过监听类A的事件，可以绑定当前功能页面的对应列，所以这个设计是比刚才的方法比较好的。

接下来继续将一个多页面通信的问题。假如有一个程序，看起来和下面的图一样，有主页面，主页面有多个页面，那么这时，如何对这些页面进行通信？假如需要点击主页面的一个按钮，控制页面A中的元素，那么如何做？简单的方法是直接主页面知道页面A，直接对他元素修改。但是这个方法很容易看到，耦合很高，如果页面A修改了，那么这时就无法在不修改主页面继续运行。如果更复杂的项目，可能需要修改的地方就会很多，如果有新人不知道，没有修改所有的地方，很容易就出现软件的质量差。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017730161636.jpg)

如果同样可以通过主页面向另一个类B发送信息，页面A监听类B，所以页面A就可以得到主页面需要删除元素的信息，由A页面来删除元素，这样可以做到耦合比较低。那么问题在很多情况都需要创建一个类来接收消息，而需要页面去获得消息，是否可以做出一个框架？我的框架就是这样想到，本来 MVVMLight 也有这样功能，看起来他的功能比较多，所以我就自己写一个，当然看完本文，相信大神们很容易就写出自己的框架。

## 通信

这个框架最好的地方在与解决多页面通信，解决方法是直接使用子页面对应的 ViewModel 向上一级页面对应的 ViewModel 发送信息，上一级的 ViewModel 接收到消息就可以判断这个消息是发送到同一级的 ViewModel 还是自己处理。

简单的方式如下所示，假如消息的类是 M ，当然，发送的消息可能很多类型，但是这里就使用 M 表示。而 AModel 就是子页面对应的 ViewModel ，BModel 就是上一级对应的 ViewModel ，那么根据上面的说法，使用 BModel 获得 AModel 的消息的代码很简单。

```
     class AModel
     {
           public Action<M> Send;
     }
     
     class BModel
     {
     
          public BModel()
          {
               AModel.Send+=Receive;
          }
          
          public AModel AModel{set;get;} = new AModel();
          
          public void Receive(M m)
          {
          
          }
     }
```

但是这个方法可以看到的是，接收到消息是在 BModel 处理，那么需要写很多代码用于判断当前是什么消息，然后还需要写很多代码进行处理，看起来就不是一个好的方法。能不能把处理给分开，答案相信你可以猜到了，如果不可以把处理分开，那么这个框架也就不好用了。

来创建一个类，这个类用于处理消息 M ，如果接收到消息 M ，就使用这个类来处理。于是处理的接收的函数就需要添加下面代码。

```
    
    public void Receive(M m)
    {
    	处理 处理 = new 处理();
    	处理.xx(m);
    }
```

但是只是输入了 m ，很多时候处理是需要修改 ViewModel ，或修改数据，如果数据有全局，那么修改比较简单，但是实际数据一般都是放在 ViewModel ，所以这时需要传入需要处理的 ViewModel 到对应的处理类。

假如有左右两个页面，左边的页面需要修改右边的页面，那么这时需要把上面的代码加上传入右边的 ViewModel ，所以处理类一般都需要传入。这样的做法可以让左边页面发送消息说，修改右边页面，而不需要让左边页面知道右边页面是什么，也不需要让他上一级页面知道右边页面是什么，上一级页面也不知道左边页面是什么，他知道的只有收到消息，找到消息发送到那个ViewModel，然后使用消息对应的处理。

可以看到一个消息可以对应多个处理，消息也对应了他需要发送到的 ViewModel ，于是上一级页面就可以根据消息找到消息需要发的 ViewModel 和找到对应处理，让他处理这个消息。这样本来 ViewModel 需要写很多代码在很多个处理类，所以需要的代码长度就没有那么长。

那么处理类如何做？我暂时就不先说了，因为从上面的描述，相信大家可以看到了框架的思想了。

使用 ViewModel 之间互发消息的方法进行联系，而实际的处理是写在其他的类，这样就可以让多个 ViewModel 之间很少耦合，而且处理不是在 ViewModel ，可以让ViewModel 的代码不太长。

## 使用

下面我来告诉大家如何使用这个框架。

首先创建一个 WPF 或 UWP 程序，可以通过 Nuget 安装或下载我的源代码方式使用。

下面告诉大家如何使用Nuget安装，如果是 wpf 的，那么请使用下面的代码


```csharp
Install-Package lindexi.wpf.Framework  
```

如果是 uwp ，那么请使用下面代码

```csharp
Install-Package lindexi.uwp.Framework -Version 1.0.15512
```

或者下载源代码 [https://github.com/lindexi/UWP/tree/master/uwp/src/Framework](https://github.com/lindexi/UWP/tree/master/uwp/src/Framework)

可以直接引用代码，我比较希望引用的方法，因为做 Nuget 用起来不好。

安装完框架就是使用框架。

我将会使用一个简单的例子告诉大家如何使用。如果之前没有读过[win10 uwp MVVM入门](http://lindexi.oschina.io/lindexi//post/win10-uwp-MVVM%E5%85%A5%E9%97%A8/ ) 那么我建议看一下

首先是创建 ViewModel ，创建的 ViewModel 可以分为两个。一个是 ViewModel 一个是 AModel。

如果是一个包括有其他的 ViewModel 如主页面，那么请继承 NavigateViewModel 。
然后需要在页面使用特性 ViewModel ，告诉这个页面使用哪个 viewModel，通过反射就可以拿到 ViewModel 对应的页面，当然这不是一定要的。


```csharp
  [ViewModel(ViewModel=typeof(ViewModel))]
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }
    } 
```

我把主 ViewModel 叫 ViewModel ，可以在 ViewModel 这样写，使用属性把他包含的 ViewModel 加起来，然后把这些放到自己的 ViewModel 列表，因为我是使用在 Page 上使用特性，所以用反射把所有页都加载。


```csharp
   public class ViewModel : NavigateViewModel
    {
        public ViewModel()
        {
        }

        public AModel AModel { set; get; }

        public LinModel LinModel { set; get; }

        public override void OnNavigatedFrom(object sender, object obj)
        {
        }

        public override void OnNavigatedTo(object sender, object obj)
        {
            FrameVisibility = Visibility.Collapsed;
            Content = (Frame) obj;
#if NOGUI
#else
            Content.Navigate(typeof(SplashPage));
#endif
            if (ViewModel == null)
            {
                ViewModel = new List<ViewModelPage>();
                //加载所有ViewModel
                var applacationAssembly = Application.Current.GetType().GetTypeInfo().Assembly;

                //CodeStorageModel = new CodeStorageModel();
                //ViewModel.Add(new ViewModelPage(CodeStorageModel, typeof(MasterDetailPage))
                //);
                foreach (
                    var temp in applacationAssembly.DefinedTypes.Where(temp => temp.IsSubclassOf(typeof(ViewModelBase)))
                )
                {
                    ViewModel.Add(new ViewModelPage(temp.AsType()));
                }

                foreach (var temp in applacationAssembly.DefinedTypes.Where(temp => temp.IsSubclassOf(typeof(Page))))
                {
                    //获取特性，特性有包含ViewModel
                    var p = temp.GetCustomAttribute<ViewModelAttribute>();

                    var viewmodel = ViewModel.FirstOrDefault(t => t.Equals(p?.ViewModel));
                    if (viewmodel != null)
                    {
                        viewmodel.Page = temp.AsType();
                    }
                }
            }

            Navigate(typeof(AModel), null);
        }

    } 
```

如果是 A 页面的 ViewModel ，这个页面没有包含其他的页面，那么可以继承 ViewModelBase ，于是简单的代码就是添加一个属性，让这个属性可以被修改。


```csharp
  public class AModel : ViewModelBase
    {
        public string Name { get; set; } = "csdn";

        public override void OnNavigatedFrom(object sender, object obj)
        {
            return;
        }

        public override void OnNavigatedTo(object sender, object obj)
        {
        }
    } 
```

如果 A 页面是需要发送消息的，那么请使用 ViewModelMessage 这样就可以使用发送。
下面就是在AModel跳转发送信息给ViewModel，于是就把自己发送到上一层。


```csharp
  public class AModel : ViewModelMessage
    {
        public string Name { get; set; } = "csdn";

        public override void OnNavigatedFrom(object sender, object obj)
        {
            return;
        }

        public override void OnNavigatedTo(object sender, object obj)
        {
            Send(new Message(this));
        }
    } 
```

如果是需要发送一个特殊的消息，那么需要继承消息



## 消息

消息需要哪些？需要知道的是，消息需要说他是发到哪里的，所以就需要一个目的，表示他需要发送到的 ViewModel 。处理表明他需要发送的 ViewModel ，他还需要说明这个消息是从哪里发送。消息还需要一个表明的标识，还有消息内容。但是消息最重要就是发送到的 ViewModel 和是哪个发送，于是创建一个接口和类。


```csharp
    public interface IMessage
    {
        ViewModelBase Source { set; get; }

        /// <summary>
        ///     判断使用哪个ViewModel，如果为空，返回上一层
        /// </summary>
        IPredicateViewModel Goal { set; get; }

        /// <summary>
        ///     判断ViewModel是否符合
        /// </summary>
        /// <param name="viewModel"></param>
        /// <returns></returns>
        bool Predicate(ViewModelPage viewModel);
    }

    public class Message : IMessage
    {
        public Message(ViewModelBase source)
        {
            Source = source;
        }

        public object Content { set; get; }

        /// <summary>
        ///     发送什么信息
        /// </summary>
        public string Key { set; get; }

        /// <summary>
        ///     发送者
        /// </summary>
        public ViewModelBase Source { set; get; }

        /// <summary>
        ///     目标
        /// </summary>
        public IPredicateViewModel Goal { set; get; }

        /// <inheritdoc />
        public bool Predicate(ViewModelPage viewModel)
        {
            if (Goal == null)
            {
                return true;
            }
            return Goal.Predicate(viewModel);
        }
    }
```

这样就可以作为消息进行发送。

## ViewModel

需要写一个简单的基类作为 ViewModel ，这个类几乎没有内容，如果是一个几乎没有内容的，那么需要写一个接口，表示这是一个 ViewModel ，这样可以用于反射获得 ViewModel 。


```csharp
/// <summary>
    /// 表示接口继承
    /// </summary>
    public interface IViewModel
    {

    } 
```

如果有类继承这个接口，就可以说这就是 ViewModel ，因为 C# 不可以多继承，所以和其他框架一起使用时，需要 ViewModel 继承其他的类，那么就可以使用继承接口。

这里 ViewModel 是可以跳转的，这是这个框架做不好的地方，让 ViewModel 需要跳转，所以就有跳转进入的和跳转出的两个函数。还需要判断当前 ViewModel 是否可用，也就是很多和页面相同，只是重新在 ViewModel 写了，于是一个可跳转的ViewModel 就需要继承 INavigable 和实现两个函数。一般使用的 ViewModel 都是可跳转的。如果使用的 ViewModel 可以继承类，那么建议继承 ViewModelBase ，请看代码。


```csharp

    /// <summary>
    /// 可跳转
    /// </summary>
    public interface INavigable: IViewModel
    {
        /// <summary>
        ///     不使用这个页面
        ///     清理页面
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="obj"></param>
        void NavigatedFrom(object sender, object obj);

        /// <summary>
        ///     跳转到
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="obj"></param>
        void NavigatedTo(object sender, object obj);
    }

    public abstract class ViewModelBase : NotifyProperty, INavigable, IViewModel
    {
        /// <summary>
        ///     表示当前ViewModel是否处于进入状态
        ///     用于命令判断是否可用
        /// </summary>
        public bool IsLoaded { get; set; }

        public bool IsEnable { get; set; }

        /// <inheritdoc />
        public virtual void NavigatedFrom(object sender, object obj)
        {
            OnNavigatedFrom(sender, obj);
            IsLoaded = false;
        }

        /// <inheritdoc />
        public virtual void NavigatedTo(object sender, object obj)
        {
            IsLoaded = true;
            OnNavigatedTo(sender, obj);
        }


        /// <summary>
        ///     从其他页面跳转出
        ///     需要释放页面
        /// </summary>
        /// <param name="source"></param>
        /// <param name="e"></param>
        public abstract void OnNavigatedFrom(object sender, object obj);

        /// <summary>
        ///     从其他页面跳转到
        ///     在这里初始化页面
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="obj"></param>
        public abstract void OnNavigatedTo(object sender, object obj);
    }
 
```

请不要告诉我上面的类继承了不需要继承的接口，原因请看微软的代码。

但是上面的 ViewModel 还没有消息的接收，下面就定义一个继承的类，这个类就可以接收消息和发送


```csharp
    /// <summary>
    ///     可以接收发送消息的页面
    /// </summary>
    public abstract class ViewModelMessage : ViewModelBase, IAdapterMessage
    {
        /// <summary>
        ///     发送消息
        /// </summary>
        EventHandler<IMessage> ISendMessage.Send { set; get; }

        /// <summary>
        ///     接收信息
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="message"></param>
        public virtual void ReceiveMessage(object sender, IMessage message)
        {
            ViewModelBase viewModel = this;
            var composite = message as ICombinationComposite;
            composite?.Run(viewModel, message);
            Composite.FirstOrDefault(temp => temp.Message == message.GetType())?.Run(viewModel, message);
        }

        /// <summary>
        ///     命令合成
        ///     全部调用发送信息的处理在<see cref="Composite" />
        /// </summary>
        protected List<Composite> Composite { set; get; } = new List<Composite>();

        /// <inheritdoc />
        public sealed override void NavigatedFrom(object sender, object obj)
        {
            base.NavigatedFrom(sender, obj);
            ((ISendMessage) this).Send = null;
        }

        /// <inheritdoc />
        public sealed override void NavigatedTo(object sender, object obj)
        {
            var viewmodel = sender as IReceiveMessage;
            if (viewmodel != null)
            {
                ((ISendMessage) this).Send += viewmodel.ReceiveMessage;
            }

            base.NavigatedTo(sender, obj);
        }

        /// <summary>
        ///     获取值
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="continueWith"></param>
        public void GetValue<T>(Action<T> continueWith)
        {
            ((ISendMessage) this).Send?.Invoke(this, new GetValueCombinationComposite<T>(this, continueWith));
        }

        /// <summary>
        /// 发送消息
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <typeparam name="U"></typeparam>
        /// <param name="run"></param>
        public virtual void SendCombinationComposite<T, U>(Action<T, U> run)
            where U : IMessage where T : IViewModel
        {
            ((ISendMessage) this).Send?.Invoke(this, new CombinationComposite<T, U>(run, this));
        }

        /// <summary>
        /// 发送消息
        /// </summary>
        public virtual void Send(IMessage message)
        {
            ((ISendMessage) this).Send?.Invoke(this, message);
        }
    }
```

那么一个负责跳转的页面，也就是有多个内部页面的窗口的 ViewModel 是建议继承 NavigateViewModel ，这样他就可以对其他 ViewModel 跳转。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017919195927.jpg)

于是主页面的 ViewModel 就可以继承 NavigateViewModel ，可以在调试模式下不传入 Frame 并且在属性生成、条件编译符添加 NOGUI 就可以进行单元测试，各种页面跳转的测试。 ViewModel 可以拿到主要方法是 Navigate ，可以让一个 ViewModel 跳转，请看代码

```csharp
            Navigate(typeof(AModel), null);

```

跳转可以加入参数，可以指定是使用哪个 Frame 跳转，如果没有指定 Frame 就会从 Content 跳转。使用跳转不需要担心异步线程，因为这里使用同步，如果异步线程使用 Navigate 也不会出现什么问题，但是我不确定在所有异步可以使用，尽量在主线程使用。

当然跳转还有支持在没有UI下的没有Frame的跳转，不过这个会在下面告诉大家，如果使用没有Frame的跳转，需要小心，因为这样就不会自动把 ViewModel 跳转出。

一般的 ViewModel 继承 ViewModelMessage 就可以使用消息，如果支持跳转的 ViewModel 继承 NavigateViewModel ，这里继承 ViewModelMessage 。如果一个继承消息的 viewModel 可以使用两个方法，这个在下面告诉大家。

## 如何发消息

如果有继承 ViewModelMessage ，发送消息很简单，只需要使用函数 Send 发送需要发送的消息就可以。

例如我有左侧列表，需要向右侧发送消息，那么简单的 ViewModel 代码就是包含两个页面，然后跳转他们

```csharp
    public class ViewModel : NavigateViewModel
    {
         public override void OnNavigatedTo(object sender, object obj)
         {
                ViewModel = new List<ViewModelPage>();
                ViewModel.Add(new ViewModelPage(typeof(AModel)));
                ViewModel.Add(new ViewModelPage(typeof(BModel))
                {
                    Key = "右侧"
                });

                Navigate(typeof(AModel), null);
                Navigate(typeof(BModel), null, new Frame());
        }
    }
```

然后AModel 向着 BModel 发送消息

```csharp
    public class AModel : ViewModelMessage
    {
        public string Name { get; set; } = "csdn";

        public override void OnNavigatedFrom(object sender, object obj)
        {
            return;
        }

        public override void OnNavigatedTo(object sender, object obj)
        {

            
        }

        public void Foo()
        {
            Send(new Message(this)
            {
                Content = Name,
                Goal = new PredicateKeyViewModel("右侧")
            });
        }
    }
```

上面的代码就可以发送消息到 BModel ，为何他知道消息是发到 BModel ，因为消息指定了发送到哪。

消息可以指定内容，指定发送到哪个 ViewModel ，如果没有指定，那就是发给他的上一个 ViewModel ，也就是跳转他的。指定可以使用 Key 指定，上面代码就是使用 Key 指定。也可以使用 PredicateInheritViewModel 指定对应的 ViewModel 需要继承什么类型，当然消息只会发送给一个 ViewModel 所以不会发送给多个，暂时框架没有做发送给多个。如果觉得当前的判断还是和需要的不同，那么可以使用 PredicateViewModel 自定义一个判断，只要符合需要，就发送消息给这个 ViewModel 。

可以看到上面的代码，对 AModel 不需要关心消息如何发送和 BModel 接到消息如何处理，于是消息处理可以在 BModel 写，但是如果消息是不同的，可能 AModel 发送一个 Name 给 右侧，也可能发送其它的消息给右侧，那么如何让右侧知道 AModel 发送的是什么？实际上可以使用指定 Key 来让 右侧知道发送的消息。

```csharp
      public void Foo()
        {
            Send(new Message(this)
            {
                Key = "name",
                Content = Name,
                Goal = new PredicateKeyViewModel("右侧")
            });
        }
```

于是 BModel 就可以通过 Key 来判断当前发送的消息是什么。

那么 BModel 收到消息是不写一个 switch 判断，如果消息少可以这样写，如果消息多，这样写就需要很多代码，框架是让 ViewModel 代码变少，如果看到这样写让 BModel 需要很多代码，那么就需要修改。

需要知道，所有的消息都是 IMessage 所以可以自己定义自己需要的消息。

## 处理

上面是如何发送消息，那么这里需要处理消息，如果消息写在 ViewModel 处理，那么 ViewModel 需要很多代码。那么如何处理代码？

实际发送的消息都不是 Message 需要创建一个消息的类，表示这是什么消息。因为使用 Message 是 Content 这没有具体类型，发送消息需要自己的类型，所以需要创建一个自己的消息。

```csharp
public class xxMessage:Message
```

这样就可以指定处理是除了哪个消息

例如有一个左侧列表，用于导航，也就是普通的菜单，那么左边列表的 ViewModel 是 NavigationPanelModel ，请看下面的图

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017730155441.jpg)

最左边的就是 NavigationPanelModel ，那么这时 NavigationPanelModel 需要进行跳转，但是 NavigationPanelModel 不包含其他 ViewModel 包含其他 ViewModel 的是主页面的。所以这时就可以使用发送消息来跳转。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20179219635.jpg)

点击 NavigationPanelModel 的选项时，可以通过发送一个消息到 ViewModel ，让 ViewModel 跳转。

于是开始定义一个消息，这个消息叫 NavigateMessage 就是导航消息，需要告诉 ViewModel 导航到哪个页面。可以看下面的代码

```csharp
    internal class NavigateMessage : Message
    {
        public NavigateMessage(ViewModelBase source) : base(source)
        {
        }

        public NavigateMessage(ViewModelBase source, string viewModel) : base(source)
        {
            ViewModel = viewModel;
        }

        /// <summary>
        ///     跳转到的页面
        /// </summary>
        public string ViewModel { get; set; }
    }
```

写好了消息，就需要在 ViewModel 写如何导航，因为消息如果没有写发送到哪，默认就是发送到上一级，所以 ViewModel 就可以收到消息。如果导航直接在 ViewModel 写，那么 ViewModel 的代码就太多，最好的方法就是我推荐的，自己写处理。定义一个类来处理。如果需要定义一个类来处理，这个类需要使用 Composite ，因为在 NavigateViewModel 会在页面跳转自动拿所有的 Composite 放在自己的处理，所以只要继承 Composite 不需要做其它的，就可以发送消息时自动处理。这个做法只有 NavigateViewModel 有，在之后我可能会把他放在一个类。

请看 NavigateViewModel 自动获得处理的代码

```csharp

         Assembly assembly = Assembly.GetCallingAssembly();
            foreach (
                var temp in
                assembly.GetTypes()
                    .Where(
                        temp =>
                            temp.IsSubclassOf(typeof(Composite)) &&
                            !typeof(ICombinationComposite).IsAssignableFrom(temp) &&
                            !temp.IsSubclassOf(typeof(CombinationComposite)) &&
                            temp != typeof(CombinationComposite)))
            {
                Composite.Add(temp.Assembly.CreateInstance(temp.FullName) as Composite);
            }

        
```

可以看到他是通过反射获得所有的处理，刚才的处理导航可以看下面的代码

```csharp
    internal class NagivateComposite : Composite
    {
        public NagivateComposite()
        {
            Message = typeof(NavigateMessage);
        }

        public override void Run(ViewModelBase source, IMessage e)
        {
            var viewModel = source as IKeyNavigato;
            var message = e as NavigateMessage;
            if (message != null)
            {
                viewModel?.Navigate(message.ViewModel, null);
            }
        }
    }
```

因为 NavigateViewModel 继承 IKeyNavigato， INavigateable ，所以就不需要判断是否 NavigateViewModel ，而 IKeyNavigato 通过 key 来找到要对应导航。另一个 INavigateable 表示支持导航，请看代码。

```csharp
  /// <summary>支持跳转</summary>
  public interface INavigateable : IViewModel
  {
    /// <summary>提供跳转的控件</summary>
    Frame Content { get; set; }

    /// <summary>正在跳转事件</summary>
    event EventHandler<ViewModelPage> Navigating;

    /// <summary>跳转完成</summary>
    event EventHandler<ViewModelPage> Navigated;

    /// <summary>跳转到页面</summary>
    /// <param name="viewModel"></param>
    /// <param name="parameter"></param>
    /// <param name="content"></param>
    void Navigate(Type viewModel, object parameter, Frame content);
  }
```

使用 INavigateable 可以指定跳转使用 Frame 所以就有一些耦合

### 合在一起的消息

可以从上面的导航看到，如果写一个消息，就可以写一个消息的处理，使用在构造写`Message`为对应处理消息类型。但是这个方法存在一个问题，如果很简单的代码，那么写两个类，看起来代码太多。

是否在发一个消息就知道他的处理，或者知道他发给什么类就做什么处理，可以。

为了减少代码，框架写了一个类 CombinationComposite 这个类可以定义消息和处理，也就是发送消息是这个类，到达指定的 ViewModel 还是使用这个类。而框架还提供泛型`CombinationComposite<T, U> : Composite, IMessage, ICombinationComposite        where U : IMessage where T : IViewModel` 泛型可以在里面不需要转换。

如果发送的消息是知道做什么，那么可以使用 CombinationComposite ，下面是发送给一个存在 Name 的 ViewModel 把他的 Name 改为 `github` 

于是定义一个存在 Name 的 ViewModel 可以定义一个接口，接口的写法一般就是继承这个接口的 ViewModel 就存在 Name 所以就可以发送给指定存在这个接口的

```csharp
    public interface INameViewModel : IViewModel
    {
        string Name { set; get; }
    }
```

然后创建 AModel 让 AModel 继承这个接口

```csharp
    public class AModel : ViewModelMessage,INameViewModel
    {
        public override void OnNavigatedFrom(object sender, object obj)
        {

        }

        public override void OnNavigatedTo(object sender, object obj)
        {

        }

        public string Name { get; set; }
    }
```

然后创建一个 ViewModel ，他包含 AModel 和 LinModel 然后LinModel就会发送消息让 AModel 修改他的名字

```csharp
           NoGui.NOGUI = true;
            ViewModel viewModel = new ViewModel();
            viewModel.NavigatedTo(null, null);
            var linModel = new LinModel();
            var amodel = new AModel();
            viewModel.ViewModel = new List<ViewModelPage>()
            {
                new ViewModelPage(linModel),
                new ViewModelPage(amodel)
            };

            viewModel.Navigate(amodel, null);
            viewModel.Navigate(linModel, null);

            linModel.Name();

```

需要定一个消息，是发送名称，请看下面是如何写

```csharp
    public class NameMessage : Message
    {
        public string Name { get; set; }

        public NameMessage(string name, ViewModelBase source) : base(source)
        {
            Name = name;
        }
    }
```

于是发送消息就只需要创建消息然后使用基类的方法

```csharp
          var name = "csdn";
            Send(new NameMessage(name, this));
```

这时在 AModel 的 ReceiveMessage 收到消息

尝试在这里处理

```csharp
    public class AModel : ViewModelMessage, INameViewModel
    {
        public override void OnNavigatedFrom(object sender, object obj)
        {

        }

        public override void OnNavigatedTo(object sender, object obj)
        {

        }

        public override void ReceiveMessage(object sender, IMessage message)
        {
            Name = ((NameMessage) message).Name;
        }

        public string Name { get; set; }
    }

```

但是如果写一个处理的类，那么在 ViewModel 添加，那么就不会在 AModel 收到消息，AModel 在没有处理才可能收到。

```csharp
    public class NameComposite : Composite
    {
        public NameComposite()
        {
            Message = typeof(NameMessage);
        }

        public override void Run(IViewModel source, IMessage message)
        {
        }
    }
```

如果写了一个除了消息的，那么在 Run 就可以收到消息，这时的 source 就是 AModel ，可以获得ViewModel 和消息就可以处理。

```csharp
    public class NameComposite : Composite
    {
        public NameComposite()
        {
            Message = typeof(NameMessage);
        }

        public override void Run(IViewModel source, IMessage message)
        {
            ((INameViewModel) source).Name = ((NameMessage) message).Name;
        }
    }
```

这样处理不需要知道 ViewModel 是什么，只需要看他是 INameViewModel 存在这个的，就进行设置，因为只有这个消息才会进来，所以可以认为就是这类型。

但是对于简单的消息，如果都这样写，看起来代码很多，但是这样的代码可以让处理不知道具体的类，消息也是不知道自己发送到哪，发送的类不需要知道具体需要做的类，把消息和处理分开虽然可以减少很多的问题。但是这样看起来代码有些多，可以使用另一个方法，如果消息和处理是在写就确定。

```csharp
    public class NameCombinationComposite : CombinationComposite
    {
        public NameCombinationComposite(ViewModelBase source, string name) : base(source)
        {
            _run = (viewModel, message) =>
            {
                ((INameViewModel) viewModel).Name = name;
            };
        }
    }
```

上面这个类就是包括发送消息和自己处理，需要在使用把消息换为这个类，所以需要做的修改很少。这个类还有一个泛型，这个可以不需要转换，现在这个组合的，需要的代码很少，于是就发送消息到具有名称的类，修改这个类的名称为输入的名。看起来的代码很简单，只需要写构造。

```csharp
        public void Name()
        {
            var name = "csdn";
            Send(new NameCombinationComposite(this, name)
            {
                Goal = new PredicateInheritViewModel(typeof(INameViewModel))
            });
        }
```

如果使用泛型的，代码看起来会比较简单

```csharp
    public class NameCombinationComposite : CombinationComposite<INameViewModel, NameCombinationComposite>
    {
        public NameCombinationComposite(ViewModelBase source, string name) : base(source)
        {
            _run = (viewModel, message) =>
            {
                viewModel.Name = name;
            };
        }
    }
```

处理还提供一个构造方法，可以使用这个构造方法在使用的地方注入如何做。请看第三个方法来修改 AModel 可以写在发出消息


```csharp
    public class NameCombinationComposite : CombinationComposite
    {
        public NameCombinationComposite(ViewModelBase source, string name) : base(source)
        {
            _run = (viewModel, message) =>
            {
                ((INameViewModel) viewModel).Name = name;
            };
        }

        public NameCombinationComposite(Action<ViewModelBase, object> run, ViewModelBase source) : base(run, source)
        {

        }
    }

          public void Name()
        {
            var name = "csdn";
            Send(new NameCombinationComposite((source, o) =>
            {
                ((INameViewModel) source).Name = name;
            }, this)
            {
                Goal = new PredicateInheritViewModel(typeof(INameViewModel))
            });
        }
```

如果不想自己定义一个处理类，那么请看下面的写法

```csharp
         public void Name()
        {
            var name = "csdn";
            Send(new CombinationComposite((source, o) =>
            {
                ((INameViewModel) source).Name = name;
            }, this)
            {
                Goal = new PredicateInheritViewModel(typeof(INameViewModel))
            });
        }
```

如果知道了使用的类的类型，那么可以使用泛型处理

```csharp
        public void Name()
        {
            var name = "csdn";
            Send(new CombinationComposite<INameViewModel, IMessage>((source, o) =>
            {
                 source.Name = name;
            }, this)
            {
                Goal = new PredicateInheritViewModel(typeof(INameViewModel))
            });
        }
```

可以看到这里不需要使用消息，那么简单的方法是换一个

```csharp
        public void Name()
        {
            var name = "csdn";
            Send(new CombinationComposite<INameViewModel>(source =>
            {
                 source.Name = name;
            }, this));
        }
```

因为默认的寻找就是找到这个指定类型的 INameViewModel 的，所以就不需要写判断，这样可以看到，从原来的很多代码，需要两个类，到只需要一个类，到在消息这里就可以写。这个过程可以让代码变少，但是这个过程让原来不需要知道的类型也在这里知道，如果需要做一个比较简单的，那么建议使用组合的，如果相对比较复杂，那么建议写两个类。

上面就已经是简单的使用了，如果需要更多的，那么请看代码。现在还有很多地方没有做好，所以欢迎你告诉我有哪些地方可以如何去写。

源代码：[https://github.com/lindexi/UWP/tree/master/uwp/src/Framework](https://github.com/lindexi/UWP/tree/master/uwp/src/Framework)

现在已经有几个项目在使用这个框架，其中有现在公司的小项目和[win10 uwp 商业游戏](http://lindexi.oschina.io/lindexi/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F/ ) [图床](ms-windows-store://pdp/?productid=9nblggh562r2) 打怪挂机 都在使用这个框架

## 感谢

[walterlv](https://walterlv.github.io/ )

[JAKE](http://niuyanjie.oschina.io/blog/ )

[张高兴 UWP IOT 大神 - 博客园](http://www.cnblogs.com/zhanggaoxing/default.html?page=1)

[杨宇杰](https://okcthouder.github.io/)

[快乐 就在你的心 的博客](https://kljzndx.github.io/kljzndx/)

