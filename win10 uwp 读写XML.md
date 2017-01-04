# win10 uwp 读写XML



## XmlDocument

创建 XmlDocument ，创建 XmlDocument 有三个方法，首先是从 StorageFile 创建。
		
```csharp
            XmlDocument.LoadFromFileAsync(file);   


```

注意要等待。

第二方法：从Uri创建，`XmlDocument.LoadFromUriAsync(uri);   `

第三方法：先创建一个 XmlDocument 然后使用 Load
		
```csharp
            
            XmlDocument doc = new XmlDocument();
              
            doc.LoadXml(str);

```

注意str是字符串。



## Linq

http://www.cnblogs.com/portalsky/archive/2008/09/11/1289461.html

http://blog.csdn.net/cdjcong/article/details/8473539

http://blog.csdn.net/ht_zhaoliubin/article/details/38900275



