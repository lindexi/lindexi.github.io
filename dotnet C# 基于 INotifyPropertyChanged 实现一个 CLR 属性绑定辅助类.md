# dotnet C# 基于 INotifyPropertyChanged 实现一个 CLR 属性绑定辅助类

习惯了 WPF 或 UWP 等的依赖属性的绑定机制之后，我在写 CLR 属性时，有时也期望将两个 CLR 属性给绑定到一起。在 dotnet 里，提供了 System.ComponentModel.INotifyPropertyChanged 接口，用于让某个类型约定了属性变更通知机制。于是有了这个基础，即可实现 CLR 属性的单向和双向绑定，核心原理就是在收到 INotifyPropertyChanged 的属性变更事件之后，更改绑定方的属性

<!--more-->

<!-- 发布 -->
<!-- 博客 -->

本文将告诉大家我实现的一个 CLR 属性绑定辅助类，包括实现的思路和对应的用法。本文属于新手友好博客。阅读本文，能让大家知道 dotnet 里面的 INotifyPropertyChanged 的设计以及绑定的用法，以及了解到如何使用和实现 CLR 属性绑定辅助类。我实现的 CLR 属性绑定辅助类和对应的全部代码，都可以在本文末尾找到代码的下载方式

本文的编写顺序是先告诉大家基础的概念定义，接着是如何编写实现 CLR 属性绑定辅助类，最后是此属性绑定辅助类的使用方法和获取源代码的方法

本文这里的 CLR 属性是和 WPF 或 UWP 等的依赖属性做对应的属性，也就是说 CLR 属性就是在脱离这些框架之外，咱日常写的普通 C# 属性。例如 `public string AProperty1 { set; get; }` 这样的属性。本文不会涉及到任何依赖属性的知识，如之前没有了解过依赖属性，那么对阅读文本来说也不会存在问题

开始之前，由于不能假定大家都是玩过 WPF 或者 UWP 或者是 MAUI 等应用框架，熟悉了属性绑定模式的玩法。于是我准备先介绍一下属性绑定的基础使用思路，所谓属性绑定就是将两个属性关联绑定在一起，当一个属性变更时，另一个属性也可以随着进行变更。更细节来说，还涉及到两个属性的类型不同问题，需要在两个属性变更更新时进行一些转换逻辑等。绑定的双方对象，大部分情况是两个不同的对象，但是也不禁止给相同的一个对象的不同两个属性进行绑定

例如以下的伪代码例子

```
// 先假定有 A 和 B 两个对象

双向绑定 A.属性1 和 B.属性1

// 此时 A 和 B 的属性就建立绑定关系了

更新 A.属性1

// 更新了 A 的属性，自然 B 的属性也会随着更新。反过来也是
```

至于双向绑定和单向绑定的差别只是在于，绑定的双方的更新方向而已。双向绑定的意思就是绑定的两个属性，无论是哪个属性更新了，另一个属性都会跟着更新。而单向绑定仅仅只是从 Source 源的一方属性，更新到 Target 目标的一方的属性而已，如果是 Target 目标的一方的属性变更了则不会更新到 Source 的一方

这就是属性绑定的大概玩法。当然，在 WPF 或者 UWP 或者是 MAUI 等应用框架里面，绑定还能有很多有趣的玩法，可以带来新的开发思路。本文这里就不展开了，大家如有兴趣，可以随意搜属性绑定的玩法，了解更多有趣的用法。尽管搜索引擎上大量的关于属性绑定的都是属于 WPF 或 UWP 等的依赖属性或附加属性等的绑定，但玩法都是差不多的，相互之间可以借鉴。额外需要说的是，进行 CLR 属性绑定的方法非我首创，此前已存在很多大佬们写过方法，只不过 CLR 属性绑定在搜索引擎上被依赖属性绑定等给淹没了

现在开始来实现 CLR 的属性绑定辅助类的编写

在 dotnet 里，提供的 System.ComponentModel.INotifyPropertyChanged 接口，此接口要求在属性变更的时候，触发 PropertyChanged 事件，如此即可让外部的代码通过监听 PropertyChanged 事件，了解到属性的变更。这里的外部指的是类型的外部，相当于其他的类型

一个简单的实现如以下代码

```csharp
class A : INotifyPropertyChanged
{
    public string AProperty1
    {
        set
        {
            _aProperty1 = value;
            OnPropertyChanged();
        }
        get => _aProperty1;
    }

    private string _aProperty1 = string.Empty;

    public event PropertyChangedEventHandler? PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

以上代码里面的 PropertyChanged 就是 INotifyPropertyChanged 接口约束的事件。以上代码的 OnPropertyChanged 的 `propertyName` 参数里，用到了编译器的黑科技，通过 `[CallerMemberName]` 特性，即可让 `propertyName` 参数自动填充调用方的名字。例如在 AProperty1 属性里面调用了 OnPropertyChanged 方法，在编译时，将会自动填补入参 `propertyName` 为 `nameof(AProperty1)` 即 `"AProperty1"` 字符串。也就是编译后的 AProperty1 的代码如下

```csharp
    public string AProperty1
    {
        set
        {
            _aProperty1 = value;
            OnPropertyChanged("AProperty1");
        }
        get => _aProperty1;
    }
```

实现 CLR 属性绑定的能力，也就是通过监听 INotifyPropertyChanged 约束的 PropertyChanged 事件，了解到属性的变更，以及是哪个属性发生了变更，从而进行更新

在绑定里面，约定了两个概念，分别是 Source 源和 Target 目标。绑定就是一个从 Source 源绑定到 Target 目标的过程。也就是说从 Source 源对象的 Source 源属性，绑定到 Target 目标对象的 Target 目标属性的过程，就是绑定的过程

先编写监听 Source 源的属性变更 PropertyChanged 事件的代码，通过此代码可以辅助大家了解此 INotifyPropertyChanged 约束的用法。为了减少引入太多概念，这里假定 Source 源对象的就是采用以上代码定义的 A 类型

```csharp
var a = new A();
var source = a;
INotifyPropertyChanged sourceNotifyPropertyChanged = source;

sourceNotifyPropertyChanged.PropertyChanged += Source_OnPropertyChanged;

