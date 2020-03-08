# C# 使用Emit深克隆

有人问，复制一个类所有属性到另一个类有多少方法？这也就是问深克隆有多少个方法，容易想的有三个。直接复制，反射复制，序列化复制。但是性能比较快的有表达式树复制 IL复制两个，本文主要讲最后一个

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


<!-- 标签：C#,dotnet,Emit -->

关于表达式树复制，参见 [Fast Deep Copy by Expression Trees (C#) - CodeProject](https://www.codeproject.com/Articles/1111658/Fast-Deep-Copy-by-Expression-Trees-C-Sharp)

在开始读本文之前，我推荐两个博客 [读懂IL代码就这么简单 (一) - Zery - 博客园](http://www.cnblogs.com/zery/p/3366175.html) [秒懂C#通过Emit动态生成代码 - 匠心十年 - 博客园](http://www.cnblogs.com/gaochundong/archive/2013/06/01/csharp_emit_generate_assembly.html)

需要先知道一点IL的，后面才比较容易说，假设大家知道了 IL 是什么， 知道了简单的 IL 如何写，那么开始进行功能的开发。第一步是命名，因为需要把一个类的所有属性复制到另一个类，需要调用方法，而方法需要名字，所以第一步就是命名。

为了创建方法 `public void Clone<T>(T source, T los)` 我就使用了下面代码

```csharp
            var dynamicMethod = new DynamicMethod("Clone", null, new[] { typeof(T), typeof(T) });

```
创建方法的第一个参数很容易看到，我就不解释了，第二个参数就是方法的返回值，因为返回是 void 所以不用写。第三个参数是函数的参数，只需要使用类型，如果有多个参数就是写数组，如果这里发现有看不懂的，请和我说。


但是定义方法后需要写方法内的代码，这时需要使用 ILGenerator ，使用他的 Emit 方法，这个方法的速度很快，使用的时候需要知道 IL 的，如果不知道，没关系，我接下来会仔细说。

```csharp
            ILGenerator generator = dynamicMethod.GetILGenerator();

```

需要获得类型的所有属性，虽然这里用了反射，但是只是用一次，因为这里用反射获得方法是在写IL代码，写完可以很多次使用，可能第一次的速度不快，但是之后的速度和自己写代码编译的速度是差不多，所以建议使用这个方法。可以自己去使用 dot trace 去查看性能，我自己看到的是性能很好。

拿出所有属性可以读写的代码`foreach (var temp in typeof(T).GetProperties().Where(temp=>temp.CanRead&&temp.CanWrite))`

查看 IL 需要先把第一个参数放在左边，第二个参数放在右边，调用第二个参数的 get 设置第一个参数的set对应的属性看起来的正常代码就是

```csharp
los.foo=source.foo;
```

这里的 foo 就是拿到一个属性，随意写的，写出来的 IL 请看下面。

```csharp
Ldarg_1 //los
Ldarg_0 //s
callvirt     instance string lindexi.Foo::get_Name()
callvirt     instance void lindexi.Foo::set_Name(string)
ret     
```
可以从上面的代码 callvirt 使用一个方法，对应压入参数，所以可以通过反射获得方法，然后调用这个方法，于是写成代码请看下面

```csharp
                generator.Emit(OpCodes.Ldarg_1);// los
                generator.Emit(OpCodes.Ldarg_0);// s
                generator.Emit(OpCodes.Callvirt,temp.GetMethod);
                generator.Emit(OpCodes.Callvirt, temp.SetMethod);
```

因为可以把这个拿出转化方法，于是所以的下面给所有代码

```csharp
        private static void CloneObjectWithIL<T>(T source, T los)
        {
            var dynamicMethod = new DynamicMethod("Clone", null, new[] { typeof(T), typeof(T) });
            ILGenerator generator = dynamicMethod.GetILGenerator();

            foreach (var temp in typeof(T).GetProperties().Where(temp=>temp.CanRead&&temp.CanWrite))
            {
                generator.Emit(OpCodes.Ldarg_1);// los
                generator.Emit(OpCodes.Ldarg_0);// s
                generator.Emit(OpCodes.Callvirt,temp.GetMethod);
                generator.Emit(OpCodes.Callvirt, temp.SetMethod);
            }
            generator.Emit(OpCodes.Ret);
            var clone = (Action<T, T>) dynamicMethod.CreateDelegate(typeof(Action<T, T>));
            clone(source, los);
        }
```

如果测试了这个方法，那么会发现，这个方法对于这个方法不可以见的类就会出现`MethodAccessException`，所以传入的类需要这个方法可以直接用。

```csharp
//A.dll
public class Foo
{

}

CloneObjectWithIL(foo1,foo2);

//B.dll
        private static void CloneObjectWithIL<T>(T source, T los)

这时无法使用
```

之外，对于静态属性，使用上面代码也是会出错，因为静态的属性的访问没有权限，所以请看修改后的。

```csharp
    /// <summary>
    /// 提供快速的对象深复制
    /// </summary>
    public static class Clone
    {
        /// <summary>
        /// 提供使用 IL 的方式快速对象深复制
        /// 要求本方法具有T可访问
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="source">源</param>
        /// <param name="los">从源复制属性</param>
        /// <exception cref="MethodAccessException">如果输入的T没有本方法可以访问，那么就会出现这个异常</exception>
        // ReSharper disable once InconsistentNaming
        public static void CloneObjectWithIL<T>(T source, T los)
        {
            //参见 http://lindexi.oschina.io/lindexi/post/C-%E4%BD%BF%E7%94%A8Emit%E6%B7%B1%E5%85%8B%E9%9A%86/
            if (CachedIl.ContainsKey(typeof(T)))
            {
                ((Action<T, T>) CachedIl[typeof(T)])(source, los);
                return;
            }
            var dynamicMethod = new DynamicMethod("Clone", null, new[] { typeof(T), typeof(T) });
            ILGenerator generator = dynamicMethod.GetILGenerator();

            foreach (var temp in typeof(T).GetProperties().Where(temp => temp.CanRead && temp.CanWrite))
            {
                //不复制静态类属性
                if (temp.GetAccessors(true)[0].IsStatic)
                {
                    continue;
                }
                
                generator.Emit(OpCodes.Ldarg_1);// los
                generator.Emit(OpCodes.Ldarg_0);// s
                generator.Emit(OpCodes.Callvirt, temp.GetMethod);
                generator.Emit(OpCodes.Callvirt, temp.SetMethod);
            }
            generator.Emit(OpCodes.Ret);
            var clone = (Action<T, T>) dynamicMethod.CreateDelegate(typeof(Action<T, T>));
            CachedIl[typeof(T)] = clone;
            clone(source, los);
        }

        private static Dictionary<Type, Delegate> CachedIl { set; get; } = new Dictionary<Type, Delegate>();
    }

```

需要注意，这里的复制只是复制类的属性，对类的属性内是没有进行复制。如果存在类型 TestA1 ，请看下面代码。

```csharp
        public class TestA1
        {
            public string Name { get; set; }
        }
```

那么在执行下面的代码之后，得到的 TestA1 是相同的。

```csharp
        public class Foo
        {
            public string Name { get; set; }
         
            public TestA1 TestA1 { get; set; }
        }

             var foo = new Foo()
            {
                Name = "123",
                TestA1 = new TestA1()
                {
                    Name = "123"
                }
            };

            var foo1 = new Foo();



            Clone.CloneObjectWithIL(foo, foo1);
            foo1.TestA1.Name == foo.TestA1.Name

            foo.Name = "";
            foo.TestA1.Name = "lindexi";

            foo1.TestA1.Name == foo.TestA1.Name

```

那么上面的代码在什么时候可以使用？实际如果在一个创建的类需要复制基类的属性，那么使用这个方法是很好，例如在 Model 会创建一些类，而在 ViewModel 有时候需要让这些类添加一些属性，如 `Checked` ，那么需要重新复制 Model 的属性，如果一个个需要自己写属性复制，那么开发速度太慢。所以这时候可以使用这个方法。

例如基类是 `Base` ，继承类是`Derived `，请看下面代码

```csharp
public class Base
{
    public string BaseField;
}

public class Derived : Base
{
    public string DerivedField;
}

Base base = new Base();
//some alother code
Derived derived = new Derived();
CloneObjectWithIL(base, derived);
```

如果需要复制一个类到一个新类，可以使用这个代码

```csharp
    private static T CloneObjectWithIL<T>(T myObject)
    {
        Delegate myExec = null;
        if (!_cachedIL.TryGetValue(typeof(T), out myExec))
        {
            // Create ILGenerator
            DynamicMethod dymMethod = new DynamicMethod("DoClone", typeof(T), new Type[] { typeof(T) }, true);
            ConstructorInfo cInfo = myObject.GetType().GetConstructor(new Type[] { });

            ILGenerator generator = dymMethod.GetILGenerator();

            LocalBuilder lbf = generator.DeclareLocal(typeof(T));
            //lbf.SetLocalSymInfo("_temp");

            generator.Emit(OpCodes.Newobj, cInfo);
            generator.Emit(OpCodes.Stloc_0);
            foreach (FieldInfo field in myObject.GetType().GetFields(System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.Public | System.Reflection.BindingFlags.NonPublic))
            {
                // Load the new object on the eval stack... (currently 1 item on eval stack)
                generator.Emit(OpCodes.Ldloc_0);
                // Load initial object (parameter)          (currently 2 items on eval stack)
                generator.Emit(OpCodes.Ldarg_0);
                // Replace value by field value             (still currently 2 items on eval stack)
                generator.Emit(OpCodes.Ldfld, field);
                // Store the value of the top on the eval stack into the object underneath that value on the value stack.
                //  (0 items on eval stack)
                generator.Emit(OpCodes.Stfld, field);
            }
            
            // Load new constructed obj on eval stack -> 1 item on stack
            generator.Emit(OpCodes.Ldloc_0);
            // Return constructed object.   --> 0 items on stack
            generator.Emit(OpCodes.Ret);

            myExec = dymMethod.CreateDelegate(typeof(Func<T, T>));
            _cachedIL.Add(typeof(T), myExec);
        }
        return ((Func<T, T>)myExec)(myObject);
    }
```

http://www.c-sharpcorner.com/uploadfile/puranindia/reflection-and-reflection-emit-in-C-Sharp/

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20178885325.jpg)

https://stackoverflow.com/a/46580446/6116637

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 