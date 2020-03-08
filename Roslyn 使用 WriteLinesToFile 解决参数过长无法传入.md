# Roslyn 使用 WriteLinesToFile 解决参数过长无法传入

在写 Roslyn 的时候，经常需要辅助编译的工具，而这些工具需要传入一些参数，在项目很大的时候，会发现自己传入的参数比微软限制控制台可以传入的参数大很多，这时就无法传入了参数。

本文告诉大家如何使用 WriteLinesToFile 先把参数写入文件，通过文件的方式传输参数

<!--more-->
<!-- CreateTime:2019/1/29 16:31:43 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

为了让大家可以方便了解 Roslyn 编译过程，所以本文是带着一个任务来做的。通过阅读本文，大家可以学会怎么写出一些神奇的代码。

任务的背景是，在编译的过程，暗中修改一些代码，而且让开发者发现不了。

最简单的方法就是修改编译流程，修改编译流程是相对比较复杂的。但是上面在 MSBuild 可以使用很多奇怪的 Task 来做很多有趣的事情，其中就有一个强大的 Task 是 `Exec ` ，这个命令就是使用命令行调用另一个程序。

也就是在编译的过程可以调用另一个程序，所以就把辅助编译的方法放在另一个程序去做。

但是要让这个辅助的程序跑，还需要告诉这个辅助的程序一下信息，如哪些需要编译的文件。

于是最简单的方式就是写一个 Target 来执行这个辅助的程序。先假设这个辅助的程序就放在 `E:\辅助程序.exe` ，调用的方法请看代码

```csharp
  <Target Name="BitedeZawpiDafuge" BeforeTargets="CoreCompile">
    <Exec Command="E:\辅助程序.exe @(Compile)"></Exec>
  </Target>
```

为了让 Target 运行需要给 Target 一个 Name ，还需要告诉 VisualStudio 什么时候运行这个 Target ，于是就使用`BeforeTargets="CoreCompile"` 告诉 VisualStudio 在开始编译之前就执行

当然，上面的写法只是为了方便第一次接触 Roslyn 的小伙伴，如果看到了这里还是有很多内容不知道，请先看[手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html )。

现在的问题是，调用 Exec 能传入的参数就是被微软限制大小，我的`Compile`编译文件有 10000000000000 个，于是就有很多文件无法传参数进去，这时我就无法在编译之前把代码修改掉，让项目可以成功运行，这样本渣还怎么去坑一个团队。

所以腹黑的本渣就需要一个方式传入很大的参数，找到了一个简单的方法就是使用`WriteLinesToFile`先把参数写到文件，然后把文件路径给辅助程序就可以

为了写入的文件可以在辅助程序找到，就需要先定义文件，请看代码

```csharp
  <ItemGroup>
    <TextFile Include="Items.txt" />
  </ItemGroup>
```

把参数写入文件的方法请看代码

```csharp
  <Target Name="WriteToFile" BeforeTargets="CoreCompile">
    <WriteLinesToFile File="@(TextFile)" Lines="@(Compile)" Overwrite="true" />
  </Target>
```

这时就会把 Compile 的所有文件都写入到`TextFile`文件

在写完之后就可以调用 Exec 执行程序了，也许这时大家会发现，本金鱼又在逗大家了，辅助的程序怎么知道 WriteLinesToFile 执行完了，会不会拿到一个空的文件。

我很负责和大家说，会的，如果现在不修改代码直接调用就会出现读取到的文件可能是空的

还记得 `DependsOnTargets ` 这个属性？通过这个属性可以指定一个 Target 在什么时候运行，在运行辅助程序的代码就需要依赖上面的代码运行。于是修改之后的全部代码请看下面

```csharp
  <Target Name="WriteToFile">
    <WriteLinesToFile File="@(TextFile)" Lines="@(Compile)" Overwrite="true" />
  </Target>

  <Target Name="BitedeZawpiDafuge" BeforeTargets="CoreCompile" DependsOnTargets="WriteToFile">
    <Exec Command="E:\辅助程序.exe $(MSBuildProjectDirectory)\@(TextFile)"></Exec>
  </Target>
```

