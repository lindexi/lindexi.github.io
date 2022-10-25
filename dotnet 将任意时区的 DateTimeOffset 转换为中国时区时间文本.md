# dotnet 将任意时区的 DateTimeOffset 转换为中国时区时间文本

本文告诉大家在拿到任意时区的 DateTimeOffset 对象，将 DateTimeOffset 转换为使用中国的 +8 时区表示的时间

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

在开始之前，需要说明的是，采用 DateTimeOffset 会比 DateTime 更优的一个点是 DateTimeOffset 是带上时区的，这就意味着方便的在多个不同的时区进行传递和序列化的时候，不会丢失原来的信息

现在也推荐更多的使用 DateTimeOffset 类型而不是 DateTime 类型，除非是明确只有本机时间且后续没有需求变更才会考虑使用 DateTime 类型

可选的转换为任意国家地区的时区时间，可以是先通过 TimeZoneInfo 的 FindSystemTimeZoneById 获取到对应的国家地区的信息，如下面代码获取到中国的信息

```csharp
                var timeZoneInfo = TimeZoneInfo.FindSystemTimeZoneById("China Standard Time");
```

这里的 FindSystemTimeZoneById 传入的 Id 可选的列表可以参阅你的本机注册表的 `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Time Zones` 的列表，详细请看 [TimeZoneInfo.FindSystemTimeZoneById(String) 方法 (System) Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/api/system.timezoneinfo.findsystemtimezonebyid?view=net-6.0 )

假设能获取到 TimeZoneInfo 那可以通过 GetUtcOffset 获取对比传入的 DateTimeOffset 的时间

```csharp
     var timeSpan = timeZoneInfo.GetUtcOffset(dateTimeOffset);
```

如此获取到的 TimeSpan 就是时区之间的差值，相加即可转换为目标国家地区的时间

```csharp
                var newDateTimeOffset = dateTimeOffset + timeSpan;
```

以上代码拿到的 `newDateTime` 就是转换后的时区时间

全部的代码如下，通过以下代码即可将任意时区的时间转换为中国对应的时区的时间

```csharp
                TimeZoneInfo? timeZoneInfo = TimeZoneInfo.FindSystemTimeZoneById("China Standard Time");
                TimeSpan timeSpan = timeZoneInfo.GetUtcOffset(dateTimeOffset);

                DateTimeOffset newDateTimeOffset = dateTimeOffset + timeSpan;
```

修改 `China Standard Time` 字符串为其他国家地区的，即可转换为其他国家地区的时区

在有一些奇怪的系统上，会抛出 TimeZoneNotFoundException 异常，此时可以使用固定中国的 +8 小时作为对比 UTC 时间

此时需要先将传入的 DateTimeOffset 转换为 UTC 时间，代码如下

```csharp
                DateTimeOffset utcDateTimeOffset = dateTimeOffset.ToUniversalTime();
```

接着直接执行 +8 小时即可转换为中国时间

```csharp
                TimeSpan timeSpan = TimeSpan.FromHours(8);
                DateTimeOffset newDateTimeOffset = utcDateTimeOffset + timeSpan;
```

全部的代码如下

```csharp
                DateTimeOffset utcDateTimeOffset = dateTimeOffset.ToUniversalTime();
                TimeSpan timeSpan = TimeSpan.FromHours(8);
                DateTimeOffset newDateTimeOffset = utcDateTimeOffset + timeSpan;
```

以上就是转换为中国时间的方法