private void Source_OnPropertyChanged(object? sender, PropertyChangedEventArgs e)
{
    ... // 忽略代码
}
```

在 `Source_OnPropertyChanged` 里面，即可通过 PropertyChangedEventArgs 类型的参数，了解当前 Source 对象变更的属性的属性名，例如进行输出属性名

```csharp
private void Source_OnPropertyChanged(object? sender, PropertyChangedEventArgs e)
{
    Log($"变更的属性名 {e.PropertyName}");
}
```

只是知道变更的属性名，那如何知道变更的属性名对应的属性变更之后的值呢？这里有两个通用的方法，第一个方法就是通过反射获取，既然知道了对象和属性名，即可通过反射获取属性的值。第二个方法就是让上层调用方传入获取此属性值的委托了

先来看看第一个方法，通过反射获取的实现

假定有对象名为 `bindableObject` 对象，获取此对象的名为 `propertyName` 的属性的属性值，可以如此编写代码

```csharp
// 反射需要获取某个类型的某个属性的 get 方法
// 先获取类型的属性
var propertyInfo = bindableObject.GetType().GetProperty(propertyName);

// 获取到属性之后，获取属性的 get 方法
var getMethod = propertyInfo?.GetGetMethod();

// 获取 get 方法之后，可以通过反射调用 get 方法获取属性值
var value = getMethod.Invoke(bindableObject, null);
```

由于 `getMethod` 是一个 MethodInfo 对象，可以缓存起来，不需要每次都获取，代码可以进行一点点优化，例如定义一个委托存放起来

```csharp
public delegate object? PropertyGetter(object bindableObject);

private PropertyGetter? _internalPropertyGetter;

    _internalPropertyGetter = o => getMethod.Invoke(o, null);
```

如此通过 `_internalPropertyGetter` 委托，就可以使用 `var value = _internalPropertyGetter!.Invoke(bindableObject)` 获取给定的对象的属性值。如果期望有更高的性能，也可以尝试构建表达式树等方法

如上面代码，可以看到需要的是一个 `PropertyGetter` 类型的委托，完全可以让上层调用时传入委托的实现，如此可以更加灵活。也就是第二个方法就是不通过默认的反射创建 `PropertyGetter` 方法，是通过上层调用传入委托实现

可以将两个方法合并，这样可以让上层调用可选传入委托的实现，如果上层调用没有传入，那就走默认的反射

```csharp
    public PropertyGetter? PropertyGetter { get; }

    internal PropertyGetter? InternalPropertyGetter
    {
        get
        {
            if (_internalPropertyGetter is not null)
            {
                return _internalPropertyGetter;
            }

            _internalPropertyGetter ??= PropertyGetter;

            ...// 忽略一些代码

            if (_internalPropertyGetter is null)
            {
                // 这里的 bindableObject 就是一个对象，期望从此对象里获取属性的值。具体的定义在下文将会给出
                var propertyInfo = bindableObject.GetType().GetProperty(Path);
                var getMethod = propertyInfo?.GetGetMethod();
                if (getMethod != null)
                {
                    _internalPropertyGetter = o => getMethod.Invoke(o, null);
                }
            }

            return _internalPropertyGetter;
        }
    }

    private PropertyGetter? _internalPropertyGetter;

public delegate object? PropertyGetter(object bindableObject);
```

以上代码的 `PropertyGetter` 属性就是上层业务端传入的委托实现。而 `InternalPropertyGetter` 就是内部真正使用的委托实现。在 InternalPropertyGetter 里判断如果有上层传入的委托实现就使用此委托实现，如果没有就使用反射的方式创建委托

同理，在获取变更的属性的属性值之后，需要将此变更的属性值赋值给到绑定的目标属性上。给属性的赋值也是如同属性的获取属性值，要么走反射调用赋值方法，要么让上层调用传入属性赋值的委托实现。代码上和上面的代码差不多

再加上无论是 Source 还是 Target 都需要类似的一套代码，如在双向绑定上，对于 Source 属性来说既需要获取值，也可以被赋值。如此可以封装一个类型，方便减少重复的代码。这里封装了 ClrBindingPropertyContext 类型用来存放属性信息

先了解一下 ClrBindingPropertyContext 的需求，首先绑定需要对象本身，但是最好不要对此对象进行强引用，防止属性一直不能被回收。这是基于如 WPF 或 UWP 的设计来说，绑定的时候，不能让绑定影响对象的引用关系，防止开发者不小心将对象绑定到静态等情况下，造成内存泄露。于是存放对象使用的是弱引用的方式。接着需要封装属性的获取和设置方法和属性的 Path 值。在这里的 Path 值是学习 WPF 或 UWP 的设计，约等于属性名的意思。也就是此 ClrBindingPropertyContext 是给哪个对象的哪个属性使用的

```csharp
public class ClrBindingPropertyContext
{
    public ClrBindingPropertyContext(object bindableObject, string path,
        PropertyGetter? propertyGetter = null,
        PropertySetter? propertySetter = null)
    {
        BindableObjectWeakReference = new WeakReference<object>(bindableObject);
        Path = path;
        PropertyGetter = propertyGetter;
        PropertySetter = propertySetter;
    }

    public WeakReference<object> BindableObjectWeakReference { get; }
    public string Path { get; }
    public PropertyGetter? PropertyGetter { get; }
    public PropertySetter? PropertySetter { get; }

    internal PropertyGetter? InternalPropertyGetter
    {
        get
        {
            if (_internalPropertyGetter is not null)
            {
                return _internalPropertyGetter;
            }

            _internalPropertyGetter ??= PropertyGetter;

            if (_internalPropertyGetter is null && BindableObjectWeakReference.TryGetTarget(out var bindableObject))
            {
                var propertyInfo = bindableObject.GetType().GetProperty(Path);
                var getMethod = propertyInfo?.GetGetMethod();
                if (getMethod != null)
                {
                    _internalPropertyGetter = o => getMethod.Invoke(o, null);
                }
            }

            return _internalPropertyGetter;
        }
    }

    private PropertyGetter? _internalPropertyGetter;

    internal PropertySetter? InternalPropertySetter
    {
        get
        {
            if (_internalPropertySetter is not null)
            {
                return _internalPropertySetter;
            }

            _internalPropertySetter ??= PropertySetter;

            if (_internalPropertySetter is null && BindableObjectWeakReference.TryGetTarget(out var bindableObject))
            {
                var propertyInfo = bindableObject.GetType().GetProperty(Path);
                var setMethod = propertyInfo?.GetSetMethod();
                if (setMethod != null)
                {
                    _internalPropertySetter = (o, value) => setMethod.Invoke(o, new object?[] { value });
                }
            }

            return _internalPropertySetter;
        }
    }
    private PropertySetter? _internalPropertySetter;
}

public delegate object? PropertyGetter(object bindableObject);

