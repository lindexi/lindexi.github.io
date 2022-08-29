
# 读 MAUI 源代码 理解可绑定对象和可绑定属性的存储机制

和 UWP 与 WPF 不同的是在 MAUI 里面，使用可绑定对象 BindableObject 替换了依赖对象的概念，我阅读了 MAUI 的源代码发现其实只是命名变更了，里面的机制和设计思想都是差不多的。在 MAUI 里面提供 BindableObject 用来支持可绑定属性机制和附加属性机制，本文将告诉大家在 MAUI 里面是如何在可绑定对象里面提供可绑定属性和附加属性的存储的机制

<!--more-->


<!-- CreateTime:2022/8/24 19:38:35 -->

<!-- 发布 -->
<!-- 博客 -->

在 WPF 里面，依赖属性的提出的一部分原因是为了省内存。在 MAUI 里面，我猜测省内存是可绑定对象提出的一个原因。由于一个界面控件，例如按钮等，有着非常庞大数量的属性，假设每个控件里面的所有属性都是需要独立的对象不能共用，那么在复杂界面上，将会因为大量的控件的大量属性占用大量的内存。可绑定对象里面可以实现在属性没有被赋值时，将可以使用默认值，而对于大部分控件来说，很多不常用的属性都是使用默认值即可。可绑定对象需要解决的是让可绑定属性可以代替普通的 CLR 属性，对可绑定属性进行赋值时，可以值和可绑定对象关联，从而可以读取出来。既然名字叫可绑定对象，那自然也要实现绑定的支持，绑定的支持的核心就是通知，需要支持在属性值变更的时候进行通知。接下来将通过阅读源代码了解在 MAUI 里是如何实现

打开 MAUI 的 BindableObject 的源代码，可以看到在 BindableObject 里有 `_properties` 字段，定义如下

```csharp
    public abstract class BindableObject : INotifyPropertyChanged, IDynamicResourceHandler
    {
        readonly Dictionary<BindableProperty, BindablePropertyContext> _properties = new Dictionary<BindableProperty, BindablePropertyContext>(4);
    }
```

没错，这就是在 MAUI 里面的可绑定对象的存储核心实现。在 MAUI 的可绑定对象里面通过 `_properties` 字典存放可绑定属性的值内容，字典的 Key 是 BindableProperty 可绑定属性，字典的 Value 是 BindablePropertyContext 可绑定属性上下文，初始化字典默认占用 4 个空间，默认初始化空间是为了优化而已，没有什么特别用途。通过此字典定义可以了解到存储的核心实现就是将可绑定属性和对应的值存入到对象的字典里，例如给某个可绑定对象的某个叫 Xxx 的可绑定属性进行赋值，那将会对 `_properties` 字典更新 Xxx 属性的值内容

在 MAUI 的实现是，在可绑定对象里面，使用 SetValueCore 方法进行属性更新赋值，我删掉了不关键的逻辑的代码如下

```csharp
        internal void SetValueCore(BindableProperty property, object value, SetValueFlags attributes, SetValuePrivateFlags privateAttributes)
        {
            // 获取或创建可绑定属性上下文信息
            BindablePropertyContext context = GetOrCreateContext(property);

            SetValueActual(property, context, value, currentlyApplying, attributes, silent);
        }

        BindablePropertyContext GetOrCreateContext(BindableProperty property) => GetContext(property) ?? CreateAndAddContext(property);

        internal BindablePropertyContext GetContext(BindableProperty property) => _properties.TryGetValue(property, out var result) ? result : null;

        BindablePropertyContext CreateAndAddContext(BindableProperty property)
        {
            var context = new BindablePropertyContext { ... };
            _properties.Add(property, context);
            return context;
        }


        void SetValueActual(BindableProperty property, BindablePropertyContext context, object value, bool currentlyApplying, SetValueFlags attributes, bool silent = false)
        {
            // 触发对象变更前事件

            context.Value = value;

            // 触发对象已变更事件
        }
```

可以看到赋值的第一步就是调用 GetOrCreateContext 方法，尝试去拿到上下文信息，如果拿不到就创建。这里的用到的 BindablePropertyContext 上下文信息是存储可绑定属性的关键，在 BindablePropertyContext 里面存放了很多字段，定义如下

```csharp
    public abstract class BindableObject : INotifyPropertyChanged, IDynamicResourceHandler
    {
        internal class BindablePropertyContext
        {
            public BindableContextAttributes Attributes;
            public BindingBase Binding;
            public Queue<SetValueArgs> DelayedSetters;
            public BindableProperty Property;
            public object Value;

            public bool StyleValueSet;
            public object StyleValue;
        }
    }
```

可以看到 BindablePropertyContext 是一个内部类型，也不对外开放。在 BindablePropertyContext 里面重要的就是 `Value` 字段，表示存储的实际值内容。其次为了更好的支持绑定，也添加了 `Binding` 字段

