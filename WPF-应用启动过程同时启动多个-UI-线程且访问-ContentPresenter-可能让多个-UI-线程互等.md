
# WPF 应用启动过程同时启动多个 UI 线程且访问 ContentPresenter 可能让多个 UI 线程互等

在应用启动过程里，除了主 UI 线程之外，如果还多启动了新的 UI 线程，且此新的 UI 线程碰到 ContentPresenter 类型，那么将可能存在让新的 UI 线程和主 UI 线程互等。这是多线程安全问题，不是很好复现，即使采用 demo 的代码，也需要几千次运行才能在某些配置比较差的机器上遇到新的 UI 线程和主 UI 线程互等，应用启动失败。本文来告诉大家复现的步骤，以及原因，和解决方法

<!--more-->



<!-- 发布 -->
<!-- 博客 -->

## 复现步骤

只需要在主 UI 线程里，加载的资源里面包含 ContentPresenter 类型的初始化。然后在主 UI 线程执行 App 时，同时启动另一个 UI 线程，让另一个 UI 线程碰到 ContentPresenter 类型。碰到 ContentPresenter 类型，让 ContentPresenter 类型的静态构造函数能被执行，代码如下

先在 App.xaml 定义资源，定义的资源刚好碰到 ContentPresenter 类型

```xml
<Application x:Class="BerehenachearbairGarciwereyer.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:BerehenachearbairGarciwereyer"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <Style x:Key="DefaultButtonStyle" TargetType="Button">
            <Setter Property="HorizontalAlignment" Value="Left" />
            <Setter Property="VerticalAlignment" Value="Center" />
            <Setter Property="Width" Value="100" />
            <Setter Property="Height" Value="100" />
            <Setter Property="Margin" Value="10,10,10,10" />
            <Setter Property="Template" >
                <Setter.Value>
                    <ControlTemplate>
                        <ContentPresenter Content="123"></ContentPresenter>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </Application.Resources>
</Application>
```

大家都知道，在 WPF 里的 XAML 将会被构建为 BAML 文件，在启动过程里面加载 BAML 将需要调用到 WPF 底层，将 BAML 展开内存。如上代码将需要创建 ContentPresenter 对象

在 App.xaml.cs 里，在 App 构造函数再启动另一个 UI 线程，在新 UI 线程里面访问 ContentPresenter 类型的 ContentProperty 属性，这是一个静态属性，在类型在程序集第一次碰到，将会调用类型的静态构造函数

```csharp
public partial class App : Application
{
    public App()
    {
        RunNewUIThread();
    }

    public static void RunNewUIThread()
    {
        Thread thread = new(Run);
        thread.SetApartmentState(ApartmentState.STA);
        thread.Start();

        void Run()
        {
            var currentDispatcher =
            System.Windows.Threading.Dispatcher.CurrentDispatcher;
            currentDispatcher.InvokeAsync(() =>
            {
                TouchContentPresenter();
            });
            System.Windows.Threading.Dispatcher.Run();
        }
    }

    [MethodImpl(MethodImplOptions.NoInlining)]
    private static void TouchContentPresenter()
    {
        // Just call the .cctor in ContentPresenter.
        var property = ContentPresenter.ContentProperty;
        CaptureObject(property);
    }

    private static void CaptureObject(object obj)
    {
        Debug.WriteLine(obj);
    }
}
```

以上的代码为了不在 Release 下被优化，于是写了 TouchContentPresenter 和 CaptureObject 两个方法。类型的静态构造函数是在类型被碰到之前，放在 TouchContentPresenter 方法里面，可以让代码在准备调用 TouchContentPresenter 方法时才尝试执行 ContentPresenter 的静态构造函数。同时加上 `MethodImplOptions.NoInlining` 让代码不会被内联

再加上 CaptureObject 方法，强行捕获参数，从而让获取属性的代码不会被优化

