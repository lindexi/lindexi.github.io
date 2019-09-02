
# dotnet 设计规范 · 结构体定义


<!--more-->



<!-- 标签：设计规范，规范 -->

X 不要给结构体默认构造函数

默认的C#编译器也不让开发者定义有默认构造的结构体

X 不要定义易变的属性

易变的属性指的是在调用属性返回值的时候返回的是新的实例，易变的属性会有很多的问题。

✓ 需要确定数据的状态，定义为 0、false、null 是正确的

防止开发者不从构造函数进行赋值

✓ 建议结构体继承 System.IEquatable 

因为默认的比较使用的是引用比较，而结构体在使用经常会被复制，如果一个复制的结构体和原来的比较，一般开发者会认为返回是相等。所以需要重写判断。而且重写 IEquatable 可以减少装箱和拆箱。

X 不要扩展 System.ValueType ，实际上大多数语言都阻止自定义，因为系统的类型有编译支持

结构体只能用在很小、很轻、而且不易变的属性，并且很少发生装箱的业务。

注：不易变的另一个解释，请看 Point 类，不能对这个类的属性进行更改。

补充：

所有的字段都禁止公开，如果结构体的定义是字段公开，请不要公开这个结构体。

一般结构体的定义都是公开属性，但是一些和 COM 传输的结构体就需要定义为字段

参见：[docs/struct.md at master · dotnet/docs](https://github.com/dotnet/docs/blob/master/docs/standard/design-guidelines/struct.md )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。