现在的代码是可以运行了，从参数可以拿到文件名，然后通过读文件的方式拿到输入的文件

现在的代码已经可以运行了，但是本渣还需要继续告诉大家一些属性的设置是为什么，第一个设置的参数就是`WriteLinesToFile` 的 `File` ，使用这个就可以设置输出到哪个文件。第二个参数`Lines`就是输入的文本，不要问我文本怎么写为 `line` 而且是按照`;`分割多行的。

最后一个参数比较重要，是设置如果存在这个文件是否覆盖，如果没有选择覆盖，那么下面的辅助程序拿到的文件都会是以前的文件。我就是没有设置这个属性用了半个钟才发现读的文件都是以前的文件。

在另一个 Target 就是调用辅助程序，需要知道在参数传入的是`$(MSBuildProjectDirectory)`路径的，原因就是刚才写入的文件相对的就是项目所在的文件夹，所以需要从项目所在的文件夹才可以拿到这个文件。

如果是想把文件写在一个临时的文件夹，那么建议使用	`$(IntermediateOutputPath)`文件夹，这个`$(IntermediateOutputPath)`文件夹就是 `obj` 文件夹，建议在这个文件夹里再创建一个文件夹用来放临时的文件。

需要注意，文件同样可以写在`PropertyGroup`里，只是在`PropertyGroup`的引用是使用`$`请看下面代码

```diff
-  <ItemGroup>
-    <TextFile Include="Items.txt" />
-  </ItemGroup>
+  <PropertyGroup>
+    <TextFile>Items.txt</TextFile>
+  </PropertyGroup>
   <Target Name="WriteToFile">
-    <WriteLinesToFile File="@(TextFile)" Lines="@(Compile)" Overwrite="true" />
+    <WriteLinesToFile File="$(TextFile)" Lines="@(Compile)" Overwrite="true" />
   </Target>
```

因为 PropertyGroup 的内容是不存在 Include 特性，所以需要使用上面的方法

如果写入的文件的文件夹是不存在，就需要先创建，如写入的是 `lindexi\foo.txt` 就需要先判断`lindexi`文件夹是否存在，如果没有判断直接使用就会出现下面代码

```csharp
严重性	代码	说明	项目	文件	行	禁止显示状态
错误	MSB3491	未能向文件“obj\Debug\netcoreapp2.0\lindexi\Items.txt”写入命令行。未能找到路径“C:\lindexi\framework\lindexi.Mvvm.framework\obj\Debug\netcoreapp2.0\lindexi\Items.txt”的一部分。	framework	C:\Users\lindexi\.nuget\packages\lindexi.Mvvm.framework\0.1.52-alpha\build\lindexi.Mvvm.framework.targets	11	

``` 

简单创建文件夹的方法是使用 MakeDir 请看下面代码

```xml
   <PropertyGroup>
     <TextDirectory>lindexi/</TextDirectory>
     <TextFile>$(TextDirectory)Items.txt</TextFile>
   </PropertyGroup>
   <Target Name="WriteToFile">
     <MakeDir Condition="!Exists($(TextDirectory))" Directories="$(TextDirectory)"></MakeDir>
     <WriteLinesToFile File="$(TextFile)" Lines="@(Compile)" Overwrite="true" />
   </Target>
```

我在测试的项目写了很长的参数，这个参数只能通过写入到文件的方式传输，不能通过参数的方法传输。测试项目请点击 [Roslyn 使用 WriteLinesToFile 解决参数过长无法传入 1.0-CSDN下载](https://download.csdn.net/download/lindexi_gd/10616166 ) 

<!-- 下载文件 NekasNugouMedapai -->

参见：
[项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://walterlv.gitee.io/post/known-properties-in-csproj.html )

![](https://i.loli.net/2018/08/20/5b7aab757d2f9.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
