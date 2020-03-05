# win10 UWP 应用设置

win10 UWP 应用设置 简单的把设置需要的，放到微软自带的LocalSettings LocalSettings.Values可以存放几乎所有数据 如果需要存放复合数据，一个设置项是由多个值组成，可以使用ApplicationDataCompositeValue将多个合并。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

存放一个string

```csharp
string str
{
  set
  {
        ApplicationData.Current.LocalSettings.Values["str"] = value;
  }
  get
  {
            object temp;
            if (ApplicationData.Current.LocalSettings.Values.TryGetValue("width", out temp))
            {
                return  temp as string;
            }
  }
}
```

如果设置在LocalSettings让程序太乱，有很多变量名称一样，可以使用新的ApplicationDataContainer

           string str = "";
            var t = ApplicationData.Current.LocalSettings.CreateContainer("str", ApplicationDataCreateDisposition.Always);
            t.Values["str"] = str;
            str = t.Values["str"] as string;
 ```
 
 <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。