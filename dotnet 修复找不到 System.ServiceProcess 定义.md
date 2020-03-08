# dotnet 修复找不到 System.ServiceProcess 定义

本文告诉大家如果复制网上一段代码发现 System.ServiceProcess 提示找不到方法或定义，需要手动添加引用

<!--more-->
<!-- CreateTime:2019/10/18 21:24:04 -->


例如下面一段代码

```csharp
using System.ServiceProcess;

        private static bool IsWindowsManagementInstrumentationAvailable
        {
            get
            {
                try
                {
                    using (var serviceController = new ServiceController("Winmgmt"))
                    {
                        return serviceController.Status == ServiceControllerStatus.Running;
                    }
                }
                catch (Exception)
                {
                    return false;
                }
            }
        }
```

在编译的时候提示

```
The type or namespace name 'ServiceProcess' does not exist in the namespace 'System' (are you missing an assembly reference?)


错误	CS0246	未能找到类型或命名空间名“ServiceController”(是否缺少 using 指令或程序集引用?)
```

修复方法是右击依赖项，点击添加引用，在程序集找到 System.ServiceProcess 点击引用就可以

如果是 SDK 的 csproj 可以直接在项目文件添加下面代码

```csharp
    <ItemGroup>
      <Reference Include="System.ServiceProcess" />
    </ItemGroup>
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
