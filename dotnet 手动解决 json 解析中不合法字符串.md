# dotnet 手动解决 json 解析中不合法字符串

如果使用 Newtonsoft Json 解析字符串，字符串里面有不清真的格式，那么默认的解析将会炸掉。如果想要自己解决字符串中的不清真格式，可以使用传入 JsonSerializerSettings 的方法

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


我写了一段字符串，里面需要解析为日期请看代码

```csharp
@"[
      '2009-09-09T00:00:00Z',
      '这是歪楼的',
      [
        1
      ],
      '1977-02-20T00:00:00Z',
      null,
      '2000-12-01T00:00:00Z'
    ]"
```

小伙伴很快就可以看到，这里有一个歪楼的字符串

如果直接解析，那么将会炸掉

```csharp

            List<DateTime> list = JsonConvert.DeserializeObject<List<DateTime>>(@"[
      '2009-09-09T00:00:00Z',
      '这是歪楼的',
      [
        1
      ],
      '1977-02-20T00:00:00Z',
      null,
      '2000-12-01T00:00:00Z'
    ]");
```

运行上面代码，可以看到一个有趣的代码

![](http://image.acmx.xyz/lindexi%2F20193208581815)

自己解决不合法的字符串可以通过 JsonSerializerSettings 的方法，里面有一个委托，在发现有不清真的字符串的时候就会调用这个委托

```csharp
             List<string> errors = new List<string>();

               new JsonSerializerSettings
                {
                    Error = (sender, e) =>
                    {
                        errors.Add(e.ErrorContext.Error.Message);
                        e.ErrorContext.Handled = true;
                    },
                    Converters = { new IsoDateTimeConverter() }
                }
```

其实这里的 DeserializeObject 可以传入两个参数，一个是字符串，一个是 JsonSerializerSettings 所以就可以做到在发现不清真的字符串的时候调用委托，同时支持传入从字符串转类的方法

```csharp
           List<string> errors = new List<string>();

            List<DateTime> list = JsonConvert.DeserializeObject<List<DateTime>>(@"[
      '2009-09-09T00:00:00Z',
      '这是歪楼的',
      [
        1
      ],
      '1977-02-20T00:00:00Z',
      null,
      '2000-12-01T00:00:00Z'
    ]",
                new JsonSerializerSettings
                {
                    Error = (sender, e) =>
                    {
                        errors.Add(e.ErrorContext.Error.Message);
                        e.ErrorContext.Handled = true;
                    },
                    Converters = { new IsoDateTimeConverter() }
                });
```

这时运行代码不会炸掉，因为设置了 `e.ErrorContext.Handled` 是 true 如果想要记录问题同时需要炸掉，那么就不要写这个代码

所有代码请看 [github](https://github.com/lindexi/lindexi_gd/tree/7d553d290d68b567f91daed16c814c023e90c1a8/BepirquwiKedoucawji)

[Serialization Error Handling](https://www.newtonsoft.com/json/help/html/SerializationErrorHandling.htm )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
