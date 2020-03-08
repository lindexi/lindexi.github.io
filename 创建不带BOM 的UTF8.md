# 创建不带BOM 的UTF8

如果使用 StreamWriter 创建的文本，都是默认带 BOM ，如果需要创建一个不带BOM的文件，请看本文。

<!--more-->
<!-- CreateTime:2018/5/19 14:11:33 -->


<div id="toc"></div>
<!-- csdn -->

因为有很多个编码，打开一个文件，很难判断这个文件是什么编码。所以微软就在文件的开始写入4个byte，来告诉程序这个文件是什么格式。需要知道，这个 BOM 是微软定义的，所以在很多的系统是没有 BOM 的，所以保存了一个  xml 文件，可以在其他系统读取就出错了，他们不知道 BOM 。

下面就来提供一个简单的方法创建不带 BOM 的文件。因为和编码有关系，所以只需要替换 StreamWriter 的编码就会好了，下面提供两个方法创建编码。

```csharp
  Encoding utf8WithoutBom = new UTF8Encoding(false);
```


```csharp
  Encoding isoLatin1Encoding = Encoding.GetEncoding("ISO-8859-1");
```

建议使用第一个方法，创建编码就可以开始写文件

下面是把 GBK 编码的文件读取然后转换为 UTF8 的代码，代码可以直接运行，当然需要修改文件为自己的文件。

```csharp
       static void Main(string[] args)
        {
            var file = new FileInfo("E:\\博客\\创建不带BOM 的UTF8.txt");
            string str = "";
            using (StreamReader stream = new StreamReader(file.FullName, Encoding.GetEncoding("GBK")))
            {
                str = stream.ReadToEnd();
            }

           Encoding utf8WithoutBom = new UTF8Encoding(false);
            using (StreamWriter stream = new StreamWriter(file.FullName, false, utf8WithoutBom))
            {
                stream.Write(str);
            }
        }
```

参见：http://stackoverflow.com/questions/2502990/create-text-file-without-bom

![](https://i.loli.net/2018/05/19/5affc0092037f.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  