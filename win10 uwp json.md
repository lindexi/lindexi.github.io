# win10 uwp json

本文讲的是关于在uwp使用json的简单使用，json应用很多，因为我只是写简单使用，说的东西可能不对或者不符合每个人的预期。如果觉得我有讲的不对的，就多多包含，或者直接关掉这篇文章，但是请勿生气或者发怒吐槽，可以在我博客评论 http://blog.csdn.net/lindexi_gd

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


<div id="toc"></div>

现在很多应用都是使用json

如果我们拿到一段json，想要把它变为我们C艹艹可以用的，我们需要先对json的类进行转换，其实很简单，我们在复制一段json

不需要我们对这json转换为类，如果需要手动写很容易写错。因为我们有vs，在我们的编辑，可以看到粘贴为类，这个功能在 VisualStudio 2017 是没有的，需要安装插件。

![这里写图片描述](http://img.blog.csdn.net/20160607111953153)

我们复制完一段json，然后点击粘贴，就好了，自动生成对应的类，不过类的命名有些奇怪。

如果我们拿到一段json，请看下面

```json
{
  "results": 
  [
  {
    "location": 
    {
      "id": "WX4FBXXFKE4F",
      "name": "北京",
      "country": "CN",
      "path": "北京,北京,中国",
      "timezone": "Asia/Shanghai",
      "timezone_offset": "+08:00"
    },
    "daily": 
    [
    {      
      "date": "2015-09-20",           
      "text_day": "多云",              
      "code_day": "4",                 
      "text_night": "晴",              
      "code_night": "0",              
      "high": "26",                
      "low": "17",                 
      "precip": "0",                 
      "wind_direction": "",           
      "wind_direction_degree": "255",   
      "wind_speed": "9.66",            
      "wind_scale": ""                 
    },
     {
      "date": "2015-09-21",
      "text_day": "晴",
      "code_day": "0",
      "text_night": "晴",
      "code_night": "0",
      "high": "27",
      "low": "17",
      "precip": "0",
      "wind_direction": "",
      "wind_direction_degree": "157",
      "wind_speed": "17.7",
      "wind_scale": "3"
    }, 
    "last_update": "2015-09-20T18:00:00+08:00"
  }]
}
```

如果需要转换这个类，那么就需要在编辑让VisualStudio自动把 json 转类。

```csharp
    public class Thinw
    {

        public class Rootobject
        {
            public Result[] results { get; set; }
        }

        public class Result
        {
            public Location location { get; set; }
            public Daily[] daily { get; set; }
            public DateTime last_update { get; set; }
        }

        public class Location
        {
            public string id { get; set; }
            public string name { get; set; }
            public string country { get; set; }
            public string path { get; set; }
            public string timezone { get; set; }
            public string timezone_offset { get; set; }
        }

        public class Daily
        {
            public string date { get; set; }
            public string text_day { get; set; }
            public string code_day { get; set; }
            public string text_night { get; set; }
            public string code_night { get; set; }
            public string high { get; set; }
            public string low { get; set; }
            public string precip { get; set; }
            public string wind_direction { get; set; }
            public string wind_direction_degree { get; set; }
            public string wind_speed { get; set; }
            public string wind_scale { get; set; }
        }
    }
```

很快我们就得到了我们需要序列的类

接着我们使用Nuget安装一下插件

![这里写图片描述](http://img.blog.csdn.net/20160607112008346)

当然我还加上九幽的插件，九幽有几个插件可以获得我们应用数据，我们启动我们关闭，还有广告很好用

我们使用 Nuget 主要下载 Newtonsoft.Json ，这是一个很好用的、性能很高的序列化和反序列。

我们转序可以使用下面代码，实际上只是传入一个泛型而已，反序列的代码实际上可以一句话来写。

```csharp
        public void Unencoding(string str)
        {
            var json = JsonSerializer.Create();
            Rootobject thinw = json.Deserialize<Rootobject>(new JsonTextReader(new StringReader(str)));
        }
```

如果我们需要把我们的类转为json，这叫序列化，可以尝试使用下面的代码

```csharp
            var file = await ApplicationData.Current.LocalFolder.CreateFileAsync("data", CreationCollisionOption.ReplaceExisting);
            using (TextWriter stream = new StreamWriter(await file.OpenStreamForWriteAsync()))
            {
                json.Serialize(stream, thinw);
            }
```

这样把我们的类保存在文件，上面用到 UWP 文件读写。

如果觉得我做的简单，想要使用微软的`Windows.Data.Json` ，其实使用`Newtosoft`的才好

如果使用`Windows.Data.Json`，那么需要这样写

```csharp
JsonArray root = JsonValue.Parse(jsonString).GetArray();  
for (uint i = 0; i < root.Count; i++) 
{  
    string name1 = root.GetObjectAt(i).GetNamedString("name");  
    string description1 = root.GetObjectAt(i).GetNamedString("description");  
    string link1 = root.GetObjectAt(i).GetNamedString("link");  
    string cat1 = root.GetObjectAt(i).GetNamedString("cat");  
    string image1 = root.GetObjectAt(i).GetNamedString("image");  
    var chan = new RootObject 
    {  
        name = name1, 
        cat = cat1, 
        description = description1, 
        image = image1, 
        link = link1  
    };  
    obs_channels.Add(chan);  
}

```

如果属性多，基本上很多人会容易就打错，当然，可以先实例一个RootObject，然后使用新关键字，name去得到实例属性名称当然在我们使用Json会遇到一些属性我们不要的，那么如何json忽略属性，其实很简单，在Newtosoft可以在属性加[JsonIgnore]，因为这些比较乱，所以也不打算在这里说。

首先是我们的类，任意的定义

```csharp
  public class Thine
  {
      public string Property{set;get;}
      public string Ignore{set;get;}
  }
```

我们要把Property包含，但是不包含Ignore，简单的代码是添加`JsonIgnore`告诉序列化不要序列化这个属性。

```csharp
  public class Thine
  {
      public string Property{set;get;}
      [JsonIgnore]
      public string Ignore{set;get;}
  }
```

但是有时候我们要包含很少，基本都是不包含的，那么如何只用包含，其实我们可以在类加`[JsonObject(MemberSerialization.OptIn)]`看到了OptIn，其实OptOut就是不包含一些，OptIn就是包含一些


```csharp

  [JsonObject(MemberSerialization.OptIn)]
  public class Thine
  {
      [JsonProperty]
      public string Property{set;get;}
      public string Ignore{set;get;}
  }
```

在反序列化时需要注意，不是所有的类都可以反序列化，必须类的属性最后都是基础类型，必须类有默认构造函数没有参数。所以 StorageFile 等就是无法反序列化的，需要使用[这篇文章](./win10-uwp-%E4%BF%9D%E5%AD%98%E7%94%A8%E6%88%B7%E9%80%89%E6%8B%A9%E6%96%87%E4%BB%B6%E5%A4%B9.html )的方法保存用户选择的文件。

## 序列枚举

例如有一个枚举


```csharp
    public enum Foo
    {
        lindexi,
        jiuyou,
        oschina,
        mslaji,
        yam,
    }
```

进行序列，可以使用下面代码


```csharp
             List<Foo> foo=new List<Foo>()
            {
                Foo.lindexi,
                Foo.oschina
            };

            string str = JsonConvert.SerializeObject(foo);
```


结果是


```csharp
    [0,2]
```

如果对上面的进行反序列会出现和原先一样的结果

但是如果尝试使用其他的枚举，不会出现错误，枚举的数量比原先多的一般就不会出现


```csharp
                var t = JsonConvert.DeserializeObject<List<KeyboardNavigationMode>>(str);

```
原来的是 Foo ，现在改为 KeyboardNavigationMode 结果还是一样

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201754151047.jpg)

如果是数值的，容易出现这个错误那么如何使用枚举的字符串？

在需要使用的枚举类添加 JsonConverter


```csharp
     [JsonConverter(typeof(StringEnumConverter))]
    public enum Foo
    {
        lindexi,
        jiuyou,
        oschina,
        mslaji,
        yam,
    }
```

得到结果


```csharp
    ["lindexi","oschina"]
```

但是如果使用的枚举不是自己写的，如使用  Key 枚举，这个是ms写的，不可以在枚举加上，这时可以在属性加上

例如有个属性 


```csharp
         
            List<Foo> foo = new List<Foo>()
            {
                Foo.lindexi,
                Foo.oschina
            };   
```

可以在属性转换时用 

```csharp
                string str = JsonConvert.SerializeObject(foo,new StringEnumConverter());

```

这样可以做到和上面一样的方法


如果需要转换中文 
参见：http://www.cnblogs.com/DomoYao/p/Json.html

参见：http://blog.csdn.net/lovegonghui/article/details/50248455

## 换行输出

默认的 SerializeObject 输出的字符串是不换行的。例如我序列化了下面的类

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018222142547.jpg)

结果拿到的数据是：

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018222142719.jpg)

通过下面的代码就可以让 json 在转换的时候换行

```csharp
   var jsonSerializerSettings = new JsonSerializerSettings();
                jsonSerializerSettings.Formatting = Formatting.Indented;

                string str = JsonConvert.SerializeObject(obj, jsonSerializerSettings);
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018222143058.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。