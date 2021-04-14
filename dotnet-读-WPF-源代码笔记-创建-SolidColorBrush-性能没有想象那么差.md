
# dotnet 读 WPF 源代码笔记 创建 SolidColorBrush 性能没有想象那么差

在 WPF 中，常用的画刷里面有纯色画刷 SolidColorBrush 类。因为画刷会对应到 DirectX 的资源，因此之前我以为纯色画刷其实会比 Color 会占用更多的资源。在 WPF 中 Color 其实是结构体，创建速度快。而 SolidColorBrush 是画刷，会对应 DirectX 资源，相对来说性能会比较差。但在通过阅读 WPF 的源代码，发现其实 SolidColorBrush 的创建的性能其实是特别好的，因此请不要担心创建了太多的纯色画刷类

<!--more-->


<!-- 发布 -->

在 WPF 中，画刷 Brush 有很多实现，本文的内容是纯色画刷的实现。在 WPF 的纯色画刷是继承 Brush 的类，这个类自己定义的只有一个字段 `_duceResource` 和 Color 一个属性，而 Color 属性是一个依赖属性。从这里可以看到 SolidColorBrush 类占用的托管内存空间其实很小

那在日常调试内存的时候，遇到的 SolidColorBrush 类占用非托管内存，这里的非托管内存是在什么时候申请的？其实非托管内存的申请是在 SolidColorBrush 被使用的时候，准确来说是被调用到 AddRefOnChannelCore 方法的时候，才会申请非托管内存。而如果只是构建出来，那么纯色画刷不会申请任何的非托管内存。也就是说此时创建纯色画刷仅仅只会用到很少量的托管内存

在 WPF 设计里面，所有继承 System.Windows.Media.Composition.DUCE.IResource 接口的类型，都可以表示这是一个 DirectX 资源类，将会在渲染过程中，申请或使用 DirectX 资源。而 DirectX 资源就是非托管资源。在 WPF 的机制，将会在 WPF 资源被使用的时候，如画刷被附加到某个元素上，在此元素渲染的时候（准确来说是之前）将会通过 IResource 接口的 AddRefOnChannel 方法让资源通过 System.Windows.Media.Composition.DUCE.Channel 申请到 DirectX 资源。以下是 IResource 接口代码

```csharp
        ///<summary>
        /// DUCE.IResource
        ///</summary>
        internal interface IResource
        {
            DUCE.ResourceHandle AddRefOnChannel(Channel channel);

            int GetChannelCount();

            DUCE.Channel GetChannel(int index);

            void ReleaseOnChannel(Channel channel);

            DUCE.ResourceHandle GetHandle(Channel channel);

            /// <summary>
            /// Only Vieport3DVisual and Visual3D implement this.
            /// Vieport3DVisual has two handles. One stored in _proxy
            /// and the other one stored in _proxy3D. This function returns
            /// the handle stored in _proxy3D.
            /// </summary>
            DUCE.ResourceHandle Get3DHandle(Channel channel);

            /// <summary>
            /// Sends a command to compositor to remove the child
            /// from its parent on the channel.
            /// </summary>
            void RemoveChildFromParent(IResource parent, DUCE.Channel channel);
        }
```

在 Brush 类将会重写 AddRefOnChannel 方法，如下面代码

```csharp
        internal abstract DUCE.ResourceHandle AddRefOnChannelCore(DUCE.Channel channel);

        /// <summary>
        /// AddRefOnChannel
        /// </summary>
        DUCE.ResourceHandle DUCE.IResource.AddRefOnChannel(DUCE.Channel channel)
        {
            // Reconsider the need for this lock when removing the MultiChannelResource.
            using (CompositionEngineLock.Acquire())
            {
                return AddRefOnChannelCore(channel);
            }
        }
```

可以看到在 Brush 类中，其实是调用了 AddRefOnChannelCore 抽象方法，在 SolidColorBrush 里面实现了 AddRefOnChannelCore 申请非托管资源

```csharp
        internal override DUCE.ResourceHandle AddRefOnChannelCore(DUCE.Channel channel)
        {
                if (_duceResource.CreateOrAddRefOnChannel(this, channel, System.Windows.Media.Composition.DUCE.ResourceType.TYPE_SOLIDCOLORBRUSH))
                {
                    Transform vTransform = Transform;
                    if (vTransform != null) ((DUCE.IResource)vTransform).AddRefOnChannel(channel);
                    Transform vRelativeTransform = RelativeTransform;
                    if (vRelativeTransform != null) ((DUCE.IResource)vRelativeTransform).AddRefOnChannel(channel);

                    AddRefOnChannelAnimations(channel);


                    UpdateResource(channel, true /* skip "on channel" check - we already know that we're on channel */ );
                }

                return _duceResource.GetHandle(channel);
} // 这个花括号在 WPF 代码里面就没对齐
```