public delegate void PropertySetter(object bindableObject, object? propertyValue);
```

有关弱引用的概念，如果不熟悉的话，还请自行了解，这不是本文的重点。通过以上的代码即可封装出对象的属性绑定基础内容，可以用在 Source 和 Target 上。而且上层业务方调用的时候，可以看到，是可选传入 PropertyGetter 等委托的

接着在实现绑定的核心代码之前，还需要定义一个枚举值，此枚举用来表示建立绑定时的行为是什么。在建立绑定时可选的行为如下

- 将 Source 的值立刻赋值给到 Target 属性
- 啥都不做，等待属性更新之后再决定赋值
- 将 Target 的值立刻赋值给到 Source 属性

默认行为继续学习 WPF 或 UWP 等的设计，选用将 Source 的值立刻赋值给到 Target 属性作为默认行为，这也是符合大部分的使用的。建立绑定很多时候就是为了让两个属性同步，在建立完成之后，期望就是同步的，而且是从 Source 赋值给到 Target 属性

定义的枚举代码如下

```csharp
/// <summary>
/// 绑定初始化时的值传递方向
/// </summary>
public enum BindingInitMode
{
    /// <summary>
    /// 使用被绑定对象的值设置当前对象的值（默认）
    /// </summary>
    SourceToTarget = 0,

    /// <summary>
    /// 啥都不干
    /// </summary>
    None = 1,

    /// <summary>
    /// 使用当前对象的值设置被绑定对象的值
    /// </summary>
    TargetToSource = 2,
}
```

完成基础类型定义之后，可以开始定义 ClrBidirectionalBinding 绑定辅助类的构造函数了

```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    public ClrBidirectionalBinding(ClrBindingPropertyContext source, ClrBindingPropertyContext target,
        BindingDirection direction = BindingDirection.TwoWay,
        //IClrValueConverter? converter = null,
        BindingInitMode initMode = BindingInitMode.SourceToTarget)
    {
        ... // 忽略代码
    }

    ... // 忽略代码
}
```

以上代码的 `BindingDirection` 是 dotnet 内置的，表示绑定的方式是单向还是双向绑定。单向绑定就是从 Source 到 Target 的单向绑定，只有 Source 属性的更新会更新给 Target 属性，而 Target 属性的更新不会更新 Source 属性。双向的话就是两个属性无论哪个更新都会让另一个属性更新

```csharp
namespace System.ComponentModel
{
    /// <summary>
    /// Specifies whether the template can be bound one-way or two-way.
    /// </summary>
    public enum BindingDirection
    {
        /// <summary>
        /// The template can only accept property values. Used with a generic ITemplate.
        /// </summary>
        OneWay = 0,

        /// <summary>
        /// The template can accept and expose property values. Used with an IBindableTemplate.
        /// </summary>
        TwoWay = 1
    }
}
```

以上代码被我注释调用的是 `IClrValueConverter` 属性，这个属性原本是在 WPF 或 UWP 等框架里使用多个绑定值之间的转换器，例如绑定的两个属性的类型是不相同的，这就需要进行一些转换。然而在 CLR 绑定上，似乎不需要再加入这样的类型，原因就是赋值方法可以是传入委托，既然都可以传入委托了，那就在属性赋值的委托上自己想怎么转换就怎么玩

开始先将 source 和 target 存起来，为了存放起来，还需要再定义两个属性

```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    public ClrBidirectionalBinding(ClrBindingPropertyContext source, ClrBindingPropertyContext target,
        BindingDirection direction = BindingDirection.TwoWay,
        //IClrValueConverter? converter = null,
        BindingInitMode initMode = BindingInitMode.SourceToTarget)
    {
    	Source = source;
        Target = target;
        ... // 忽略代码
    }

    ... // 忽略代码

    public ClrBindingPropertyContext Source { get; }
    public ClrBindingPropertyContext Target { get; }
}
```

由于传入的是弱引用，顺带判断一下对象是否存活，如果绑定的对象其中存在一个被回收了，那自然绑定也就不需要成立了

```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    public ClrBidirectionalBinding(ClrBindingPropertyContext source, ClrBindingPropertyContext target,
        BindingDirection direction = BindingDirection.TwoWay,
        //IClrValueConverter? converter = null,
        BindingInitMode initMode = BindingInitMode.SourceToTarget)
    {
    	Source = source;
        Target = target;

        if (!source.BindableObjectWeakReference.TryGetTarget(out var sourceObject)
            || !target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
        {
            return;
        }
        ... // 忽略代码
    }

    ... // 忽略代码
}
```

这个判断弱引用所引用的对象是否存在的方法就是尝试去获取其对象，可以获取成功，证明对象还没有被回收。此方法也可以用在测量某个对象是否回收上。调用 TryGetTarget 是一个非常快速的过程，几乎不需要担心性能问题。通过这个方式，也可以定义出判断此绑定是否还生效的方法。判断是否还生效那就是判断绑定的对象是否没有被回收

```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    ... // 忽略代码

    public bool IsAlive() =>
        Source.BindableObjectWeakReference.TryGetTarget(out _)
        && Target.BindableObjectWeakReference.TryGetTarget(out _);
}
```

由于绑定是肯定要求订阅 Source 对象的属性变更通知的，于是可以先监听 Source 的对象变更事件


```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    public ClrBidirectionalBinding(ClrBindingPropertyContext source, ClrBindingPropertyContext target,
        BindingDirection direction = BindingDirection.TwoWay,
        //IClrValueConverter? converter = null,
        BindingInitMode initMode = BindingInitMode.SourceToTarget)
    {
    	Source = source;
        Target = target;

        if (!source.BindableObjectWeakReference.TryGetTarget(out var sourceObject)
            || !target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
        {
            return;
        }

        f (sourceObject is not INotifyPropertyChanged sourceNotifyPropertyChanged)
        {
            throw new ArgumentException("Source not implement interface INotifyPropertyChanged");
        }

        sourceNotifyPropertyChanged.PropertyChanged += Source_OnPropertyChanged;
        ... // 忽略代码
    }

    private void Source_OnPropertyChanged(object? sender, PropertyChangedEventArgs e)
    {
        ... // 忽略代码
    }

    ... // 忽略代码
}
```

现在先忽略 `Source_OnPropertyChanged` 方法的实现，再来看看 Target 的属性变更是否需要监听。是否需要监听 Target 的属性变更，取决于是否双向绑定。如果非双向绑定那就不要求监听，也就不要求 Target 继承 INotifyPropertyChanged 接口

```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    public ClrBidirectionalBinding(ClrBindingPropertyContext source, ClrBindingPropertyContext target,
        BindingDirection direction = BindingDirection.TwoWay,
        //IClrValueConverter? converter = null,
        BindingInitMode initMode = BindingInitMode.SourceToTarget)
    {
    	... // 忽略代码

        if (direction == BindingDirection.OneWay)
        {
            // 单向，不需要监听 Target 的事件
        }
        else if (direction == BindingDirection.TwoWay)
        {
            ... // 忽略代码

            // 需要监听 Target 的事件
            if (targetObject is not INotifyPropertyChanged targetNotifyPropertyChanged)
            {
                throw new ArgumentException("Target not implement interface INotifyPropertyChanged");
            }

            targetNotifyPropertyChanged.PropertyChanged += Target_OnPropertyChanged;
        }
        ... // 忽略代码
    }

    private void Target_OnPropertyChanged(object? sender, PropertyChangedEventArgs e)
    {
        ... // 忽略代码
    }

    ... // 忽略代码
}
```

接着根据初始化 initMode 的方式，决定是否绑定时立刻更新属性

```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    public ClrBidirectionalBinding(ClrBindingPropertyContext source, ClrBindingPropertyContext target,
        BindingDirection direction = BindingDirection.TwoWay,
        //IClrValueConverter? converter = null,
        BindingInitMode initMode = BindingInitMode.SourceToTarget)
    {
    	... // 忽略代码

        switch (initMode)
        {
            case BindingInitMode.SourceToTarget:
                SetSourceToTarget();
                break;
            case BindingInitMode.None:
                break;
            case BindingInitMode.TargetToSource:
                SetTargetToSource();
                break;
            default:
                throw new ArgumentOutOfRangeException();
        }

        ... // 忽略代码
    }

    private void SetSourceToTarget()
    {
        ... // 忽略代码
    }

    private void SetTargetToSource()
    {
        ... // 忽略代码
    }

    ... // 忽略代码
}
```

回到属性值变更方法，属性值更新时，将判断更新的属性是否当前绑定的属性，如果是，那就调用对应的更新方法。如 Source 的属性变更，就更新 Target 的属性。反过来则更新 Source 的属性

```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    private void Source_OnPropertyChanged(object? sender, PropertyChangedEventArgs e)
    {
        if (_isInnerSet)
        {
            return;
        }

        if (string.Equals(e.PropertyName, Source.Path, StringComparison.Ordinal))
        {
            SetSourceToTarget();
        }

        ... // 忽略代码
    }

    private void Target_OnPropertyChanged(object? sender, PropertyChangedEventArgs e)
    {
        if (_isInnerSet)
        {
            return;
        }

        if (string.Equals(e.PropertyName, Target.Path, StringComparison.Ordinal))
        {
            SetTargetToSource();
        }

        ... // 忽略代码
    }

    private bool _isInnerSet;

    ... // 忽略代码
}
```

由于 `Target_OnPropertyChanged` 属于按需监听，于是只要 `Target_OnPropertyChanged` 方法被调用，那就更新 Source 的属性，不需要额外判断绑定方向

上面代码的 `_isInnerSet` 是为了防止属性重复变更，例如当 Source 属性变更时，更新 Target 属性，从而导致 `Target_OnPropertyChanged` 方法被调用，再次去更新 Source 属性。这个字段将在 `SetSourceToTarget` 和 `SetTargetToSource` 方法里面被赋值，接下来看看这两个方法的实现

对于 `SetSourceToTarget` 来说，需要做的就是从 Source 里面获取属性值，赋值给到 Target 属性上

```csharp
public class ClrBidirectionalBinding
{
    ... // 忽略代码

