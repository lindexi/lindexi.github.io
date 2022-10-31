
# dotnet 6 在 System.Text.Json 使用 source generation 源代码生成提升 JSON 序列化性能

这是一个在 dotnet 6 早就引入的功能，此功能的使用方法能简单，提升的效果也很棒。使用的时候需要将 Json 序列化工具类换成 dotnet 运行时自带的 System.Text.Json 进行序列化，再加上约 5 行的辅助代码，即可完成对接

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

官方文档： [如何在 System.Text.Json 中使用源生成 Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/standard/serialization/system-text-json/source-generation?pivots=dotnet-6-0)

先假定有名为 WeatherForecast 的类型，定义如下

```csharp
public class WeatherForecast
{
    public DateTime Date { get; set; }
    public int TemperatureCelsius { get; set; }
    public string? Summary { get; set; }
}
```

此类型将要进行 Json 的序列化

为了能让此类型接入 source generation 源代码生成，生成 Json 序列化和反序列化的自动生成代码逻辑，需要再加上以下的约 5 行的辅助代码

```csharp
[JsonSerializableAttribute(typeof(WeatherForecast))]
internal partial class MyJsonContext : JsonSerializerContext
{

}
```

以上代码是随意新建一个类型，类型的名称没有约束，让此类型继承 JsonSerializerContext 类型，且标记 JsonSerializableAttribute 特性，在此特性上面加上需要对接的原始数据类型，如 WeatherForecast 类

标记了 JsonSerializableAttribute 特性之后，将可以在 Visual Studio 的解决方案资源管理器里面的项目的依赖项->分析器->System.Text.Json.SourceGeneration->System.Text.Json.SourceGeneration.JsonSourceGeneration 里面，展开看到 xx.g.cs 文件

这些 xx.g.cs 文件就是生成的文件，想不开可以点进去看看，核心逻辑就是写了如何从原始数据类型进行转换获取 Json 的属性和值对应关系，也就是对每个属性构建出 JsonPropertyInfoValues 类型

比如说对上面的 WeatherForecast 的 Date 属性进行生成，生成的代码大概如下，以下代码最重要的有三个点，分别是如何获取和如何设置，以及属性名

```csharp
            new global::System.Text.Json.Serialization.Metadata.JsonPropertyInfoValues<global::System.DateTime>()
            {
                IsProperty = true,
                IsPublic = true,
                IsVirtual = false,
                DeclaringType = typeof(global::GemjabemrawWohearcebola.WeatherForecast),
                PropertyTypeInfo = jsonContext.DateTime,
                Converter = null,

                // 告诉 json 框架，如何进行赋值和获取值
                Getter = static (obj) => ((global::GemjabemrawWohearcebola.WeatherForecast)obj).Date,
                Setter = static (obj, value) => ((global::GemjabemrawWohearcebola.WeatherForecast)obj).Date = value!,
                IgnoreCondition = null,
                HasJsonInclude = false,
                IsExtensionData = false,
                NumberHandling = default,

                // 告诉 json 框架，这个属性名叫什么
                PropertyName = "Date",
                JsonPropertyName = null
            }
```

大家都知道，在 JSON 反序列化里面，可以强行分为两步。第一步就是对 JSON 这个格式本身的处理，展开为内存模型。第二步就是从展开的内存模型转换为具体的类型。同理，序列化也是相同的道理

其中第一步对 Json 这个格式本身的处理方面上，这是可以做到非常通用的。但是第二步就会根据具体的业务开发，转换为不同的类型。在此之前，这个转换步骤基本都是采用反射来实现的。而大家都知道，反射的性能是跟不上的。如今有了 Source Generation 源代码生成，就可以使用生成的代码实现此第二步的逻辑，通过生成的代码，从 json 展开的内存模型，对接到具体的类型上，给对应的具体类型的对象的各个属性调用赋值方法，通过生成的代码了解到对象的属性名，获取到匹配的属性的值

这就是为什么使用 Source Generation 源代码生成能在 Json 序列化中提升性能的原因

写完了 MyJsonContext 类型之后，再来看看如何对接 JSON 反序列化和序列化逻辑

只需要调用 JsonSerializer 的 对应的反序列化和序列化方法，传入 MyJsonContext 对应的参数即可，传入参数的作用是让 System.Text.Json 底层可以了解到如何应用上源代码生成的代码

假定有以下字符串，准备用来反序列化

```csharp
        string jsonString =
            @"{
  ""Date"": ""2019-09-01T00:00:00"",
  ""TemperatureCelsius"": 25,
  ""Summary"": ""Hot""
}
";
```

逻辑代码可以如此写

```csharp
        WeatherForecast? weatherForecast;

        weatherForecast = JsonSerializer.Deserialize<WeatherForecast>(
            jsonString, MyJsonContext.Default.WeatherForecast);
        Console.WriteLine($"Date={weatherForecast?.Date}");
```

可以看到，在 Deserialize 方法里面，将 MyJsonContext.Default.WeatherForecast 作为参数传入。如此即可让 System.Text.Json 使用到 MyJsonContext 的生成代码

反过来，在拿到 WeatherForecast 对象之后，可以调用 Serialize 方法进行序列化，同理传入 MyJsonContext.Default.WeatherForecast 用来对接生成代码

```csharp
        jsonString = JsonSerializer.Serialize(
            weatherForecast!, MyJsonContext.Default.WeatherForecast);
        Console.WriteLine(jsonString);
```

以上的例子代码都是从官方文档抄的，更多用法还请参阅官方文档

[如何在 System.Text.Json 中使用源生成 Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/standard/serialization/system-text-json/source-generation?pivots=dotnet-6-0)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。