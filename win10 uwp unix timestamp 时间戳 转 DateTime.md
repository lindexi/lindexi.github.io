# win10 uwp unix timestamp 时间戳 转 DateTime

有时候需要把网络的  unix timestamp 转为 C# 的 DateTime ，在 UWP 可以如何转换？

<!--more-->
<!-- csdn -->

转换函数可以使用下面的代码

```csharp
        private static DateTime UnixTimeStampToDateTime(long unixTimeStamp)
        {
            System.DateTime dtDateTime = new System.DateTime(1970, 1, 1, 0, 0, 0, 0);
            dtDateTime = dtDateTime.AddSeconds(unixTimeStamp);
            return dtDateTime;
        }
```

如何从 DateTime 转 unix timestamp ，请用下面代码

```csharp
       public static long ToUnixTimestamp(DateTime time)
        {
            var date = new DateTime(1970, 1, 1, 0, 0, 0, time.Kind);
            var unixTimestamp = System.Convert.ToInt64((time - date).TotalSeconds);

            return unixTimestamp;
        }
```

如果就是这么简单代码，我就不会写博客专门来说。

我遇到一个问题，我拿到的是 json ，里面的时间是 unix timestamp ，我需要把 long 的时间转换 DateTime ，但是我不喜欢在使用的时候再经过转换，能够在写的时候，把所有的 unix timestamp 自动转换为 DateTime？我找到一个简单方法，可以从 Json 转换过程，直接把 DateTime 和 unix timestamp  相互转换，方法很简单。

问题在：https://stackoverflow.com/q/44643498/6116637

下面来讲下如何解决。下面需要用到了 JsonConverter 的高级用法。首先需要使用 Nuget 下载 json 的库，当然搜 json 下载第一个就好。

然后创建一个类，用于类型转换，类型转换的意思就是从输入一个类型转换为输出的一个类型

关于更多 json 高级使用，参见：http://www.cnblogs.com/yanweidie/p/4605212.html

```csharp
class UnixConvert : JsonConverter
```
可以看到，创建的类型必须重新三个函数

```csharp
WriteJson
ReadJson
CanConvert
```

很容易知道，`WriteJson` 就是从一个存在的类转换为 json ，遇到类型需要做什么转换。 `ReadJson` 从一个json转换为类的时候，遇到json需要如何转换。 `CanConvert`当前的输入是否支持转换。

当然这几个函数是对于属性的，所以读取一个值就好了。

一开始需要把  unix timestamp 转换为 DateTime ，所以就是从 json 的字符串转属性。

写的代码就是 `ReadJson` ，于是开始写这个函数

```csharp
        public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
        {
            long unixTimeStamp = long.Parse(reader.Value.ToString());
            return UnixTimeStampToDateTime(unixTimeStamp);
        }
```

这样就是转换的代码，看起来很简单。

读取 一个值，把他转 long ，然后使用上面的函数转换 DateTime ，为什么这里使用的是  long.Parse ，因为保证输入的json是对的，如果json不对，那么直接告诉错误才是好的做法。

和读函数反过来，需要把 属性转json的字符串，可以从参数看到，需要转换的值是 value ，这里使用强转，因为知道了他的类型。writer 可以直接写入 很多类型

```csharp
      public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
        {
            var time = ToUnixTimestamp((DateTime) value);
            writer.WriteValue(time);
        }
```

看起来大概就写好了，但是最后一个函数只需要返回 true 就好，暂时不需要做什么。

开始写一个例子进行测试。

测试之前，我先把上面的转换类所有代码写出来

```csharp
 class UnixConvert : JsonConverter
    {
        public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
        {
            var time = ToUnixTimestamp((DateTime) value);
            writer.WriteValue(time);
        }

        public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
        {
            long unixTimeStamp = long.Parse(reader.Value.ToString());
            return UnixTimeStampToDateTime(unixTimeStamp);
        }

        public override bool CanConvert(Type objectType)
        {
            return true;
        }

        private static DateTime UnixTimeStampToDateTime(long unixTimeStamp)
        {
            System.DateTime dtDateTime = new System.DateTime(1970, 1, 1, 0, 0, 0, 0);
            dtDateTime = dtDateTime.AddSeconds(unixTimeStamp);
            return dtDateTime;
        }

        public static long ToUnixTimestamp(DateTime time)
        {
            var date = new DateTime(1970, 1, 1, 0, 0, 0, time.Kind);
            var unixTimestamp = System.Convert.ToInt64((time - date).TotalSeconds);

            return unixTimestamp;
        }
    }
```

<script src="https://gist.github.com/lindexi/9a5bc7cd455add6ab87f81270dbf9768.js"></script>

测试是写一个类，把他进行转换json，然后使用json转类，看得到结果是否一样。

```csharp
   class Foo
    {
        [JsonConverter(typeof(UnixConvert))]
        public DateTime created_utc { set; get; }
    }

             Foo foo = new Foo()
            {
                created_utc = DateTime.Now
            };
            var str = JsonConvert.SerializeObject(foo);
            foo = JsonConvert.DeserializeObject<Foo>(str);
```

转换 得到json 为 `{"created_utc":1498037127}` 

因为我使用的是 DateTime.Now ，所以如果大家使用这个类，得到的结果可能和我得到的不一样。

可以看到从json转换结果和创建的类的属性一样，所以这个方法可以拿来使用。

https://stackoverflow.com/questions/44643498/convert-unix-timestamp-to-normal-date-uwp/44650513#44650513

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  