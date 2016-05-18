# win10 UWP Hmac

HMAC是密钥相关的哈希运算消息认证码，输入密钥和信息。

在uwp，Hmac在很多网络使用，我最近写qiniu SDK，把原来C#改为UWP，需要使用HMAC。

上传文件

```
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

需要凭据，凭据有上传策略


```
            string str_alg_name = MacAlgorithmNames.HmacSha1;
            MacAlgorithmProvider obj_mac_prov = MacAlgorithmProvider.OpenAlgorithm(str_alg_name);
            IBuffer buff_msg = CryptographicBuffer.CreateFromByteArray(path_and_query_bytes);
            IBuffer buff_key_material = CryptographicBuffer.CreateFromByteArray(mac.SecretKey);
            CryptographicKey hmac_key = obj_mac_prov.CreateKey(buff_key_material);
            IBuffer hmac = CryptographicEngine.Sign(hmac_key, buff_msg);
            byte[] digest = hmac.ToArray();
```

`string str_alg_name = MacAlgorithmNames.HmacSha1;`微软有AesCmac、HmacMd5、HmacSha1、HmacSha256、HmacSha384、HmacSha512

`MacAlgorithmProvider.OpenAlgorithm`传入使用算法

Hmac输入buffer，byte`CryptographicBuffer.CreateFromByteArray`

Hmac密钥`obj_mac_prov.CreateKey(buff_key_material)`