上面代码核心就是 `_duceResource.CreateOrAddRefOnChannel` 创建 ResourceHandle 以及通过 UpdateResource 将颜色更新到 DirectX 资源

在 UpdateResource 里面，将会通过如下代码在非托管层注册纯色画刷

```csharp
        internal override void UpdateResource(DUCE.Channel channel, bool skipOnChannelCheck)
        {
                // 忽略代码

                DUCE.MILCMD_SOLIDCOLORBRUSH data;
                unsafe
                {
                    data.Type = MILCMD.MilCmdSolidColorBrush;
                    data.Handle = _duceResource.GetHandle(channel);
                    if (hOpacityAnimations.IsNull)
                    {
                        data.Opacity = Opacity;
                    }
                    data.hOpacityAnimations = hOpacityAnimations;
                    data.hTransform = hTransform;
                    data.hRelativeTransform = hRelativeTransform;
                    if (hColorAnimations.IsNull)
                    {
                    	// 将颜色给到非托管层
                        data.Color = CompositionResourceManager.ColorToMilColorF(Color);
                    }
                    // 如果有动画，那么设置动画的颜色
                    data.hColorAnimations = hColorAnimations;

                    // Send packed command structure
                    channel.SendCommand(
                        (byte*)&data,
                        sizeof(DUCE.MILCMD_SOLIDCOLORBRUSH));
                }
            }
        }
```

回到主题，在创建 SolidColorBrush 时，在 WPF 框架里面做了什么？通过上文可以看到申请非托管资源是在使用到画刷的时候，如果我创建的纯色画刷只是存放而已，而不会使用他去参加渲染，那么纯色画刷将不会占用任何非托管资源，也不需要有任何逻辑调用到非托管的 DirectX 层

在 SolidColorBrush 的构造函数里面，可以选择传入或不传入 Color 参数。如上文可以了解到在 SolidColorBrush 的颜色属性是依赖属性，假定没有传入构造参数，那么将会使用依赖属性默认值，也就是说此实例仅仅只使用到字段 `_duceResource` 的内存。从性能角度上，如果没有传入构造参数，那么如下面代码，这是一个空白的构造函数，啥都没有做

```csharp
        public SolidColorBrush()
        {
        }
```

当然了 SolidColorBrush 继承了 Brush 类，咱也需要看一下 Brush 类的构造函数的定义

```csharp
        protected Brush()
        {
        }
```

可以看到 Brush 也是空白。但 Brush 继承了 Animatable 类，咱继续看接下来的继承的类的构造

```csharp
    public abstract partial class Animatable : Freezable, IAnimatable, DUCE.IResource
    {
        protected Animatable()
        {
        }

        // 忽略代码
    }

    public abstract class Freezable : DependencyObject, ISealable
    {

      	protected Freezable()
        {
            Debug.Assert(!Freezable_Frozen
                    && !Freezable_HasMultipleInheritanceContexts
                    && !(HasHandlers || HasContextInformation),
                    "Initial state is incorrect");
        } 
    }

    public class DependencyObject : DispatcherObject
    {
        public DependencyObject()
        {
            Initialize();
        }

        private void Initialize()
        {
            CanBeInheritanceContext = true;
            CanModifyEffectiveValues = true;
        }

        internal bool CanBeInheritanceContext
        {
            [FriendAccessAllowed] // Built into Base, also used by Framework.
            get { return (_packedData & 0x00200000) != 0; }

            [FriendAccessAllowed] // Built into Base, also used by Framework.
            set
            {
                if (value)
                {
                    _packedData |= 0x00200000;
                }
                else
                {
                    _packedData &= 0xFFDFFFFF;
                }
            }
        }

        private bool CanModifyEffectiveValues
        {
            get { return (_packedData & 0x00080000) != 0; }

            set
            {
                Debug.Assert(!DO_Sealed, "A Sealed DO cannot be modified");

                if (value)
                {
                    _packedData |= 0x00080000;
                }
                else
                {
                    _packedData &= 0xFFF7FFFF;
                }
            }
        }
    }

    public abstract class DispatcherObject
    {
        protected DispatcherObject()
        {
            _dispatcher = Dispatcher.CurrentDispatcher;
        }
    }
```

