# win10 uwp 反射

本文在[h神](http://www.cnblogs.com/h82258652)的指导下完成。

反射是强大的好用的，我们可以添加新功能不修改之前的代码，通过使用反射得到。

但是在WPF的反射我们在UWP如何才能获的程序集所有类，本文下面和大家说如何做。

<!--more-->

## uwp 程序集所有类

我们可以使用
		
```
Application.Current.GetType().GetTypeInfo().Assembly

```

获得程序集所有的类

		
```
            foreach (var temp in applacationAssembly.DefinedTypes)
            {
             
            }

```

那么我们如何获得属于ViewModel的类，如果我们没有继承base，那我们有简单方法。

		
```
    public class ViewModelAssembly:Attribute
    {
    
    }
    [ViewModelAssembly]
    public class DstidModel : ViewModelBase

```
我们可以通过Attribute，查看是否有，如果有，就是ViewModel

		
```
            foreach (var temp in applacationAssembly.DefinedTypes)
            {
                if (temp.CustomAttributes.Any(t => t.AttributeType == typeof(ViewModelAssembly)))
                {

                }
            }

```

当然我们还修改下，因为我们不需要写那么多

		
```
            var applacationAssembly = Application.Current.GetType().GetTypeInfo().Assembly;
            foreach (var temp in applacationAssembly.DefinedTypes
                .Where(temp=>temp.CustomAttributes.Any(t => t.AttributeType == typeof(ViewModelAssembly))))
            {
                
            }

```
那么我们可以使用type得到ViewModel，参见http://lindexi.oschina.io/lindexi/post/win10-uwp-%E4%BB%8EType%E4%BD%BF%E7%94%A8%E6%9E%84%E9%80%A0/ 

我们可以获所有的ViewModel，在添加新的ViewModel，我们不需要做改动




