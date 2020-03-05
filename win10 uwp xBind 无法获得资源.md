# win10 uwp x:Bind 无法获得资源

本文告诉大家，如果在 使用 x:Bind 转换器写在资源，而运行出现找不到资源的错误，如果解决。

在运行的时候，出现`System.Runtime.InteropServices.COMException Cannot find a resource with the given key`

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->

<!-- csdn -->

这个问题就是资源寻找路径问题，因为 Binding 是性能比较差，他的资源是从他的自己，如果找不到，就到他的上一级，直到找到或没有。但是 `x:bind` 的资源寻找和 Bind 不同，他是在元素的最顶级元素和应用资源寻找。假如有一个用户控件 Foo ，那么打开他的代码，可以看到这样的代码

```csharp
public global::Windows.UI.Xaml.Data.IValueConverter LookupConverter(string key)
{
    if (this.localResources == null)
    {
        global::Windows.UI.Xaml.FrameworkElement rootElement;
        this.converterLookupRoot.TryGetTarget(out rootElement);
        this.localResources = rootElement.Resources;
        this.converterLookupRoot = null;
    }
    return (global::Windows.UI.Xaml.Data.IValueConverter) (this.localResources.ContainsKey(key) ? this.localResources[key] : global::Windows.UI.Xaml.Application.Current.Resources[key]);
}
```

这就是说，元素资源从根元素找。页面的根元素就是页面本身，用户控件就是他自己本身，可以打开一个 xaml 页面，看到的第一个标签就是根元素。如果无法找到资源，会在应用资源寻找，如果找不到，就报错 System.Runtime.InteropServices.COMException 。应用资源是写在 App.xaml 的资源，所以如果希望使用`x:bind`可以获得资源，或者把资源写在根元素，或者写在应用。

假如有元素 `Slider` 他需要资源转换器，那么转换器需要在哪定义，请看下面的代码

```csharp
                 <Slider Margin="10,10,10,10" Value="{x:Bind xx,Mode=TwoWay,Converter={StaticResource Convert}}" >
                        <Slider.Resources>
                            <local:DoubleConvert x:Name="Convert"></local:DoubleConvert>
                        </Slider.Resources>
                    </Slider>
```

这样写运行会错误，说未指定，因为资源找不到，因为资源寻找不是从元素开始寻找，他是从最顶级元素开始，所以如果让上面的代码可以运行，需要把资源定义在顶级元素。上面的代码可以做修改，让他可以运行

```csharp
    <UserControl.Resources>
        <local:DoubleConvert x:Name="Convert"></local:DoubleConvert>
    </UserControl.Resources>
        <Slider Margin="10,10,10,10" Value="{x:Bind xx,Mode=TwoWay,Converter={StaticResource Convert}}" >
                     
         </Slider>

```

或者把资源写在 app.xaml 也是可以，但是写在这里的资源不会回收，会一直在内存。如果在这里写很多资源，启动速度会很慢。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017915191724.jpg)


[win10 uwp 后台获取资源](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E5%90%8E%E5%8F%B0%E8%8E%B7%E5%8F%96%E8%B5%84%E6%BA%90/)

参见：https://stackoverflow.com/a/39735867/6116637