复现的代码放在 [https://github.com/lindexi/lindexi_gd/tree/de8bdfbf4715c7200631913cecd24749c98228a3/BerehenachearbairGarciwereyer](https://github.com/lindexi/lindexi_gd/tree/de8bdfbf4715c7200631913cecd24749c98228a3/BerehenachearbairGarciwereyer) 上，拉下来之后，构建运行，大概运行几千次，预计是可以复现

在复现时，可以看到线程 Id 为 22436 的主 UI 线程在等待 ContentPresenter 的静态构造函数完成，如下图

<!-- ![](https://user-images.githubusercontent.com/16054566/169640528-d5972c5c-ae40-4813-a484-f589edb6192b.png) -->
![](http://image.acmx.xyz/lindexi%2F20225211828538986.jpg)

这是因为在 .NET 里面，一个类型的静态构造函数，只能由一个线程执行，不会存在多线程同时执行静态构造函数。如果有某个线程在执行静态构造函数，那么其他的线程将需要等待静态构造函数执行完成才能继续碰类型。也就是相当于静态构造函数进入时加了锁，需要在执行完成之后才会释放锁，其他的线程都在等待静态构造函数的锁，也就是等待静态构造函数执行完

在线程 Id 为 16100 的新 UI 线程，执行到 ContentPresenter 的静态构造函数，然而静态构造函数在等待一个被主 UI 线程拿到的锁，静态构造函数无法执行完成

<!-- ![](https://user-images.githubusercontent.com/16054566/169640586-81efdacf-045c-4beb-a64f-d3eaec30493c.png) -->

![](http://image.acmx.xyz/lindexi%2F20225211829329892.jpg)

## 原理

核心原因是一个不良设计导致的，在 ContentPresenter 的静态构造函数里面，干的活太多了。其中就包括调用了 CreateTextBlockFactory 等方法，如下代码

```csharp
        static ContentPresenter()
        {
            DataTemplate template;
            FrameworkElementFactory text;
            Binding binding;

            // Default template for strings when hosted in ContentPresener with RecognizesAccessKey=true
            template = new DataTemplate();
            text = CreateAccessTextFactory();
            text.SetValue(AccessText.TextProperty, new TemplateBindingExtension(ContentProperty));
            template.VisualTree = text;
            template.Seal();
            s_AccessTextTemplate = template;

            // Default template for strings
            template = new DataTemplate();
            text = CreateTextBlockFactory();
            text.SetValue(TextBlock.TextProperty, new TemplateBindingExtension(ContentProperty));
            template.VisualTree = text;
            template.Seal();
            s_StringTemplate = template;

            // 忽略其他代码
        }

        internal static FrameworkElementFactory CreateAccessTextFactory()
        {
            FrameworkElementFactory text = new FrameworkElementFactory(typeof(AccessText));

            return text;
        }
```

在 CreateAccessTextFactory 创建的 FrameworkElementFactory 对象的构造函数代码如下，在构造函数将会给 `FrameworkElementFactory.Type` 属性赋值

```csharp
        public FrameworkElementFactory(Type type, string name)
        {
            Type = type;
            Name = name;
        }
```

然而 `FrameworkElementFactory.Type` 属性是比较复杂的，在赋值积分啊里面将会调用到 `XamlReader.BamlSharedSchemaContext.GetKnownXamlType` 方法
 
```csharp
        public Type Type
        {
            get { return _type; }
            set
            {
                // 忽略其他代码

                // If this is a KnownType in the BamlSchemaContext, then there is a faster way to create
                // an instance of that type than using Activator.CreateInstance.  So in that case
                // save the delegate for later creation.
                WpfKnownType knownType = null;
                if (_type != null)
                {
                    knownType = XamlReader.BamlSharedSchemaContext.GetKnownXamlType(_type) as WpfKnownType;
                }
                _knownTypeFactory = (knownType != null) ? knownType.DefaultConstructor : null;
            }
        }
```

在 `GetKnownXamlType` 里面将需要等待 `_syncObject` 对象的锁。然而 `XamlReader.BamlSharedSchemaContext` 是一个静态属性，这就意味着在使用此属性，无论是主 UI 线程还是新 UI 线程都拿到相同的 `WpfSharedBamlSchemaContext` 类型对象，也就是说调用到 `WpfSharedBamlSchemaContext` 的其他方法时，等待的是相同的一个 `_syncObject` 对象

```csharp
        internal XamlType GetKnownXamlType(Type type)
        {
            XamlType xamlType;

            lock (_syncObject)
            {
                // 忽略其他代码
               
            }
            return xamlType;
        }
```

如果是在 新 UI 线程先碰到 ContentPresenter 类型，那么 ContentPresenter 的静态构造函数将在 新 UI 线程执行。执行的静态构造函数将会等待 `WpfSharedBamlSchemaContext` 的 `_syncObject` 对象的锁。如果刚好主 UI 线程正在展开 Baml 需要使用 `Create_BamlProperty_ContentPresenter_ContentSource` 方法，那么在此方法进入时，将因为碰到了 ContentPresenter 类型，需要等待 ContentPresenter 的静态构造函数执行完成

```csharp
        [System.Runtime.CompilerServices.MethodImpl(System.Runtime.CompilerServices.MethodImplOptions.NoInlining)]
        private WpfKnownMember Create_BamlProperty_ContentPresenter_ContentSource()
        {
            Type type = typeof(System.Windows.Controls.ContentPresenter);
            DependencyProperty  dp = System.Windows.Controls.ContentPresenter.ContentSourceProperty;
            var bamlMember = new WpfKnownMember( this,  // Schema Context
                            this.GetXamlType(typeof(System.Windows.Controls.ContentPresenter)), // DeclaringType
                            "ContentSource", // Name
                             dp, // DependencyProperty
                            false, // IsReadOnly
                            false // IsAttachable
                                     );
            bamlMember.TypeConverterType = typeof(System.ComponentModel.StringConverter);
            bamlMember.Freeze();
            return bamlMember;
        }
```

在进入 `Create_BamlProperty_ContentPresenter_ContentSource` 方法之前，其实主 UI 线程已获取了 `_syncObject` 对象的锁。也就是说 ContentPresenter 的静态构造函数必须等待主 UI 线程释放锁才能完成，然而主 UI 线程必须等待 ContentPresenter 的静态构造函数执行完成才能释放锁

于是就构成了两个线程相互等待。在主 UI 线程进入 `Create_BamlProperty_ContentPresenter_ContentSource` 方法，需要等待 ContentPresenter 的静态构造函数执行完成，才能释放主 UI 线程的锁，让 ContentPresenter 的静态构造继续执行。执行在新 UI 线程的 ContentPresenter 的静态构造函数在等待主 UI 线程释放锁才能执行完成。主 UI 线程在等待新 UI 线程的静态构造函数执行完成。新 UI 线程在等待主 UI 线程等待静态构造函数执行完成之后释放的锁

两个 UI 线程进入摸鱼，应用就起不来

看到以上的原理，在实际的应用里面，想要遇到这个坑还是很难。因为 ContentPresenter 的静态构造函数只会执行一次，谁能说一定不在主 UI 线程执行？而且即使在新 UI 线程执行，那也不一定刚好在进入静态构造函数，主 UI 线程也需要用到 ContentPresenter 的相关属性。这个是需要刚好的，如果在主 UI 线程需要用到 ContentPresenter 的相关属性比较前，就在新 UI 线程进入 ContentPresenter 的静态构造函数，那将因为在新 UI 线程能等到锁而成功执行完成 ContentPresenter 的静态构造函数。如果在主 UI 线程碰到 ContentPresenter 的相关属性时，那么此时的 ContentPresenter 的静态构造函数就由主 UI 线程执行，也没有任何问题。只有在主 UI 线程拿到了锁，在准备碰到 ContentPresenter 的上一个方法时，也就是 `WpfSharedBamlSchemaContext.CreateKnownMember` 方法，此时的主 UI 线程已拿到锁，在新 UI 线程进入 ContentPresenter 的静态构造函数，如此才能让两个线程相互等待

## 解决方法

了解了原理，解决方法就十分简单了，只需要不让 ContentPresenter 的静态构造方法被新的 UI 线程调度执行即可。在新的 UI 线程执行之前，先碰一下 ContentPresenter 类型即可，例如获取此类型的某个属性之类，如以下代码

```csharp
    [MethodImpl(MethodImplOptions.NoInlining)]
    private static void TouchContentPresenter()
    {
        // Just call the .cctor in ContentPresenter.
        var property = ContentPresenter.ContentProperty;
        CaptureObject(property);
    }

    private static void CaptureObject(object obj)
    {
        Debug.WriteLine(obj);
    }
```

在开启新的 UI 线程之前，先调用一下 TouchContentPresenter 方法即可。由于碰到了类型里面的某个属性，无论是否静态，都会先调用对应的类型的静态构造函数，静态构造函数只会被调用一次，因此即可解决线程安全问题

另一个解决方法是不要尝试在应用启动的过程里面开启多个 UI 线程。在应用启动完成之后，再开启，就基本不会遇到此问题

这个问题已报告给 WPF 官方，详细请看 [Multi UI thread visit the ContentPresenter at application startup may deadlock · Issue #6609 · dotnet/wpf](https://github.com/dotnet/wpf/issues/6609)

我认为这也是一个设计缺陷，稍微熟悉 .NET 的开发者都知道，在静态构造函数里面碰锁是很危险的。因为静态构造函数的调用是不确定的，取决于第一次碰到此类型的代码进入之前。因此静态构造函数里面的碰锁的时机将是不可预期的。再加上静态构造函数只能被调用一次，这就让其他多线程碰到此类型，都需要等待静态构造函数执行完成。由于静态构造函数的调用是不可预期的，多线程里只有一个线程能进入静态构造函数，其他线程需要等待，于是此等待就相当于一个锁，如果在静态构造函数里面会碰到另一个锁，那就相当于有两个锁。有两个锁加上不可预期的调用，那这个逻辑很好构成相互等待

<!-- 

* .NET Core Version: All
* Windows version: All
* Does the bug reproduce also in WPF for .NET Framework 4.8?: Yes
 
 **Problem description:**

I start the new UI thread in the constructor in App.xaml.cs

And both my Main UI thread and the new UI thread visit the ContentPresenter properties.

The deadlock was found after running the application thousands of times.

 **Actual behavior:** 

The main UI thread which thread id is 22436 is waiting the static constructor of FrameworkElementFactory.

![](https://user-images.githubusercontent.com/16054566/169640528-d5972c5c-ae40-4813-a484-f589edb6192b.png)

The new UI thread which thread id is 16100 is waiting the lock which capture by main UI thread.

![](https://user-images.githubusercontent.com/16054566/169640586-81efdacf-045c-4beb-a64f-d3eaec30493c.png)

 **Expected behavior:**

It can work well. 

 **Minimal repro:**

Run my demo thousands of times and you can see that the application is hangs

https://github.com/lindexi/lindexi_gd/tree/de8bdfbf4715c7200631913cecd24749c98228a3/BerehenachearbairGarciwereyer



The reasons are as follows:

The ContentPresenter will create the FrameworkElementFactory in the static constructor.

```csharp
        static ContentPresenter()
        {
            DataTemplate template;
            FrameworkElementFactory text;
            Binding binding;

            // Default template for strings when hosted in ContentPresener with RecognizesAccessKey=true
            template = new DataTemplate();
            text = CreateAccessTextFactory();
            text.SetValue(AccessText.TextProperty, new TemplateBindingExtension(ContentProperty));
            template.VisualTree = text;
            template.Seal();
            s_AccessTextTemplate = template;

            // Default template for strings
            template = new DataTemplate();
            text = CreateTextBlockFactory();
            text.SetValue(TextBlock.TextProperty, new TemplateBindingExtension(ContentProperty));
            template.VisualTree = text;
            template.Seal();
            s_StringTemplate = template;

            // ...
        }

        internal static FrameworkElementFactory CreateAccessTextFactory()
        {
            FrameworkElementFactory text = new FrameworkElementFactory(typeof(AccessText));

            return text;
        }
```

And the FrameworkElementFactory will set the `FrameworkElementFactory.Type` in the constructor of FrameworkElementFactory.

```csharp
        public FrameworkElementFactory(Type type, string name)
        {
            Type = type;
            Name = name;
        }
```
 
And the `Type` property will call the `XamlReader.BamlSharedSchemaContext.GetKnownXamlType` in the `set`.

```csharp
        public Type Type
        {
            get { return _type; }
            set
            {
                // ...

                WpfKnownType knownType = null;
                if (_type != null)
                {
                    knownType = XamlReader.BamlSharedSchemaContext.GetKnownXamlType(_type) as WpfKnownType;
                }
                _knownTypeFactory = (knownType != null) ? knownType.DefaultConstructor : null;
            }
        }
```

But the `GetKnownXamlType` will wait the lock with `_syncObject` object and the `XamlReader.BamlSharedSchemaContext` is the static property which means that both the Main UI thread and the new UI thread will get the same `XamlReader.BamlSharedSchemaContext` object and wait the same `_syncObject` object.

When the new UI thread visit the ContentPresenter, the static constructor of ContentPresenter will run and wait the lock with `_syncObject` object. But the Main UI thread is creating ContentSource objects from Baml in `Create_BamlProperty_ContentPresenter_ContentSource`

```csharp
        [System.Runtime.CompilerServices.MethodImpl(System.Runtime.CompilerServices.MethodImplOptions.NoInlining)]
        private WpfKnownMember Create_BamlProperty_ContentPresenter_ContentSource()
        {
            Type type = typeof(System.Windows.Controls.ContentPresenter);
            DependencyProperty  dp = System.Windows.Controls.ContentPresenter.ContentSourceProperty;
            var bamlMember = new WpfKnownMember( this,  // Schema Context
                            this.GetXamlType(typeof(System.Windows.Controls.ContentPresenter)), // DeclaringType
                            "ContentSource", // Name
                             dp, // DependencyProperty
                            false, // IsReadOnly
                            false // IsAttachable
                                     );
            bamlMember.TypeConverterType = typeof(System.ComponentModel.StringConverter);
            bamlMember.Freeze();
            return bamlMember;
        }
```

The `Create_BamlProperty_ContentPresenter_ContentSource` much wait the static constructor of ContentPresenter in Main UI thread. And the Main UI thread get the lock with `_syncObject` object before enter the `Create_BamlProperty_ContentPresenter_ContentSource`

The new UI thread is waiting the lock with `_syncObject` object in the static constructor of ContentPresenter. And the Main UI thread get the lock with `_syncObject` object and wait the static constructor of ContentPresenter return. The new UI thread wait the Main UI thread, and the Main UI thread wait the new UI thread...

How to fix it? 

Visit ContentPresenter to enter the static constructor before both the new UI thread and the Main UI thread visit the ContentPresenter.

Please don't worry. This problem will only happen when the application starts, and multiple UI threads are required to start and access ContentPresenter at the same time. Event using the demo application, it needs thousands of starts to reproduce the problem.


 and this problem can occur only after repeated startup for many times


Even if the application is used, it needs thousands of starts to reproduce the problem
 -->





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。