    private void SetSourceToTarget()
    {
        _isInnerSet = true;

        try
        {
            if (!Source.BindableObjectWeakReference.TryGetTarget(out var sourceObject)
                || !Target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
            {
                BreakBinding();
                return;
            }

            var sourceValue = Source.InternalPropertyGetter!.Invoke(sourceObject);
            Target.InternalPropertySetter!.Invoke(targetObject, sourceValue);
        }
        finally
        {
            _isInnerSet = false;
        }
    }

    ... // 忽略代码
}
```

在开始更新之前，还需要先判断一下绑定是否依然成立，判断 Source 和 Target 对象是否被回收，被回收了就调用 `BreakBinding` 断开绑定关系，断开绑定关系之后，此 `ClrBidirectionalBinding` 将可以减少被引用关系，方便被 GC 回收。当然了，如果还有其他代码引用此 `ClrBidirectionalBinding` 那就不会被回收了

```csharp
public class ClrBidirectionalBinding
{
    ... // 忽略代码

    /// <summary>
    /// 断开绑定
    /// </summary>
    public void BreakBinding()
    {
        if (Source.BindableObjectWeakReference.TryGetTarget(out var sourceObject))
        {
            if (sourceObject is INotifyPropertyChanged sourceNotifyPropertyChanged)
            {
                sourceNotifyPropertyChanged.PropertyChanged -= Source_OnPropertyChanged;
            }
        }

        if (Target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
        {
            if (targetObject is INotifyPropertyChanged targetNotifyPropertyChanged)
            {
                targetNotifyPropertyChanged.PropertyChanged -= Target_OnPropertyChanged;
            }
        }
    }

    ... // 忽略代码
}
```

同理实现 `SetTargetToSource` 方法

```csharp
public class ClrBidirectionalBinding
{
    ... // 忽略代码

    private void SetTargetToSource()
    {
        _isInnerSet = true;

        try
        {
            if (!Source.BindableObjectWeakReference.TryGetTarget(out var sourceObject)
                || !Target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
            {
                BreakBinding();
                return;
            }

            var targetValue = Target.InternalPropertyGetter!.Invoke(targetObject);
            Source.InternalPropertySetter!.Invoke(sourceObject, targetValue);
        }
        finally
        {
            _isInnerSet = false;
        }
    }

    ... // 忽略代码
}
```

以上就是绑定的核心实现方法

实现差不多了，先来看看使用方法

先定义 A 和 B 两个用来演示的类型

```csharp
class A : INotifyPropertyChanged
{
    public string AProperty1
    {
        set
        {
            if (value == _aProperty1) return;
            _aProperty1 = value;
            OnPropertyChanged();
        }
        get => _aProperty1;
    }

    private string _aProperty1 = string.Empty;

    public int AProperty2
    {
        set
        {
            if (value == _aProperty2) return;
            _aProperty2 = value;
            OnPropertyChanged();
        }
        get => _aProperty2;
    }
    private int _aProperty2;

    public string? APropertyWithoutSet { get; }

    public string? APropertyWithoutGet
    {
        set
        {
            if (value == _aPropertyWithoutGet) return;
            _aPropertyWithoutGet = value;
            OnPropertyChanged();
        }
    }
    private string? _aPropertyWithoutGet;

