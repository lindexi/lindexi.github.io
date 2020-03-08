# win10 UWP Hmac

HMAC是密钥相关的哈希运算消息认证码，输入密钥和信息。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

在uwp，Hmac在很多网络使用，我最近写qiniu SDK，把原来C#改为UWP，需要使用HMAC。

上传文件时需要填写 form ，这是官方要求的

```xml
<form method="post" action="http://upload.qiniu.com/"
 enctype="multipart/form-data">
  <input name="key" type="hidden" value="<resource_key>">
  <input name="x:<custom_name>" type="hidden" value="<custom_value>">
  <input name="token" type="hidden" value="<upload_token>">
  <input name="file" type="file" />
  <input name="crc32" type="hidden" />
  <input name="accept" type="hidden" />
</form>
```

里面需要凭据，凭据有上传策略，而做这个需要 Hmac，我找了好久才得到，希望大家遇到 Hmac 问题可以在我这里发现解决方法


```csharp
            string str_alg_name = MacAlgorithmNames.HmacSha1;
            MacAlgorithmProvider obj_mac_prov = MacAlgorithmProvider.OpenAlgorithm(str_alg_name);
            IBuffer buff_msg = CryptographicBuffer.CreateFromByteArray(path_and_query_bytes);
            IBuffer buff_key_material = CryptographicBuffer.CreateFromByteArray(mac.SecretKey);
            CryptographicKey hmac_key = obj_mac_prov.CreateKey(buff_key_material);
            IBuffer hmac = CryptographicEngine.Sign(hmac_key, buff_msg);
            byte[] digest = hmac.ToArray();
```

`string str_alg_name = MacAlgorithmNames.HmacSha1;`是从预设的算法中拿出Hmac，而微软有这么多算法：AesCmac、HmacMd5、HmacSha1、HmacSha256、HmacSha384、HmacSha512

`MacAlgorithmProvider.OpenAlgorithm` 传入使用算法

Hmac 输入是 buffer，如果我们只有 byte 请使用 `CryptographicBuffer.CreateFromByteArray` 转Buffer

Hmac密钥 `obj_mac_prov.CreateKey(buff_key_material)`

最后使用 ` CryptographicEngine.Sign(hmac_key, buff_msg);` 

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。