
# dotnet 设计规范 · 抽象类


<!--more-->



<!-- 标签：设计规范，规范 -->

X 不要定义 public 或 protected internal 访问的构造函数。默认 C# 语言不提供抽象类的公开构造函数方法。

如果一个构造函数定义为公开，只有在开发者需要创建这个类的实例的时候才可以这样做。因为开发者不应该创建一个抽象类的实例，所以如果一个抽象类可以创建实例就是一个不好的实现，容易让开发者做出不恰当代码。

✓ 定义抽象类的构造函数为 protected 或 internal 访问

定义抽象类的构造函数为 protected 是比较推荐的方法，因为定义为 protected 让这个类只能在基类继承，创建可实例化的基类的实例。

定义抽象类的构造函数为 internal 可以限制开发者只能在只是程序集内使用这个类，当然即使定义为 internal 还是无法创建抽象类的实例。

✓ 提供至少一个可实例化类继承抽象类在自己的库里。

做这个是为了方便验证抽象类的设计。如 System.IO.FileStream 继承抽象类 System.IO.Stream 而且 FileStream 可以创建实例。

当然这个只是建议，如果在开发一个 dotnet standard 项目，需要注入一个和具体框架有关的类，于是先设计一个抽象类放在库。在具体的 dotnet framework 等框架创建这个抽象类的基类，用于做和具体平台相关代码。那么不在库放一个实现抽象类的类也是可以的。但是这时建议在测试项目继承这个抽象类，尝试测试类的设计。

参见：[docs/abstract-class.md at master · dotnet/docs](https://github.com/dotnet/docs/blob/master/docs/standard/design-guidelines/abstract-class.md )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。