    public event PropertyChangedEventHandler? PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetField<T>(ref T field, T value, [CallerMemberName] string? propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(field, value)) return false;
        field = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}

class B : INotifyPropertyChanged
{

    public string BProperty1
    {
        get => _bProperty1;
        set
        {
            if (value == _bProperty1) return;
            _bProperty1 = value;
            OnPropertyChanged();
        }
    }
    private string _bProperty1 = string.Empty;

    public int BProperty2
    {
        get => _bProperty2;
        set
        {
            if (value == _bProperty2) return;
            _bProperty2 = value;
            OnPropertyChanged();
        }
    }
    private int _bProperty2;

    public string BPropertyWithoutSet { get; }

    public string? BPropertyWithoutGet
    {
        set
        {
            if (value == _bPropertyWithoutGet) return;
            _bPropertyWithoutGet = value;
            OnPropertyChanged();
        }
    }
    private string? _bPropertyWithoutGet;

    public event PropertyChangedEventHandler? PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetField<T>(ref T field, T value, [CallerMemberName] string? propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(field, value)) return false;
        field = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}
```

先来试试单向绑定的效果，也就是给 A 和 B 两个对象设置单向绑定，可以在 A 更新时，给 B 赋值；在 B 更新时，啥都不做

```csharp
            var a = new A();
            var b = new B();

            // 给 A 和 B 两个对象设置单向绑定
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1),
                    propertyGetter: bindableObject => ((A) bindableObject).AProperty1),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1),
                    propertySetter: (bindableObject, value) => ((B) bindableObject).BProperty1 = (string) value!),
                BindingDirection.OneWay
            );

            // 可以在 A 更新时，给 B 赋值
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);

            // 在 B 更新时，啥都不做
            b.BProperty1 = Guid.NewGuid().ToString();
            Assert.AreEqual(value, a.AProperty1);
```

以上代码是放在单元测试里面跑的，一份还能用的代码是需要有单元测试来辅助测试功能的

以上代码是给定了属性获取和属性赋值的委托，看起来代码比较长，接下来不给委托，使用默认的反射实现

```csharp
            // 给 A 和 B 两个对象设置绑定，没有给定默认的赋值和获取值委托
            var a = new A();
            var b = new B();

            // 给 A 和 B 两个对象设置单向绑定
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                BindingDirection.OneWay
            );

            // 可以自动生成，等于绑定生效
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);
```

这代码看起来就简单很多了

再试试双向绑定，给 A 和 B 两个对象设置双向绑定，无论哪个对象更改，都能更新

```csharp
            // 给 A 和 B 两个对象设置双向绑定
            var a = new A();
            var b = new B();
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                BindingDirection.TwoWay
            );

            // 无论哪个对象更改，都能更新
            // 先设置 A 的属性，再测试设置 B 的属性
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);

            // 多次设置
            value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);

            value = Guid.NewGuid().ToString();
            b.BProperty1 = value;
            Assert.AreEqual(value, a.AProperty1);

            // 多次设置
            value = Guid.NewGuid().ToString();
            b.BProperty1 = value;
            Assert.AreEqual(value, a.AProperty1);

            // 在 B 设置完成之后，再次设置 A 的属性
            value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);
```

绑定只对绑定的属性生效，如果修改了对象里非当前绑定的属性，那什么都没有发生

```csharp
            // 设置 A 和 B 属性绑定
            var a = new A();
            var b = new B();
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1))
            );

            // 更改非绑定属性，不会影响原有的属性
            a.AProperty2 = 10;

            Assert.AreEqual(string.Empty, b.BProperty1);
            Assert.AreEqual(0, b.BProperty2);

            // 更新属性之后，依然也是不会影响
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;
            Assert.AreEqual(value, b.BProperty1);

            a.AProperty2 = 20;
            Assert.AreEqual(value, b.BProperty1);
            Assert.AreEqual(0, b.BProperty2);
