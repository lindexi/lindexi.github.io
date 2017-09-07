# win10 uwp MVVM 轻量框架

如果在开发过程，遇到多个页面之间，需要传输信息，那么可能遇到设计的问题。如果因为一个页面内包含多个子页面和多个子页面之间的通信问题找不到一个好的解决方法，那么请看本文。如果因为ViewModel代码越来越多烦恼，请试试本文提供的框架。

本文介绍我做的框架，这是一个轻量的框架，可以同时使用其它的框架，用于多个页面之间，多个 ViewModel 之间的通信。

<!--more-->
<!-- csdn -->

一般的通信是一个页面内存在多个子页面，而且经常需要对多个页面进行通信，为了降低多个页面的耦合度，于是我就做了自己的框架。这个框架比较简单，很多地方都抄袭了MVVMLight，所以是他的轻量版。

暂时我把框架放在 Nuget ，因为还没写文档，所以暂时不告诉大家地址啦。

## 多页面存在的问题

这里所说的页面包括用户控件，很多情况，可以使用用户控件代替页面。在存在一个页面创建之后，就不需要替换，那么使用用户控件也可以，但是页面的等级是比用户控件更高，所以在比较大的功能，建议使用页面。另一个是存在一个功能，需要替换多个页面，这时使用用户控件就不太好，建议使用页面。接下来，我将会分析两个情况来告诉大家。

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017730155441.jpg)

第一个情况如上图的设计，左侧的选项页面就是在程序运行中不会改变的，即使改变，也只是某一些选项，所以这个左侧页面，就可以尝试使用用户控件，或者连用户控件也不要，直接写就可以了。

第二个情况就是上图功能页面，在点击不同的选项，显示不同的页面，那么这时建议使用的就是Frame和页面，因为这样比较容易导航。虽然使用用户控件同样可以做到，并且加上 DataTemplate 也可以做出来。

假如左侧的页面和功能页面是两个不同的页面，那么这两个页面如何进行通信？简单的方法是，直接使用 MianPage 传给左侧页面一个 Frame ，于是左侧页面就可以通过 Frame 进行跳转到需要显示的功能页面。看起来这是一个不错的方法，但是如果有一天，需要把功能页面修改为用户控件，那么这个动作就会很大。当然，程序员是最大的，所以不会修改，好吧，我相信了。那么如果存在另一个按钮，用于跳转到某个特定的功能页面，这时怎么办？

如果说，那就让他跳，反正我不关心，这时需要想一下，左侧页面，是否对当前显示的页面做出不同的颜色，如果功能页面修改了，如何知道？

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201773016113.jpg)

所以多页面通信之间还是有坑的，最好让他们耦合降低，这样就不会出现修改时，准备拿刀砍人。如果耦合很低，点击左侧页面，只是向一个类A发送信息，那么之后这个类A不会改变，那么左侧页面就不会进行修改，如果需要修改功能页面。如果这时有一个按钮可以控制功能页面，那么这个按钮同样对类A发送消息就可以，不需要去关心里面的逻辑，而功能页面通过监听类A的事件，可以绑定当前功能页面的对应列，所以这个设计是比刚才的方法比较好的。

接下来继续将一个多页面通信的问题。假如有一个程序，看起来和下面的图一样，有主页面，主页面有多个页面，那么这时，如何对这些页面进行通信？假如需要点击主页面的一个按钮，控制页面A中的元素，那么如何做？简单的方法是直接主页面知道页面A，直接对他元素修改。但是这个方法很容易看到，耦合很高，如果页面A修改了，那么这时就无法在不修改主页面继续运行。如果更复杂的项目，可能需要修改的地方就会很多，如果有新人不知道，没有修改所有的地方，很容易就出现软件的质量差。

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017730161636.jpg)

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

首先创建一个 WPF 或 UWP 程序，现在我还没在 Nuget 做好 UWP 的，不过可以通过下载我的源代码方式使用。

然后可以通过 nuget 下载框架。


```csharp
Install-Package lindexi.wpf.Framework  
```

或者下载源代码 [https://github.com/lindexi/UWP/tree/master/uwp/src/Framework](https://github.com/lindexi/UWP/tree/master/uwp/src/Framework)

可以直接引用代码，我比较希望引用的方法，因为做 Nuget 用起来不好。

安装完框架就是使用框架。

我将会使用一个简单的例子告诉大家如何使用。

首先是创建 ViewModel ，创建的 ViewModel 可以分为两个，如果是一个包括有其他的 ViewModel 如主页面，那么请继承 NavigateViewModel 。

然后需要在页面使用特性 ViewModel ，告诉这个页面使用哪个 viewModel


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

我把主 ViewModel 叫 ViewModel ，可以在 ViewModel 这样写


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

如果是 A 页面的 ViewModel ，这个页面没有包含其他的页面，那么可以继承 ViewModelBase ，于是简单的代码


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

需要写一个简单的基类作为 ViewModel ，这个类几乎没有内容，如果是一个几乎没有内容的，那么需要写一个接口


```csharp
/// <summary>
    /// 表示接口继承
    /// </summary>
    public interface IViewModel
    {

    } 
```

如果有类继承这个接口，就可以说这就是 ViewModel ，因为 C# 不可以多继承，所以和其他框架一起使用时，需要 ViewModel 继承其他的类，那么就可以使用继承接口。

这里 ViewModel 是可以跳转的，这是这个框架做不好的地方，让 ViewModel 需要跳转，所以就有跳转进入的和跳转出的两个函数。还需要判断当前 ViewModel 是否可用，也就是很多和页面相同。


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