在获取到 BindablePropertyContext 上下文之后，即可进行赋值，赋值是调用 SetValueActual 方法进行赋值，赋值前后分别触发事件用来通知。触发通知事件最重要的功能是让绑定可以有刷新的时机。如此即可完成赋值过程

通知事件是分别触发可绑定的对象的通知事件和对应的可绑定属性的通知事件，如下面代码

```csharp
        void SetValueActual(BindableProperty property, BindablePropertyContext context, object value, bool currentlyApplying, SetValueFlags attributes, bool silent = false)
        {
            // 触发对象变更前事件
            property.PropertyChanging?.Invoke(this, original, value);
            OnPropertyChanging(property.PropertyName);

            context.Value = value;

            // 触发对象已变更事件
            OnPropertyChanged(property.PropertyName);
            property.PropertyChanged?.Invoke(this, original, value);
        }
```

通过以上代码可以看到，可绑定对象给可绑定属性赋值的时候，就是先获取或创建可绑定属性上下文，将赋值的参数值给到 可绑定属性上下文 的 Value 字段。如此完成赋值过程

由于赋值的参数值被放入到 可绑定属性上下文 的 Value 字段，而 可绑定属性上下文 又放入到 `_properties` 字典里，相当于间接将 赋值的参数值 放入到 `_properties` 字典里。自然在获取值过程里，也需要从字典里面读取。在 MAUI 里面读取可绑定属性是通过 GetValue 方法实现，代码如下

```csharp
        public object GetValue(BindableProperty property)
        {
            if (property == null)
                throw new ArgumentNullException(nameof(property));

            var context = property.DefaultValueCreator != null ? GetOrCreateContext(property) : GetContext(property);

            return context == null ? property.DefaultValue : context.Value;
        }
```

以上代码的判断 BindableProperty 的 DefaultValueCreator 属性逻辑是 MAUI 特有的逻辑，和 WPF 与 UWP 不相同，咱下文再聊。回到获取属性的方法上，是通过先获取对象的可绑定上下文信息，如果能获取到可绑定上下文，证明此可绑定对象的这个可绑定属性曾经被赋值过，需要用赋值更新的内容。如果拿到的可绑定属性上下文是空，那就使用可绑定属性定义的默认值即可

在 MAUI 里面，通过 BindableProperty 的 DefaultValueCreator 属性简化了可绑定属性的定义，和让可绑定属性更加强大。使用 MAUI 的可绑定属性和可绑定对象对比 WPF 的依赖属性和依赖对象的实现，可以看到 MAUI 的实现实在简洁很多。在 MAUI 里的 BindableProperty 的 DefaultValueCreator 属性是一个委托，定义如下

```csharp
    public sealed class BindableProperty
    {
        public delegate object CreateDefaultValueDelegate(BindableObject bindable);

        internal CreateDefaultValueDelegate DefaultValueCreator { get; }
    }
```

可以看到 BindableProperty 的 DefaultValueCreator 属性的委托是支持给传入的可绑定对象进行处理，对可绑定对象返回特定的默认值。这里值得说明的是，通过委托是可以特例给可绑定对象不同的默认值的，但不代表着一定是不同的可绑定对象都一定需要不同的默认值对象。这里只是一个委托，让委托返回相同的对象是完全可以的。这个委托更多的是使用在判断可绑定对象类型，根据可绑定类型对象或者状态，返回不同的默认值。或者是返回一个需要运行时动态计算值，而不是一个可以写固定在代码里面的参数

例如对于 FontSize 的可绑定属性的定义里，就采用让不同的控件返回不同的默认字体大小，定义如下

```csharp
        public static readonly BindableProperty FontSizeProperty =
            BindableProperty.Create("FontSize", typeof(double), typeof(IFontElement), 0d,
                                    propertyChanged: OnFontSizeChanged,
                                    defaultValueCreator: FontSizeDefaultValueCreator);

        static object FontSizeDefaultValueCreator(BindableObject bindable)
            => ((IFontElement)bindable).FontSizeDefaultValueCreator();
```

也就是说对于不同的可绑定对象，获取到的默认的字体大小是根据对应的可绑定对象的 FontSizeDefaultValueCreator 方法实现决定，不同的可绑定对象可以有不同的实现，从而实现了让默认值关联上具体的可绑定对象类型。这个创新的设计，可以省掉在 WPF 里面的大量默认依赖属性值重写的逻辑代码，省掉了这部分代码，也可以大量减少的机制，从而减少更多的代码

例如 Span 和 Editor 控件对字体大小默认值有不同的实现

```csharp
    public class Span : GestureElement, IFontElement
    {
        double IFontElement.FontSizeDefaultValueCreator() =>
            double.NaN;
    }

    public partial class Button : View, IFontElement
    {
        double IFontElement.FontSizeDefaultValueCreator() =>
            this.GetDefaultFontSize();
    }
```