可以看到性能层面上，几乎构造函数是啥都没有做。通过上面的代码也可以看到，如果一个类的继承很长，那么构造函数的调用性能，也许需要关注。在另一个仓库，也算是跨平台版本的 WPF 仓库 [https://github.com/AvaloniaUI/Avalonia](https://github.com/AvaloniaUI/Avalonia) 这里面的元素定义，元素的类型继承十分长，这是设计上的缺点

那如果在 SolidColorBrush 的构造加上参数，传入颜色，此时发生了什么？在 SolidColorBrush 的构造函数将会给依赖属性设置值，如下面代码

```csharp
        public SolidColorBrush(Color color)
        {
            Color = color;
        }

        public Color Color
        {
            get
            {
                return (Color) GetValue(ColorProperty);
            }
            set
            {
                SetValueInternal(ColorProperty, value);
            }
        }

        public static readonly DependencyProperty ColorProperty;


        static SolidColorBrush()
        {
            // We check our static default fields which are of type Freezable
            // to make sure that they are not mutable, otherwise we will throw
            // if these get touched by more than one thread in the lifetime
            // of your app. 



            // Initializations
            Type typeofThis = typeof(SolidColorBrush);
            ColorProperty =
                  RegisterProperty("Color",
                                   typeof(Color),
                                   typeofThis,
                                   Colors.Transparent,
                                   new PropertyChangedCallback(ColorPropertyChanged),
                                   null,
                                   /* isIndependentlyAnimated  = */ true,
                                   /* coerceValueCallback */ null);
        }

        private static void ColorPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            SolidColorBrush target = ((SolidColorBrush) d);


            target.PropertyChanged(ColorProperty);
        }
```

从上面代码可以看到给依赖属性设置值将会触发 ColorPropertyChanged 函数。在这个函数调用了 PropertyChanged 方法。这是定义在 Animatable 的方法，代码如下

```csharp
        internal void PropertyChanged(DependencyProperty dp)
        {
            AnimationStorage animationStorage = AnimationStorage.GetStorage(this, dp);
            IndependentAnimationStorage independentAnimationStorage = animationStorage as IndependentAnimationStorage;

            if (independentAnimationStorage != null)
            {
                independentAnimationStorage.InvalidateResource();
            }
            else
            {
                RegisterForAsyncUpdateResource();
            }
        }
```

刚创建的 SolidColorBrush 是不存在 AnimationStorage 的，因此 independentAnimationStorage 一定是空，将会调用 RegisterForAsyncUpdateResource 方法

```csharp
        internal void RegisterForAsyncUpdateResource()
        {
            DUCE.IResource resource = this as DUCE.IResource;

            if (resource != null)
            {
                if ((Dispatcher != null) && Animatable_IsResourceInvalidationNecessary)
                {
                    MediaContext mediaContext = MediaContext.From(Dispatcher);

                    //
                    // Only register for a deferred resource update if this
                    // is actually on the channel.
                    //
                    if (!resource.GetHandle(mediaContext.Channel).IsNull)
                    {
                        // Add this handler to this event means that the handler will be
                        // called on the next UIThread render for this Dispatcher.
                        mediaContext.ResourcesUpdated += new MediaContext.ResourcesUpdatedHandler(UpdateResource);
                        Animatable_IsResourceInvalidationNecessary = false;
                    }
                }
            }
        }
```

在上面代码中，因为 Animatable_IsResourceInvalidationNecessary 默认值是 falst 因此这个函数啥都没有做

可以看到无论是在 SolidColorBrush 的构造函数有没有设置参数，执行的代码逻辑都非常少，执行时间基本都可以忽略。从执行性能层面，可以认为创建 SolidColorBrush 的性能是特别好的，以上代码的执行时间预计不会比创建一个空对象慢多少。从内存层面，在 SolidColorBrush 类本身，不算继承类的情况下，只有一个字段和一个依赖属性，占用内存量不会比 Color 结构体多多少。所以可以放心创建 SolidColorBrush 对象。好吧，本文说的是创建的性能，如果要将 SolidColorBrush 用上，这就是另一个坑了，建议如果是要使用的 SolidColorBrush 对象，还是使用缓存比较好，非托管的占用还是比较多的

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。