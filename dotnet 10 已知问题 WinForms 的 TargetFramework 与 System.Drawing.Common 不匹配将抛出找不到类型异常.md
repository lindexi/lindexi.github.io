# dotnet 10 已知问题 WinForms 的 TargetFramework 与 System.Drawing.Common 不匹配将抛出找不到类型异常

本文记录 dotnet 10 新引入的问题。如果 TargetFramework 是 .NET 9 版本，而引用的 System.Drawing.Common 包是 10.0 版本，那么运行程序时，可能抛出找不到类型异常

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

此问题我已经在 WinForms 仓库反馈： <https://github.com/dotnet/winforms/issues/14145>

最简复现步骤如下：

先创建一个空的 .NET 项目，编辑 csproj 文件，替换为以下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <UseWindowsForms>true</UseWindowsForms>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="System.Drawing.Common" Version="10.0.1" />
  </ItemGroup>
</Project>
```

创建 Program.cs 文件，添加如下代码

```csharp
System.Windows.Forms.SendKeys.SendWait("Hello, World!");
```

将项目里面的其余文件全删掉。然后构建运行项目

此时可见抛出如下错误

```
System.TypeInitializationException: The type initializer for 'System.Windows.Forms.SendKeys' threw an exception.
   ---> System.TypeInitializationException: The type initializer for 'System.Windows.Forms.ScaleHelper' threw an exception.
   ---> System.TypeLoadException: Could not load type 'System.Private.Windows.Core.OsVersion' from assembly 'System.Private.Windows.Core, Version=10.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'.
     at System.Windows.Forms.ScaleHelper.<InitializeStatics>g__GetPerMonitorAware|8_1()
     at System.Windows.Forms.ScaleHelper.InitializeStatics()
     --- End of inner exception stack trace ---
     at System.Windows.Forms.ScaleHelper.get_InitialSystemDpi()
     at System.Windows.Forms.Control..ctor(Boolean autoInstallSyncContext)
     at System.Windows.Forms.SendKeys.SKWindow..ctor()
     at System.Windows.Forms.SendKeys..cctor()
     --- End of inner exception stack trace ---
     at System.Windows.Forms.SendKeys.SendWait(String keys)
     at sendkey_error.MainWindow.Button_Click(Object sender, RoutedEventArgs e)
```

问题的原因是在 <https://github.com/dotnet/winforms/pull/12839> 里面修改了命名空间

这就导致了 OsVersion 类型从 `System.Private.Windows.Core` 命名空间改到 `System.Private.Windows` 命名空间，这里的不匹配就导致了类型找不到异常

本次问题属于情有可原，毕竟更新的是主版本号，主版本更新发生不兼容是合理的

此问题一开始是在 WPF 仓库报告的，但经过我调查，和 WPF 毫无关系，详细请看： <https://github.com/dotnet/wpf/issues/11313>

解决方法十分简单，保持 System.Drawing.Common 跟随主入口项目的 TargetFramework 版本，如修改 csproj 为如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWindowsForms>true</UseWindowsForms>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="System.Drawing.Common" Version="9.0.11" />
  </ItemGroup>
</Project>
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1534db9e5f807d2c958cc8f7509f9f45229651e5/WPFDemo/LaiwerelawkewaFeereajemle) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1534db9e5f807d2c958cc8f7509f9f45229651e5/WPFDemo/LaiwerelawkewaFeereajemle) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1534db9e5f807d2c958cc8f7509f9f45229651e5
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1534db9e5f807d2c958cc8f7509f9f45229651e5
```

获取代码之后，进入 WPFDemo/LaiwerelawkewaFeereajemle 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )