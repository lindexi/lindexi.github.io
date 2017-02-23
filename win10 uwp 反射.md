# win10 uwp 反射

本文在[h神](http://www.cnblogs.com/h82258652)的指导下完成。

反射是强大的好用的，我们可以添加新功能不修改之前的代码，通过使用反射得到。

本文下面和大家说如何做一个和WPF一样的反射功能，如何才能获的 UWP 程序集所有类。

<!--more-->
<!-- csdn -->

## uwp 程序集所有类

我们可以使用下面代码获得程序集所有的类
		

```csharp
Application.Current.GetType().GetTypeInfo().Assembly

```

```csharp
            foreach (var temp in applacationAssembly.DefinedTypes)
            {
             
            }

```

那么我们如何获得属于ViewModel的类，如果我们没有继承base，那我们有简单方法。

		

```csharp
    public class ViewModelAssembly:Attribute
    {
    
    }
    [ViewModelAssembly]
    public class DstidModel : ViewModelBase

```
我们可以通过Attribute，查看是否有，如果有，就是ViewModel

		

```csharp
            foreach (var temp in applacationAssembly.DefinedTypes)
            {
                if (temp.CustomAttributes.Any(t => t.AttributeType == typeof(ViewModelAssembly)))
                {

                }
            }

```

当然我们还修改下，因为我们不需要写那么多

		

```csharp
            var applacationAssembly = Application.Current.GetType().GetTypeInfo().Assembly;
            foreach (var temp in applacationAssembly.DefinedTypes
                .Where(temp=>temp.CustomAttributes.Any(t => t.AttributeType == typeof(ViewModelAssembly))))
            {
                
            }

```
那么我们可以使用type得到ViewModel，参见http://lindexi.oschina.io/lindexi/post/win10-uwp-%E4%BB%8EType%E4%BD%BF%E7%94%A8%E6%9E%84%E9%80%A0/ 

## WPF 反射获得所有类


```csharp
            Assembly assembly = Assembly.GetExecutingAssembly();
            foreach (var temp in assembly.GetTypes())
            {


            }
```

### 判断一个类是另一个的子类


```csharp
    class A
    {

    }

    class A1:A
    {

    }

    A1 a=new A1();
    a.GetType().IsSubclassOf(typeof(A)) 如果返回true，那么a就是继承 A
```


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 

我们可以获所有的ViewModel，在添加新的ViewModel，我们不需要做改动






