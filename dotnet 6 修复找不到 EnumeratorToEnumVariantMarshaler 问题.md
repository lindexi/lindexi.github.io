# dotnet 6 修复找不到 EnumeratorToEnumVariantMarshaler 问题

我将在一个 .NET Framework 项目升级到 dotnet 6 时发现构建不通过，因为原先的代码使用到了 EnumeratorToEnumVariantMarshaler 类型，在 dotnet 6 里面找不到。本文将告诉大家如何修复此问题

<!--more-->
<!-- CreateTime:2023/3/28 14:54:11 -->


<!-- 发布 -->
<!-- 博客 -->

在 .NET Framework 定义的 [EnumeratorToEnumVariantMarshaler](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.custommarshalers.enumeratortoenumvariantmarshaler ) 类型是在 dotnet core 里缺失的

代替方法是使用 MarshalType 属性写字符串，如以下代码

```csharp
        [MethodImpl(MethodImplOptions.InternalCall)]
        [DispId(1)]
        [TypeLibFunc(1)]
        [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalType = "System.Runtime.InteropServices.CustomMarshalers.EnumeratorToEnumVariantMarshaler")]
        IEnumerator GetEnumerator();
```


参考文档

[CustomMarshalers - EnumeratorToEnumVariantMarshaler · Issue #47243 · dotnet/runtime](https://github.com/dotnet/runtime/issues/47243 )