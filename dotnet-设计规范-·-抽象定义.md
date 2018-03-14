
# dotnet 设计规范 · 抽象定义


<!--more-->



严格来说，只有一个类被其他的类继承，那么这个类就是基类。在很多时候，基类的定义是提供足够的抽象和通用方法和属性、默认实现。在继承关系中，基类定义在上层抽象和底层自定义之间。

他们充当抽线实现的实现帮助者，充当框架的部分。例如，处于框架的列表接口 IList 定义。定义一个 IList 在框架中具有重要的作用，可以抽象出具有数组列表的类型，有很多类都继承这个接口，如`System.Collections.ObjectModel.Collection`、`:System.Collections.ObjectModel.KeyedCollection` 但是这些类都定义了属于自己的存储方法。

基类有时候不适合充当自己的抽象，因为他们存在太多需要定义的内容。例如`Collection<T>`基类包含了很多具体实现，很多都是在 IList 之外的方法，因为集成的方式由于开放的方式。事实上，他是一个把数据字段里，用字段来存储内容的数组，其他的方法都是对存储字段的封装。

从上面的讨论可以知道，基类对于定义抽象很有帮助，但是在定义的时候，基类需要有自己的职责，因为基类添加了继承深度所以对框架的复杂度会增加。所以定义基类必须基类具有意义。需要避免为了定义相同的类型定义基类，基类的定义需要执行特殊的方法，基类定义需要很清楚。如果提供很多基类，需要让开发者容易找到使用的基类而不是对继承哪个基类需要经过想的时间很长。

✓ 建议设置基类抽象，即使他没有任何抽象的方法或属性。这个定义是任何需要使用这个类定义都需要继承，但是另一个方法是设置这个类的构造是私有。

✓ 把基类和继承类的命名空间分开，这样基类有更大的扩展。

X 建议不要把公开的基类使用 Base 做后缀，如果一个类需要使用这个命名做后缀无法让他有意义的命名，那么这个基类可能是违反了上面的原则。

参见：[docs/base-classes-for-implementing-abstractions.md at master · dotnet/docs](https://github.com/dotnet/docs/blob/master/docs/standard/design-guidelines/base-classes-for-implementing-abstractions.md )



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。