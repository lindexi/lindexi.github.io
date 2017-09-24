# win10 uwp 反射

本文在[h神](http://www.cnblogs.com/h82258652)的指导下完成。

反射是强大的好用的，我们可以添加新功能不修改之前的代码，通过使用反射得到。

本文下面和大家说如何做一个和WPF一样的反射功能，如何才能获的 UWP 程序集所有类。

<!--more-->

<div id="toc"></div>

先来说下反射。

.Net  最小单位是装配件，什么是装配件？其实就是 dll  或 exe 。.Net 程序包括 程序集 ，模块 ， 类型 这几个。反射可以在程序运行得到这几个组成部分的相关信息。

反射可以获得`Assembly`，他可以获得正在运行的装配件信息，也可以动态的加载装配件，以及在装配件中查找类型信息，并创建该类型的实例。可以获得`Type`，他可以获得对象的类型信息，包括属性方法，可以调用属性方法。可以获得`MethodInfo`，他可以得到类方法的参数、返回值，可以调用方法。和`MethodInfo`差不多的，还有很多，都在`System.Reflection`可以看到。

反射是做什么？反射其实应用在对于类型差不多，但是需要对修改时不需要修改多处的代码使用。

 - 可以使用反射动态地创建类型的实例，将类型绑定到现有对象，或从现有对象中获取类型

 - 应用程序需要在运行时从某个特定的程序集中载入一个特定的类型，以便实现某个任务时可以用到反射

 - 反射主要应用与类库，这些类库需要知道一个类型的定义，以便提供更多的功能。

在我写的[MVVM](./win10-uwp-MVVM%E5%85%A5%E9%97%A8/)，就使用反射获得ViewModel，这样添加ViewModel 不需要修改写的代码。

反射可以添加类型不需要修改代码，这是很好的，但是反射性能比较差，在需要使用的时候才使用反射，不要每次都使用。

编译可以知道类型写错，反射不知道，可以得到错误的，一般使用反射需要小心，如果使用一些工具修改，那么反射得到的容易错误，好在C# 6有 `name of` 可以获得一个属性或方法的名称，这样使用他进行反射，得到的值才不容易出错。

反射可以获得安全类型的类，如internal或其他不是public的访问的类或类的字段，都可以获得。



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

## 判断类型继承

经常需要判断 UWP 的类是否继承接口，如果需要判断继承接口，可以使用下面代码

1. 判断类型是否接口


```csharp
    type.GetTypeInfo().IsInterface
```
就可以判断是否接口

接口的判断继承和类不同，接口判断使用下面的方式。这里 type 是一个类型，而 a 就是一个实例。


```csharp
    type.IsAssignableFrom(a.GetType().GetTypeInfo());

```
上面的代码可以写为


```csharp
    type.IsInstanceOfType(a)
```

如果不是接口，可以使用下面代码


```csharp
    a.GetType().GetTypeInfo().IsSubclassOf(type);
```



## WPF 反射获得所有类


```csharp
            Assembly assembly = Assembly.GetExecutingAssembly();
            foreach (var temp in assembly.GetTypes())
            {


            }
```



### 获得类型

1. typeof


```csharp
    Type type = typeof(类型);
```


2. System.Object.GetType

所有的类存在的方法，通过一个实例，可以获得实例的



```csharp
    类型 t = new 类型();

    Type type=t.GetType();
```


1. System.Type.GetType


```csharp
    Type type=Type.GetType("cvte.类型",false,true);
```

注意参数0是类名，参数1表示若找不到对应类时是否抛出异常，参数1表示类名是否区分大小写

### 创建对象

创建对象的方法很多

```csharp
   Assembly assembly = Assembly.Load("Assembly");

   Type type = assembly.GetType("Example");

   object obj =Activator.CreateInstance(type);
```
可以传入参数，参数可以传入多个

```csharp
    object obj = Activator.CreateInstance(type,参数);
```

另一个方法

```csharp
     object obj = type.Assembly.CreateInstance(type.FullName);
```

### 获得方法

获得类型方法


```csharp
    MethodInfo[] listMethodInfo = type.GetMethods();
```

使用方法


```csharp
     object obj =Activator.CreateInstance(type);
     MethodInfo methodInfo = type.GetMethod("方法");
     methodInfo.Invoke(obj,null);  //参数1类型为object[]，代表方法的对应参数，输入值为null代表没有参数
```

### 获得属性


```csharp
   object obj =Activator.CreateInstance(type);

   PropertyInfo propertyInfo = obj.GetProperty("Name");    //获取Name属性对象

   var name = propertyInfo.GetValue(obj,null）;            //获取Name属性的值

   propertyInfo.SetValue(obj,"cvte",null);                //设置Name属性
```


### 获得Attribute


```csharp
    object[] typeAttributes =type.GetCustomAttributes(false);   
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

注意，虽然可以使用`BaseType`判断，但是如果继承多了，那么判断是否存在A，是不对的。

虽然说了很多，但很多都是大神讲的，于是

推荐大神讲的反射：http://www.cnblogs.com/wangshenhe/p/3256657.html

https://www.codeproject.com/Articles/55710/Reflection-in-NET

### 反射判断类继承接口

判断一个类继承接口的方法不可以使用 IsSubclassOf 判断

假如有下面的代码


```csharp
    interface IA
    {
        
    }

    class A : IA
    {

    }
```

这个代码会返回false `a.GetType().IsSubclassOf(typeof(IA));`

那么在C#如何判断一个类继承了接口，和一个类实现了接口？

使用方法是用 接口的IsAssignableFrom


```csharp
                A a = new A();

            Console.WriteLine(typeof(IA).IsAssignableFrom(a.GetType()));
```

这时返回的是 true。

但是 IsAssignableFrom 使用的是类型，如果有一个类实现，可以尝试下面代码


```csharp
    typeof(IA).IsInstanceOfType(a)
```

这个方法也可以获得类是否继承接口。

参见： [在C#中判断某个类是否实现了某个接口 ](https://leonax.net/p/3697/determine-if-a-class-implements-a-certain-interface/)

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201792392836.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 







