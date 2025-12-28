# dotnet 9 已知问题 默认开启 CET 导致进程崩溃

本文记录 dotnet 9 的一个已知且当前已修问题。默认开启 CET 导致一些模块执行时触发崩溃

<!--more-->
<!-- CreateTime:2025/02/06 07:13:36 -->

<!-- 发布 -->
<!-- 博客 -->

什么是 CET ，有什么用？ Control-flow Enforcement Technology (CET) Shadow Stack 是一项提供安全性的功能。细节请参阅 [CET Shadow Stack compatible - Microsoft Learn](https://learn.microsoft.com/en-us/cpp/build/reference/cetcompat?view=msvc-170 ) 和 [A Technical Look at Intel’s Control-flow Enforcement Technology](https://www.intel.com/content/www/us/en/developer/articles/technical/technical-look-control-flow-enforcement-technology.html )

官方文档： [Breaking change: CET supported by default - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/compatibility/interop/9.0/cet-support )

表现：

调用 OpenFileDialog 的 ShowDialog 将会异常崩溃，崩溃异常是 `FAST_FAIL_SET_CONTEXT_DENIED`

或退出错误码为 0xc0000409 STATUS_STACK_BUFFER_OVERRUN

修复的代码：

此问题已在 <https://github.com/dotnet/runtime/pull/109074> 修复

永久解决方法：

更新 dotnet sdk 版本，再次构建发布

理论分析更新 dotnet runtime 也有用，但我没有尝试过，欢迎大佬补充

临时解决方法：

如果不便更新 dotnet sdk 版本，可通过如下设置禁用 CET 开启

```xml
<CETCompat>false</CETCompat>
```

以上设置放在 csproj 项目文件的 PropertyGroup 里面，更改之后的 csproj 代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <CETCompat>false</CETCompat>
  </PropertyGroup>

</Project>
```

相关问题：

- <https://github.com/dotnet/wpf/issues/10305>
- <https://github.com/dotnet/wpf/issues/10318>
- <https://github.com/dotnet/docs/issues/42600>

---

在一台 Win10 22H2 的设备上，由于 CET 的开启导致运行过程中出现 0xC0000602 Fail Fast 异常，直接终止进程

其 DUMP 调用堆栈如下：

```
  KERNELBASE.dll!RaiseFailFastException()
> hostpolicy.dll!coreclr_t::create(const std::wstring & libcoreclr_path, const char * exe_path=0x00000296ebe499a0, const char * app_domain_friendly_name=0x00007ffc46a6b300, const coreclr_property_bag_t & properties={...}, std::unique_ptr<coreclr_t,std::default_delete<coreclr_t>> & inst={...}) 行 73  C++
  hostpolicy.dll!`anonymous namespace'::create_coreclr() 行 82 C++
  hostpolicy.dll!corehost_main(const int argc=1, const wchar_t * * argv=0x00000296ebe39560) 行 422 C++
  hostfxr.dll!execute_app(const std::wstring & impl_dll_dir, corehost_init_t * init=0x00000296ebe4b3f0, const int argc=1, const wchar_t * * argv=0x00000296ebe39560) 行 145  C++
  hostfxr.dll!`anonymous namespace'::read_config_and_execute(const std::wstring & host_command={...}, const host_startup_info_t & host_info, const std::wstring & app_candidate, const std::unordered_map<enum known_options,std::vector<std::wstring,std::allocator<std::wstring>>,known_options_hash,std::equal_to<enum known_options>,std::allocator<std::pair<enum known_options const ,std::vector<std::wstring,std::allocator<std::wstring>>>>> & opts, int new_argc=1, const wchar_t * * new_argv=0x00000296ebe39560, host_mode_t mode=apphost, const bool is_sdk_command=false, wchar_t * out_buffer=0x0000000000000000, int buffer_size=0, int * required_buffer_size=0x0000000000000000) 行 532  C++
  hostfxr.dll!fx_muxer_t::handle_exec_host_command(const std::wstring & host_command={...}, const host_startup_info_t & host_info={...}, const std::wstring & app_candidate={...}, const std::unordered_map<enum known_options,std::vector<std::wstring,std::allocator<std::wstring>>,known_options_hash,std::equal_to<enum known_options>,std::allocator<std::pair<enum known_options const ,std::vector<std::wstring,std::allocator<std::wstring>>>>> & opts={...}, int argc=1, const wchar_t * * argv=0x00000296ebe39560, int argoff=1, host_mode_t mode=apphost, const bool is_sdk_command=false, wchar_t * result_buffer=0x0000000000000000, int buffer_size=0, int * required_buffer_size=0x0000000000000000) 行 1007  C++
  hostfxr.dll!fx_muxer_t::execute(const std::wstring host_command={...}, const int argc=1, const wchar_t * * argv=0x00000296ebe39560, const host_startup_info_t & host_info={...}, wchar_t * result_buffer=0x0000000000000000, int buffer_size=0, int * required_buffer_size=0x0000000000000000) 行 578  C++
  hostfxr.dll!hostfxr_main_startupinfo(const int argc=1, const wchar_t * * argv=0x00000296ebe39560, const wchar_t * host_path=0x00000296ebe47eb0, const wchar_t * dotnet_root=0x00000296ebe46c80, const wchar_t * app_path=0x00000296ebe46920) 行 62 C++
  lindexi.exe!00007ff79d50aae3()
  lindexi.exe!00007ff79d50aef6()
  lindexi.exe!00007ff79d512818()
  kernel32.dll!BaseThreadInitThunk() 
  ntdll.dll!RtlUserThreadStart() 
```

最后的方法堆栈 `coreclr_t::create` 其实现如下

```csharp
    hr = coreclr_contract.coreclr_initialize(
        exe_path,
        app_domain_friendly_name,
        propertyCount,
        keys.data(),
        values.data(),
        &host_handle,
        &domain_id);
```

不能从以上方法看出来任何有用的信息，而且相同的 exe 文件，只有在这一台 Win10 设备上运行就崩溃，还没有找到自己的 C# 代码就崩溃。在其他设备上都能跑得好好的

如 <https://github.com/dotnet/docs/issues/42600> 的相关讨论帖子，尝试使用 `<CETCompat>false</CETCompat>` 禁用 CET 之后，就能好好运行了。以下是 [Jan Kotas](https://github.com/jkotas) 大佬在 <https://github.com/dotnet/runtime/issues/108589> 帖子上的原话：

> Some older Windows versions had issues in CET
> 一些较旧的 Windows 版本在 CET 方面存在问题

这台出现问题的 Win10 设备的 ntdll.dll 是 10.0.19041.1949 (WinBuild.160101.0800) 的版本，低于 [Jan Kotas](https://github.com/jkotas) 大佬说的所需 10.0.19041.5007 版本。以下是 [Jan Kotas](https://github.com/jkotas) 大佬在 <https://github.com/dotnet/runtime/issues/110920> 帖子上的原话：

> It means that ntdll.dll version 10.0.19041.5007 has the required CET fixes, but ntdll.dll version 10.0.19041.2788 does not have the required CET fixes, both come from Windows 22H2.
> 这意味着 ntdll.dll 版本 10.0.19041.5007 具有所需的 CET 修复，但 ntdll.dll 版本 10.0.19041.2788 没有所需的 CET 修复，它们都来自 Windows 22H2 版本

相关链接：

- <https://github.com/dotnet/runtime/issues/108589>
- <https://github.com/dotnet/runtime/issues/114671>
- <https://github.com/dotnet/runtime/issues/110920>
- <https://github.com/dotnet/runtime/issues/114673>
- <https://github.com/dotnet/runtime/issues/112253>