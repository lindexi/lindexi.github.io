# ReSharper 已知问题 在 Directory.Build.props 属性求值结果不能应用到项目里

本文记录一个 ReSharper 已知问题，在在 Directory.Build.props 属性求值结果不能应用到项目里，此行为将导致 ReSharper 获取的构建文件与 VisualStudio 不同，导致了可能存在的冲突

此问题我已经报告给 ReSharper 官方，详细请看： [https://youtrack.jetbrains.com/issue/RSRP-500675](https://youtrack.jetbrains.com/issue/RSRP-500675)

以下是我的报告内容：

ReSharper Reports 'Ambiguous reference' Error Incorrectly in Project with Cross - Project References

If a project simultaneously marks and references the code of another project or conducts project references, an "Ambiguous reference" error may appear in ReSharper. However, the project can actually be built successfully in Visual Studio.

The project structure to reproduce this issue is a bit complex, but don't worry. I've built it. I've placed the minimal reproducible code on GitHub, and you can find all the code via this link https://github.com/lindexi/lindexi\_gd/tree/40067cf83fece460a7d80c3d2445e6ab652531b3/Workbench/QeedekallheweWhanenurnellere .\
Here are the steps to reproduce:

1. Create three new projects: Lib, AllInOne, and App projects.
2. Put the Directory.Build.props file in the folder where the sln file is located.
3. Establish the association between projects: The AllInOne project depends on the Lib project, and the App project depends on the AllInOne project.

After creating the projects, fill in the necessary content.

Add the following custom attribute content to the Directory.Build.props file:

```xml
  <PropertyGroup>
    <PackAllInOne>true</PackAllInOne>
    <PackAllInOne Condition="'$(Configuration)'=='debug'">false</PackAllInOne>
  </PropertyGroup>
```

Add an empty Foo type to the Lib project with no code inside, as only the type is needed to reproduce this issue.

```csharp
namespace Lib;

public class Foo
{
}
```

Write the Program.cs file in the App project to create a Foo object. The code is as follows:\
csharp

```csharp
using Lib;

var foo = new Foo();
```

Up to this point, everything works fine. This is because the App project indirectly depends on the Lib project through the AllInOne project and can access the Foo type.

The final step is to edit the content of the AllInOne's csproj file. Configure the AllInOne project to decide whether to reference the Lib project or the code inside the Lib project through the PackAllInOne property. The code is as follows:

```xml
  <ItemGroup Condition=" $(PackAllInOne) == true ">
    <Compile Include="..\Lib\**\*.cs" />
  </ItemGroup>

  <ItemGroup Condition=" $(PackAllInOne) != true ">
    <ProjectReference Include="..\Lib\Lib.csproj" />
  </ItemGroup>
```

After completing this step, you'll see that the `Program.cs` file, which originally worked fine in the App project, now shows an "Ambiguous reference" error in ReSharper. Even though the project can still be built successfully in Visual Studio, it means ReSharper's error message is an incorrect false alarm.

You can find all my code in https://github.com/lindexi/lindexi\_gd/tree/40067cf83fece460a7d80c3d2445e6ab652531b3/Workbench/QeedekallheweWhanenurnellere
