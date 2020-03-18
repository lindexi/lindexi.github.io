# C# 反射判断属性是抽象继承

我在写一个有趣的 WPF 软件可以通过 dll 反射重新生成代码，我需要了解dll里面每个属性是抽象的还是继承的还是虚属性

<!--more-->
<!-- 发布 -->

在 C# 中可以方便通过反射拿到一个类里面的所有属性，在阅读本文之前，我期望你先看 [win10 uwp 反射](https://blog.lindexi.com/post/win10-uwp-%E5%8F%8D%E5%B0%84.html)

## 抽象

需要知道在 C# 中属性是使用两个方法做的，一个是 Set 方法一个是 Get 方法，其中 Set 方法用来做设置，而 Get 方法用来做获取。但是不一定每个属性都有 Set 和 Get 方法，但是至少有一个。而 C# 中没有提供任何一个判断属性是否抽象的方法，但是提供了方法的判断是否抽象。所以可以尝试获取某个属性的 Set 或 Get 方法是否是抽象方法从而判断这个属性是否抽象属性

假设通过反射拿到了一个属性 `PropertyInfo pi` 那么可以通过下面代码判断 Set 方法是否是抽象的

```csharp
if(pi.GetSetMethod().IsAbstract)
{
}
```

如果发现 Set 方法是抽象的，那么这个属性就是抽象属性

## 虚属性

判断属性是虚属性也是通过属性的 Set 或 Get 方法判断，但是属性不一定存在 Set 或 Get 方法，如上面代码说的，可以通过 CanRead 属性判断能否读取，如果能读取那么可以用 GetMethod 的方法

```csharp
var m = p.CanRead ? p.GetMethod() : p.SetMethod();
if(m.IsVirtual)
{

}
```

## 重写

也就是继承重写基类的方法，简单的判断就是这个属性的 Set 或 Get 方法的定义的类和当前的类不相同

```csharp
var getMethod = property.GetGetMethod();
```

使用 GetBaseDefinition 可以尝试拿到基类的定义，如果能拿到和当前类不同的，那么这个 Get 方法就是继承的，也就是属性是重写的

```csharp
if (getMethod.GetBaseDefinition().DeclaringType != getMethod.DeclaringType)
{

}
```

[c# - How to identify abstract members via reflection - Stack Overflow](https://stackoverflow.com/questions/1025803/how-to-identify-abstract-members-via-reflection )

[c# - How to check if a property is virtual with reflection?](https://stackoverflow.com/questions/12305945/how-to-check-if-a-property-is-virtual-with-reflection )

[c# - How do I determine if a property was overridden? - Stack Overflow](https://stackoverflow.com/questions/4505232/how-do-i-determine-if-a-property-was-overridden )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
