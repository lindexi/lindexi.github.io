# win10 uwp json

本文讲的是关于在uwp使用json的简单使用，json应用很多，因为我只是写简单使用，可能带有一定的主观性和局限性，说的东西可能不对或者不符合每个人的预期。如果觉得我有讲的不对的，就多多包含，或者直接关掉这篇文章，但是请勿生气或者发怒吐槽，可以在我博客评论 http://blog.csdn.net/lindexi_gd

现在很多应用都是使用json

如果我们拿到一段json，想要把它变为我们C艹艹可以用的，我们需要先对json的类进行转换，其实很简单，我们在复制一段json

不需要我们对这json打，因为我们有vs，在我们的编辑，可以看到

![这里写图片描述](http://img.blog.csdn.net/20160607111953153)

我们复制完一段json，然后点击粘贴，就好了，自动生成

如果我们拿到一段json

```
{
  "results": [{
    "location": {
      "id": "WX4FBXXFKE4F",
      "name": "北京",
      "country": "CN",
      "path": "北京,北京,中国",
      "timezone": "Asia/Shanghai",
      "timezone_offset": "+08:00"
    },
    "daily": [{      
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
    }, {
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
    }, {
    }],
    "last_update": "2015-09-20T18:00:00+08:00" 
  }]
}
```

我们弄个新的类，粘贴

```
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

接着我们使用Nuget

![这里写图片描述](http://img.blog.csdn.net/20160607112008346)

当然我还加上九幽的插件，九幽有几个插件可以获得我们应用数据，我们启动我们关闭，还有广告很好用

我们使用Nuget主要下载Newtonsoft.Json

我们转序可以使用

```
        public void Unencoding(string str)
        {
            var json = JsonSerializer.Create();
            Rootobject thinw = json.Deserialize<Rootobject>(new JsonTextReader(new StringReader(str)));
        }
```

如果我们需要把我们的类转为json

```
            var file = await ApplicationData.Current.LocalFolder.CreateFileAsync("data", CreationCollisionOption.ReplaceExisting);
            using (TextWriter stream = new StreamWriter(await file.OpenStreamForWriteAsync()))
            {
                json.Serialize(stream, thinw);
            }
```

这样把我们的类保存在文件

如果觉得我做的简单，想要使用微软的Windows.Data.Json ，其实使用Newtosoft的才好

如果使用Windows.Data.Json

```
JsonArray root = JsonValue.Parse(jsonString).GetArray();  
for (uint i = 0; i < root.Count; i++) {  
    string name1 = root.GetObjectAt(i).GetNamedString("name");  
    string description1 = root.GetObjectAt(i).GetNamedString("description");  
    string link1 = root.GetObjectAt(i).GetNamedString("link");  
    string cat1 = root.GetObjectAt(i).GetNamedString("cat");  
    string image1 = root.GetObjectAt(i).GetNamedString("image");  
    var chan = new RootObject {  
        name = name1, cat = cat1, description = description1, image = image1, link = link1  
    };  
    obs_channels.Add(chan);  
}); 

```

如果属性多，基本上很多人会容易就打错，当然，可以先实例一个RootObject，然后使用新关键字，name去得到实例属性名称

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

