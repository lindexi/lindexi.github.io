# dotnet 给 NuGet 包加上 Aliases 别名解决类型冲突

有时某个相同命名空间相同名字的类型被两个不同的 NuGet 包定义了，尽管这是非常少见的事情，咱需要使用到其中的一个 NuGet 包的类型，但默认情况下将会因为类型冲突而构建不通过。本文将告诉大家如何给 NuGet 包加上 Aliases 别名解决类型冲突

<!--more-->
<!-- CreateTime:2023/6/27 8:41:25 -->


<!-- 发布 -->


给 NuGet 包加上 Aliases 别名，且使用别名限定的方式解决类型冲突分为两步。第一步就是修改 NuGet 引用加上别名名称。第二步就是在代码里面使用别名作为完全限定符

第一步里，只需要在原有的 NuGet 包引用上，添加 Aliases 别名属性，给 NuGet 包添加别名命名，如以下代码

```xml
    <PackageReference Include="Lindexi.Doubi" Version="1.0.0">
      <Aliases>Doubi</Aliases>
    </PackageReference>
```

这里的别名命名推荐是一个首字符大写的单词，就和属性的命名方法差不多

添加命名之后，所有放在命名别名的 NuGet 包里面的 Lib 文件夹，即被 NuGet 带过来的程序集都会应用上此别名。大部分情况下，每个 NuGet 包基本只带一个程序集 dll 文件，约等于给此 dll 文件添加别名。当然，在 NuGet 里面也不限制一个 NuGet 包带多个 dll 程序集文件在 Lib 文件夹下，无论带多少个 dll 程序集文件，这些 dll 程序集都会被应用上相同的别名

第二步里面即可使用给 NuGet 包添加别名进行完全限定类型，大概的代码如下

```csharp
extern alias Doubi;  // 这句 extern alias 必须放在文件开始，不能放在 using 之后

using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using System.Windows;
using Doubi::Lindexi.Doubi;

namespace Lindexi.App
{
	public class Fxx
	{
      public void F1()
      {
          var foo = new Doubi::Lindexi.Doubi.Foo();
      }		
	}
}
```

在具体代码使用里面，约等于将 `global::` 限定替换为别名