同样，对于某些可绑定属性来说，需要给每个可绑定对象的对象不同的默认值对象，例如 Grid 里面的 RowDefinitions 属性。大家都知道，在 Grid 里面的 RowDefinitions 是一个集合，如果集合也是一个共享的默认值，那自然会存在默认值污染。如果默认值是一个空值，那么将会让 Grid 逻辑里面存在大量的判断空逻辑，或者需要其他额外的初始化逻辑。在 MAUI 里面，通过 DefaultValueCreator 委托，实现了每个 Grid 对象使用独立的默认值对象，代码如下

```csharp
    public class Grid : Layout, IGridLayout
    {
        public static readonly BindableProperty RowDefinitionsProperty = BindableProperty.Create("RowDefinitions",
            typeof(RowDefinitionCollection), typeof(Grid), null, validateValue: (bindable, value) => value != null,
            propertyChanged: UpdateSizeChangedHandlers, defaultValueCreator: bindable =>
            {
                // 每个 Grid 对象使用独立的，新创建的默认值对象
                var rowDef = new RowDefinitionCollection();
                rowDef.ItemSizeChanged += ((Grid)bindable).DefinitionsChanged;
                return rowDef;
            });
    }
```

在 MAUI 里面除了可绑定属性之外，还有一个特殊的属性类型，附加属性。附加属性可以定义在任意的类型里面，通过附加属性，给某个现有的类型附加上属性。功能上和 WPF 或 UWP 的附加属性功能是相同的。可绑定属性和附加属性都是相同的 BindableProperty 类型，只是在创建的时候，调用的静态创建方法不同而已。对于可绑定属性来说，调用的是 `BindableProperty.Create` 方法创建。对于附加属性来说，调用 `BindableProperty.CreateAttached` 创建。在 MAUI 里面，通过阅读代码，我认为分开两个方法更多的是为了兼容 WPF 或 UWP 的写法，没有非常本质的差别，参数也差不多，如下面代码

```csharp
        internal static BindableProperty Create(string propertyName, Type returnType, [DynamicallyAccessedMembers(DeclaringTypeMembers)] Type declaringType, object defaultValue, BindingMode defaultBindingMode, ValidateValueDelegate validateValue,
                                                BindingPropertyChangedDelegate propertyChanged, BindingPropertyChangingDelegate propertyChanging, CoerceValueDelegate coerceValue, BindablePropertyBindingChanging bindingChanging,
                                                CreateDefaultValueDelegate defaultValueCreator = null)
        {
            return new BindableProperty(propertyName, returnType, declaringType, defaultValue, defaultBindingMode, validateValue, propertyChanged, propertyChanging, coerceValue, bindingChanging,
                defaultValueCreator: defaultValueCreator);
        }

        internal static BindableProperty CreateAttached(string propertyName, Type returnType, [DynamicallyAccessedMembers(DeclaringTypeMembers)] Type declaringType, object defaultValue, BindingMode defaultBindingMode, ValidateValueDelegate validateValue,
                                                        BindingPropertyChangedDelegate propertyChanged, BindingPropertyChangingDelegate propertyChanging, CoerceValueDelegate coerceValue, BindablePropertyBindingChanging bindingChanging,
                                                        bool isReadOnly, CreateDefaultValueDelegate defaultValueCreator = null)
        {
            return new BindableProperty(propertyName, returnType, declaringType, defaultValue, defaultBindingMode, validateValue, propertyChanged, propertyChanging, coerceValue, bindingChanging, isReadOnly,
                defaultValueCreator);
        }
```

如此可以看到可绑定属性和附加属性从参数上是似乎相同的。由于附加属性也是一个可绑定属性类型，同理可以了解到附加属性的存储也和可绑定对象的可绑定属性的存储是相同的。如此也能解答一个问题，在 MAUI 的附加属性，附加到对象上，附加属性的参数值是如何跟随对象的生命周期的问题。由于附加属性也是一个可绑定属性，同样将参数值存在可绑定对象的 `_properties` 字典里面，在对象会 GC 回收时，自然 `_properties` 字段也被回收，那放在字典里面的参数值也自然被减去引用，当参数值的没有被引用时，也就自然被回收

在 MAUI 里面，可绑定对象基类型的意义就是提供了可绑定属性的机制，存储可绑定属性的方式就是通过 `_properties` 字典存放。通过字典存放的内容是被赋值更改的属性，没有赋值更改的属性是没有被放入到字典里面，获取在字典里面没有存放的属性时，将会通过对应的可绑定属性获取到默认值。默认值的获取有两个方式，一个是可绑定属性的固定的默认值属性，另一个是通过可绑定属性的默认值创建委托创建默认值。在 MAUI 里的可绑定属性的默认值创建委托是一个创新，可以写出让不同的可绑定对象使用不同的默认值的功能，也可以写出根据不同的可绑定对象类型返回不同的默认值，通过委托的方式灵活实现复杂的功能

更多的 MAUI 相关博客，还请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。