```

这个就是基础的用法了，其他玩法可以参考 WPF 或 UWP 等的绑定的使用思路

本文的 `ClrBidirectionalBinding` 实现代码如下

```csharp
/// <summary>
/// 实现两个 CLR 属性的双向绑定
/// </summary>
public class ClrBidirectionalBinding
{
    public ClrBidirectionalBinding(ClrBindingPropertyContext source, ClrBindingPropertyContext target,
        BindingDirection direction = BindingDirection.TwoWay,
        //IClrValueConverter? converter = null,
        BindingInitMode initMode = BindingInitMode.SourceToTarget)
    {
        Source = source;
        Target = target;
        //Direction = direction;
        //InitMode = initMode;

        //ValueConverter = converter;

        if (!source.BindableObjectWeakReference.TryGetTarget(out var sourceObject)
            || !target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
        {
            return;
        }

        if (sourceObject is not INotifyPropertyChanged sourceNotifyPropertyChanged)
        {
            throw new ArgumentException("Source not implement interface INotifyPropertyChanged");
        }

        sourceNotifyPropertyChanged.PropertyChanged += Source_OnPropertyChanged;
        bool needSourceGetter = true; // 这是一定的，绑定的时候，需要 Source 的 Getter 方法
        bool needSourceSetter = false; // 如果不是 TwoWay 或者是 TargetToSource 那就不需要 Source 的 Setter 方法
        bool needTargetGetter = false; // 如果不是  TwoWay 或者是 TargetToSource 那就不需要 Target 的 Getter 方法
        bool needTargetSetter = true; // 这是一定的，绑定的时候，需要 Target 的 Setter 方法

        if (direction == BindingDirection.OneWay)
        {
            // 单向，不需要监听 Target 的事件
        }
        else if (direction == BindingDirection.TwoWay)
        {
            needSourceSetter = true;
            needTargetGetter = true;

            // 需要监听 Target 的事件
            if (targetObject is not INotifyPropertyChanged targetNotifyPropertyChanged)
            {
                throw new ArgumentException("Target not implement interface INotifyPropertyChanged");
            }

            targetNotifyPropertyChanged.PropertyChanged += Target_OnPropertyChanged;
        }

        // 初始化赋值
        switch (initMode)
        {
            case BindingInitMode.SourceToTarget:
                needSourceGetter = true;
                needTargetSetter = true;
                break;
            case BindingInitMode.None:
                break;
            case BindingInitMode.TargetToSource:
                needTargetGetter = true;
                needSourceSetter = true;
                break;
            default:
                throw new ArgumentOutOfRangeException();
        }

        if (needSourceGetter && source.InternalPropertyGetter is null)
        {
            throw new ArgumentNullException("source.PropertyGetter");
        }

        if (needSourceSetter && source.InternalPropertySetter is null)
        {
            throw new ArgumentNullException("source.PropertySetter");
        }

        if (needTargetGetter && target.InternalPropertyGetter is null)
        {
            throw new ArgumentNullException("target.PropertyGetter");
        }

        if (needTargetSetter && target.InternalPropertySetter is null)
        {
            throw new ArgumentNullException("target.PropertySetter");
        }

        switch (initMode)
        {
            case BindingInitMode.SourceToTarget:
                SetSourceToTarget();
                break;
            case BindingInitMode.None:
                break;
            case BindingInitMode.TargetToSource:
                SetTargetToSource();
                break;
            default:
                throw new ArgumentOutOfRangeException();
        }
    }

    public static void SetBinding(ClrBindingPropertyContext source, ClrBindingPropertyContext target,
        BindingDirection direction = BindingDirection.TwoWay,
        //IClrValueConverter? converter = null,
        BindingInitMode initMode = BindingInitMode.SourceToTarget) =>
        // 建立绑定关系即可，不需要存放此对象
        _ = new ClrBidirectionalBinding(source, target, direction, initMode);

    public static void SetBinding(object source, string sourcePropertyPath,
        object target, string targetPropertyPath, BindingDirection direction = BindingDirection.TwoWay,
        BindingInitMode initMode = BindingInitMode.SourceToTarget,
        PropertyGetter? sourcePropertyGetter = null, PropertySetter? sourcePropertySetter = null,
        PropertyGetter? targetPropertyGetter = null, PropertySetter? targetPropertySetter = null)
        => SetBinding
        (
            new ClrBindingPropertyContext(source, sourcePropertyPath, sourcePropertyGetter, sourcePropertySetter),
            new ClrBindingPropertyContext(target, targetPropertyPath, targetPropertyGetter, targetPropertySetter),
            direction,
            initMode
        );

    public static void SetOneWayBinding(object source, string sourcePropertyPath,
        object target, string targetPropertyPath, BindingInitMode initMode = BindingInitMode.SourceToTarget,
        PropertyGetter? sourcePropertyGetter = null,
        PropertySetter? targetPropertySetter = null) =>
        SetBinding(new ClrBindingPropertyContext(source, sourcePropertyPath, sourcePropertyGetter),
            new ClrBindingPropertyContext(target, targetPropertyPath, propertySetter: targetPropertySetter),
            BindingDirection.OneWay, initMode);

    private void Source_OnPropertyChanged(object? sender, PropertyChangedEventArgs e)
    {
        if (_isInnerSet)
        {
            return;
        }

        if (string.Equals(e.PropertyName, Source.Path, StringComparison.Ordinal))
        {
            _lastUpdateTick = Environment.TickCount64;
            SetSourceToTarget();
        }
        else
        {
            CheckAlive();
        }
    }

    private void Target_OnPropertyChanged(object? sender, PropertyChangedEventArgs e)
    {
        if (_isInnerSet)
        {
            return;
        }

        if (string.Equals(e.PropertyName, Target.Path, StringComparison.Ordinal))
        {
            _lastUpdateTick = Environment.TickCount64;
            SetTargetToSource();
        }
        else
        {
            CheckAlive();
        }
    }

    private void CheckAlive()
    {
        // 距离上次更新的毫秒数
        var current = Environment.TickCount64;
        var delta = current - _lastUpdateTick; // 理论上不会存在负数的，除非开机 （long.MaxValue / 1000 / 3600 / 24 / 365）= 292471208 年
        // 约定，大概 100 秒之后没有更新，那就判断一次是否存在吧
        if (delta > 100_1000)
        {
            // 放心，这个判断是非常快捷的，约等于获取一个属性，然后判断是否空
            if (!IsAlive())
            {
                // 如果不存活了，那就断开关系吧。断开关系之后，此对象即可被回收
                // 不存活的意思就是绑定的双方对象里面，超过一个对象已经被回收
                BreakBinding();
            }

            _lastUpdateTick = current;
        }
    }

    private void SetSourceToTarget()
    {
        _isInnerSet = true;

        try
        {
            if (!Source.BindableObjectWeakReference.TryGetTarget(out var sourceObject)
                || !Target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
            {
                BreakBinding();
                return;
            }

            var sourceValue = Source.InternalPropertyGetter!.Invoke(sourceObject);
            Target.InternalPropertySetter!.Invoke(targetObject, sourceValue);
        }
        finally
        {
            _isInnerSet = false;
        }
    }

    private void SetTargetToSource()
    {
        _isInnerSet = true;

        try
        {
            if (!Source.BindableObjectWeakReference.TryGetTarget(out var sourceObject)
                || !Target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
            {
                BreakBinding();
                return;
            }

            var targetValue = Target.InternalPropertyGetter!.Invoke(targetObject);
            Source.InternalPropertySetter!.Invoke(sourceObject, targetValue);
        }
        finally
        {
            _isInnerSet = false;
        }
    }

    private bool _isInnerSet;

    /// <summary>
    /// 最后更新时间
    /// </summary>
    private long _lastUpdateTick;

    public ClrBindingPropertyContext Source { get; }
    public ClrBindingPropertyContext Target { get; }

    //public IClrValueConverter? ValueConverter { get; }

    ///// <summary>
    ///// 绑定方向，默认 TwoWay。  
    ///// </summary>
    //public BindingDirection Direction { get; }

    ///// <summary>
    ///// 初始化模式，默认 SourceToTarget。
    ///// SourceToTarget：初始值以 Source 为准；TargetToSource：初始值以 Target 为准。
    ///// </summary>
    //public BindingInitMode InitMode { get; }

    public bool IsAlive() =>
        Source.BindableObjectWeakReference.TryGetTarget(out _)
        && Target.BindableObjectWeakReference.TryGetTarget(out _);

    /// <summary>
    /// 断开绑定
    /// </summary>
    public void BreakBinding()
    {
        if (Source.BindableObjectWeakReference.TryGetTarget(out var sourceObject))
        {
            if (sourceObject is INotifyPropertyChanged sourceNotifyPropertyChanged)
            {
                sourceNotifyPropertyChanged.PropertyChanged -= Source_OnPropertyChanged;
            }
        }

        if (Target.BindableObjectWeakReference.TryGetTarget(out var targetObject))
        {
            if (targetObject is INotifyPropertyChanged targetNotifyPropertyChanged)
            {
                targetNotifyPropertyChanged.PropertyChanged -= Target_OnPropertyChanged;
            }
        }
    }
}

/// <summary>
/// 绑定初始化时的值传递方向
/// </summary>
public enum BindingInitMode
{
    /// <summary>
    /// 使用被绑定对象的值设置当前对象的值（默认）
    /// </summary>
    SourceToTarget = 0,

    /// <summary>
    /// 啥都不干
    /// </summary>
    None = 1,

    /// <summary>
    /// 使用当前对象的值设置被绑定对象的值
    /// </summary>
    TargetToSource = 2,
}

public class ClrBindingPropertyContext
{
    public ClrBindingPropertyContext(object bindableObject, string path,
        PropertyGetter? propertyGetter = null,
        PropertySetter? propertySetter = null)
    {
        BindableObjectWeakReference = new WeakReference<object>(bindableObject);
        Path = path;
        PropertyGetter = propertyGetter;
        PropertySetter = propertySetter;
    }

    public WeakReference<object> BindableObjectWeakReference { get; }
    public string Path { get; }
    public PropertyGetter? PropertyGetter { get; }
    public PropertySetter? PropertySetter { get; }

    internal PropertyGetter? InternalPropertyGetter
    {
        get
        {
            if (_internalPropertyGetter is not null)
            {
                return _internalPropertyGetter;
            }

            _internalPropertyGetter ??= PropertyGetter;

            if (_internalPropertyGetter is null && BindableObjectWeakReference.TryGetTarget(out var bindableObject))
            {
                var propertyInfo = bindableObject.GetType().GetProperty(Path);
                var getMethod = propertyInfo?.GetGetMethod();
                if (getMethod != null)
                {
                    _internalPropertyGetter = o => getMethod.Invoke(o, null);
                }
            }

            return _internalPropertyGetter;
        }
    }

    private PropertyGetter? _internalPropertyGetter;

    internal PropertySetter? InternalPropertySetter
    {
        get
        {
            if (_internalPropertySetter is not null)
            {
                return _internalPropertySetter;
            }

            _internalPropertySetter ??= PropertySetter;

            if (_internalPropertySetter is null && BindableObjectWeakReference.TryGetTarget(out var bindableObject))
            {
                var propertyInfo = bindableObject.GetType().GetProperty(Path);
                var setMethod = propertyInfo?.GetSetMethod();
                if (setMethod != null)
                {
                    _internalPropertySetter = (o, value) => setMethod.Invoke(o, new object?[] { value });
                }
            }

            return _internalPropertySetter;
        }
    }
    private PropertySetter? _internalPropertySetter;
}

public delegate object? PropertyGetter(object bindableObject);

public delegate void PropertySetter(object bindableObject, object? propertyValue);
```

单元测试代码如下

```csharp
[TestClass]
public class ClrBidirectionalBindingTest
{
    [ContractTestCase]
    public void GCTest()
    {
        "设置 A 和 B 的单向绑定，当 A 和 B 被回收之后，设置的绑定也会被回收".Test(() =>
        {
            // 设置 A 和 B 的单向绑定
            var weakReference = SetBinding();

            // 当 A 和 B 被回收之后
            GC.Collect();
            GC.WaitForFullGCComplete();
            GC.Collect();

            // 设置的绑定也会被回收
            Assert.AreEqual(false, weakReference.TryGetTarget(out _));

            static WeakReference<ClrBidirectionalBinding> SetBinding()
            {
                var a = new A();
                var b = new B();
                var binding = new ClrBidirectionalBinding
                (
                    source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                    target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                    BindingDirection.OneWay
                );
                return new WeakReference<ClrBidirectionalBinding>(binding);
            }
        });

        "设置 A 和 B 的单向绑定，当 B 被回收后，当 A 触发绑定事件时，设置的绑定也会被回收".Test(() =>
        {
            var a = new A();
            var weakReference = SetBinding(a);

            GC.Collect();
            GC.WaitForFullGCComplete();
            GC.Collect();

            // 当 A 触发绑定事件时
            a.AProperty1 = Guid.NewGuid().ToString();

            // 放在独立的方法，否则局部变量将会引用，从而不会回收
            AssertClrBidirectionalBinding(weakReference);

            // 设置的绑定也会被回收
            GC.Collect();
            GC.WaitForFullGCComplete();
            GC.Collect();

            Assert.AreEqual(false,weakReference.TryGetTarget(out _));

            static void AssertClrBidirectionalBinding(WeakReference<ClrBidirectionalBinding> weakReference)
            {
                if (weakReference.TryGetTarget(out var binding))
                {
                    Assert.AreEqual(false, binding.IsAlive());
                }
            }

            static WeakReference<ClrBidirectionalBinding> SetBinding(A a)
            {
                var b = new B();
                var binding = new ClrBidirectionalBinding
                (
                    source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                    target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                    BindingDirection.OneWay
                );
                return new WeakReference<ClrBidirectionalBinding>(binding);
            }
        });
    }

    [ContractTestCase]
    public void BindingTest()
    {
        "设置 A 和 B 的单向绑定，设置 TargetToSource 初始化，创建绑定完成，即将 B 属性的值赋值给到 A 上".Test(() =>
        {
            // 先给 B 一个初始值，即将 B 属性的值赋值给到 A 上
            var a = new A();

            var b = new B()
            {
                BProperty1 = Guid.NewGuid().ToString(),
            };

            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                BindingDirection.OneWay,
                initMode: BindingInitMode.TargetToSource
            );

            Assert.AreEqual(a.AProperty1, b.BProperty1);
        });

        "设置 A 和 B 的双向绑定，设置 TargetToSource 初始化，创建绑定完成，即将 B 属性的值赋值给到 A 上".Test(() =>
        {
            // 先给 B 一个初始值，即将 B 属性的值赋值给到 A 上
            var a = new A();

            var b = new B()
            {
                BProperty1 = Guid.NewGuid().ToString(),
            };

            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                initMode: BindingInitMode.TargetToSource
            );

            Assert.AreEqual(a.AProperty1, b.BProperty1);
        });

        "设置 A 和 B 的单向绑定，创建绑定完成，即将 A 属性的值赋值给到 B 上".Test(() =>
        {
            // 先给 A 一个初始值，用来测试是否 A 属性的值赋值给到 B 上
            var a = new A()
            {
                AProperty1 = Guid.NewGuid().ToString()
            };

            var b = new B();

            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                BindingDirection.OneWay
            );

            Assert.AreEqual(a.AProperty1, b.BProperty1);
        });

        "设置 A 和 B 的双向绑定，创建绑定完成，即将 A 属性的值赋值给到 B 上".Test(() =>
        {
            // 先给 A 一个初始值，用来测试是否 A 属性的值赋值给到 B 上
            var a = new A()
            {
                AProperty1 = Guid.NewGuid().ToString()
            };

            var b = new B();

            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1))
            );

