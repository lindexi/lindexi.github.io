# 读书笔记 dotnet 的字符串在内存是如何存放

本文是读伟民哥翻译的 .NET内存管理宝典 这本书的笔记，我认为读书的过程也需要实践，这样对一知半解的知识也有较为清晰的了解。在阅读到 string 在内存的布局时，我看到 RuntimeHelpers 的 OffsetToStringData 数据，据说此属性可以获取到字符串的字符在内存存放的实际地址，本文将来写一个混合 C# 和 C++\CLI 的应用来进行测试

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

本文将完全采用 .NET 6 进行编写，分别创建 .NET 6 的 C# 控制台程序，和 .NET 6 的 C++\CLI 空项目。这里需要稍微说明的是 C++\CLI 是通过 C++ 编写的 .NET 应用程序，基于 .NET 运行时运行的程序

在 C++\CLI 项目里面添加一个叫 Foo 的类，在类里面添加一个方法，用来输出字符串的内容

```csharp
namespace JuyurchelhiLewecujai
{
	public ref class Foo
	{
	public:
		static void Output(System::String^ input);
	};
}
```

以上代码放在 Foo.h 文件里面，接下来实现 Output 方法。期望是在此方法里面获取在 .NET 定义的字符串对象的实际存放字符的内存指针，实现方法如下

```C++
#include "Foo.h"

#include <iostream>
#include "vcclr.h"

void JuyurchelhiLewecujai::Foo::Output(System::String^ input)
{
	const pin_ptr<const wchar_t> p = PtrToStringChars(input);
	wchar_t const* c = p;
	wprintf(L"%s", c);
}
```

通过 VCClr 提供的 PtrToStringChars 方法可以取出 input 字符串里面的实际存放字符的指针，接着采用 `pin_ptr` 定住此对象。为什么需要采用 `pin_ptr` 定住？原因是 .NET 世界随时都会有 GC 将对象的地址变更，因此为了进行安全使用，需要使用 `pin_ptr` 定住此对象，这样在 GC 时就不会修改此对象的内存地址。细节请参阅 [从C++到C++/CLI - feisky - 博客园](https://www.cnblogs.com/feisky/archive/2009/11/22/1607999.html)

另一个细节是咱在 .NET 里面的字符串的编码格式都是 Unicode 也就是 U16 编码方式，需要对应到 `wchar_t` 类型，也需要使用 `wprintf` 输出而不能使用 `printf` 输出，否则将会读取到 `\0` 而只输出第一个字符。当然了，在 C++\CLI 项目里面依然是不推荐使用 iostream 进行输出的

那以上的 PtrToStringChars 是通过什么魔法进行实现的？可以看到此方法的实现如下

```C++
//
// get an interior gc pointer to the first character contained in a System::String object
//
inline __const_Char_ptr PtrToStringChars(__const_String_handle s) {

	_Byte_ptr bp = const_cast<_Byte_ptr>(reinterpret_cast<__const_Byte_ptr>(s));
	if( bp != _NULLPTR ) {
		bp += System::Runtime::CompilerServices::RuntimeHelpers::OffsetToStringData;
	}
	return reinterpret_cast<__const_Char_ptr>(bp);
}
```

核心逻辑就是通过 RuntimeHelpers 的 OffsetToStringData 属性获取相对于字符串类型的地址的实际字符存放地址

尝试在 C# 项目里面调用刚才定义的 Foo 类型的 Output 代码，方法如下

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            JuyurchelhiLewecujai.Foo.Output("Hello");
        }
    }
```

运行控制台项目，可以看到输出了 Hello 文本，这也就是说字符串的内存布局里面，存放字符数组的地方就是在距离字符串对象指针的 RuntimeHelpers.OffsetToStringData 的地方

然而在 .NET 5 和以上版本，标记了 OffsetToStringData 方法过时，官方推荐使用 GetPinnableReference 代替。关于 GetPinnableReference 请参阅 [C#7.3 新增功能 - 张传宁 - 博客园](https://www.cnblogs.com/SavionZhang/p/11201364.html)

更改 C++\CLI 代码如下

```C++
void JuyurchelhiLewecujai::Foo::Output(System::String^ input)
{
	auto pinString = &input->GetPinnableReference();
	wprintf(L"%s", pinString);
}
```

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/414b803c3c4faa93d1075c28c85e5826c611d9cb/CemholerecelQerrairdoway) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/414b803c3c4faa93d1075c28c85e5826c611d9cb/CemholerecelQerrairdoway) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 414b803c3c4faa93d1075c28c85e5826c611d9cb
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 CemholerecelQerrairdoway 文件夹

更多内存相关，我推荐伟明的 《.NET内存管理宝典 - 提高代码质量、性能和可扩展性》 这本书

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
