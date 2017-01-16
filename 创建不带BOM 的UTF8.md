# 创建不带BOM 的UTF8



<!--more-->

```csharp
  Encoding utf8WithoutBom = new UTF8Encoding(false);
```


```csharp
  Encoding isoLatin1Encoding = Encoding.GetEncoding("ISO-8859-1");
```



参见：http://stackoverflow.com/questions/2502990/create-text-file-without-bom