            Assert.AreEqual(a.AProperty1, b.BProperty1);
        });

        "设置 A 和 B 的单向绑定，如果 B 绑定的属性没有 Set 方法，抛出 ArgumentNullException 异常".Test(() =>
        {
            var a = new A();
            var b = new B();

            Assert.ThrowsException<ArgumentNullException>(() =>
            {
                _ = new ClrBidirectionalBinding
                (
                    source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                    target: new ClrBindingPropertyContext(b, nameof(b.BPropertyWithoutSet)),
                    BindingDirection.OneWay
                );
            });
        });

        "设置 A 和 B 的单向绑定，如果 A 绑定的属性没有 Get 方法，抛出 ArgumentNullException 异常".Test(() =>
        {
            var a = new A();
            var b = new B();

            Assert.ThrowsException<ArgumentNullException>(() =>
            {
                _ = new ClrBidirectionalBinding
                (
                    source: new ClrBindingPropertyContext(a, nameof(a.APropertyWithoutGet)),
                    target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                    BindingDirection.OneWay
                );
            });
        });

        "设置 A 和 C 的双向绑定，如 C 不继承 INotifyPropertyChanged 接口，将抛出 ArgumentException 异常".Test(() =>
        {
            // 设置 A 和 C 的单向绑定
            var a = new A();
            var c = new C();

            Assert.ThrowsException<ArgumentException>(() =>
            {
                _ = new ClrBidirectionalBinding
                (
                    source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                    target: new ClrBindingPropertyContext(c, nameof(c.CProperty1)),
                    BindingDirection.TwoWay
                );
            });
        });

        "设置 C 和 A 的单向绑定，如 C 不继承 INotifyPropertyChanged 接口，将抛出 ArgumentException 异常".Test(() =>
        {
            // 设置 C 和 A 的单向绑定
            var c = new C();
            var a = new A();

            Assert.ThrowsException<ArgumentException>(() =>
            {
                _ = new ClrBidirectionalBinding
                (
                    source: new ClrBindingPropertyContext(c, nameof(c.CProperty1)),
                    target: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                    BindingDirection.OneWay
                );
            });
        });

        "设置 A 和 C 的单向绑定，可以让 C 不继承 INotifyPropertyChanged 接口".Test(() =>
        {
            // 设置 A 和 C 的单向绑定
            var a = new A();
            var c = new C();

            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(c, nameof(c.CProperty1)),
                BindingDirection.OneWay
            );

            // 可以让 C 不继承 INotifyPropertyChanged 接口
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, c.CProperty1);
        });

        "设置 A 和 B 的两个属性绑定，两个属性之间的更新互不影响".Test(() =>
        {
            // 设置 A 和 B 两个属性绑定
            var a = new A();
            var b = new B();
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1))
            );
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty2)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty2))
            );

            // 两个属性之间的更新互不影响
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);

            a.AProperty2 = 10;
            Assert.AreEqual(a.AProperty2, b.BProperty2);

            value = Guid.NewGuid().ToString();
            b.BProperty1 = value;
            Assert.AreEqual(value, a.AProperty1);

            b.BProperty2 = 100;
            Assert.AreEqual(b.BProperty2, a.AProperty2);
        });

        "设置 A 和 B 属性绑定，更改非绑定属性，不会影响原有的属性".Test(() =>
        {
            // 设置 A 和 B 属性绑定
            var a = new A();
            var b = new B();
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1))
            );

            // 更改非绑定属性，不会影响原有的属性
            a.AProperty2 = 10;

            Assert.AreEqual(string.Empty, b.BProperty1);
            Assert.AreEqual(0, b.BProperty2);

            // 更新属性之后，依然也是不会影响
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;
            Assert.AreEqual(value, b.BProperty1);

            a.AProperty2 = 20;
            Assert.AreEqual(value, b.BProperty1);
            Assert.AreEqual(0, b.BProperty2);
        });

        "给 A 和 B 两个对象设置双向绑定，无论哪个对象更改，都能更新".Test(() =>
        {
            // 给 A 和 B 两个对象设置双向绑定
            var a = new A();
            var b = new B();
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                BindingDirection.TwoWay
            );

            // 无论哪个对象更改，都能更新
            // 先设置 A 的属性，再测试设置 B 的属性
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);

            // 多次设置
            value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);

            value = Guid.NewGuid().ToString();
            b.BProperty1 = value;
            Assert.AreEqual(value, a.AProperty1);

            // 多次设置
            value = Guid.NewGuid().ToString();
            b.BProperty1 = value;
            Assert.AreEqual(value, a.AProperty1);

            // 在 B 设置完成之后，再次设置 A 的属性
            value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);
        });

        "给 A 和 B 两个对象设置绑定，没有给定默认的赋值和获取值委托，可以自动生成".Test(() =>
        {
            // 给 A 和 B 两个对象设置绑定，没有给定默认的赋值和获取值委托
            var a = new A();
            var b = new B();

            // 给 A 和 B 两个对象设置单向绑定
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1)),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1)),
                BindingDirection.OneWay
            );

            // 可以自动生成，等于绑定生效
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);
        });

        "给 A 和 B 两个对象设置单向绑定，可以在 A 更新时，给 B 赋值；在 B 更新时，啥都不做".Test(() =>
        {
            var a = new A();
            var b = new B();

            // 给 A 和 B 两个对象设置单向绑定
            _ = new ClrBidirectionalBinding
            (
                source: new ClrBindingPropertyContext(a, nameof(a.AProperty1),
                    propertyGetter: bindableObject => ((A) bindableObject).AProperty1),
                target: new ClrBindingPropertyContext(b, nameof(b.BProperty1),
                    propertySetter: (bindableObject, value) => ((B) bindableObject).BProperty1 = (string) value!),
                BindingDirection.OneWay
            );

            // 可以在 A 更新时，给 B 赋值
            var value = Guid.NewGuid().ToString();
            a.AProperty1 = value;

            Assert.AreEqual(value, b.BProperty1);

            // 在 B 更新时，啥都不做
            b.BProperty1 = Guid.NewGuid().ToString();
            Assert.AreEqual(value, a.AProperty1);
        });
    }
}
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f7946836b58718c65273084554a854b673503a84/BefawafereKehufallkee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f7946836b58718c65273084554a854b673503a84/BefawafereKehufallkee) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f7946836b58718c65273084554a854b673503a84
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f7946836b58718c65273084554a854b673503a84
```

获取代码之后，进入 BefawafereKehufallkee 文件夹

更多 dotnet 的博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
