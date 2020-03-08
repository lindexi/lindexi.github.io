# win10 uwp 网络编程
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>
<!-- csdn -->

 我们来弄简单协议
 
 首先第一层是传输层，这一层是Head+Data
 
 其中Head就是有传输的头，加上Data长度
 
 Head=Head+length
 
 length是Data长度
 
 上面传输的头，Head=Head+length 中的第二个Head，包含
 传输者id，当前传输是传输的消息最后一段还是中间,当前传输
 是服务器第消息
 
 传输的最后一段还是中间指的是在上一层，有很长的数据，被拆为多个Data发送，
 这时就需要标注接下来几条消息要合并为一条
 
 传输头Head=id+stx+count
 
 count就是服务器随机给的序号，客户端接收到，就返回接收到+count，这样服务器就可以知道客户端收到，如果超过时间，那么服务器就重新发送
 
 id、count都是16位int，我上面说的服务器其实就是发送的，客户端就是接收，不是说服务器协议
 
 我们接收是byte那么如何从byte两个转换为int，其实简单，short才对，不过说16位那是我C艹
 
 我们一个byte是高8位，一个是低8位
 
 `value = 256 * high + low;`
 
 `(high << 8) | low`
 
 stx=stx+length stx和length是8位byte
 
 `stx=1`是发送中间
 
 `stx=2`是发送结束
 
 `stx=3`是回复
 
 length在`stx=1` `stx=2`是表示消息顺序，如果接收到`stx=1`那么把消息放到缓存区，然后直到接收到`stx=2`才把消息合并，通知，这时需要按照length顺序组合，如果接收到的length不对，缺少，那么废掉信息。
 
 
 我们开始接收就接收16+16+16+16位，然后使用length接收下面，这样就可以解决

技术很简单，其实我们需要做服务器，和客户端，一般我们可以在UWP做两个，这个参见http://www.wangchenran.com/uwp-streamsocket-chatroom-1.html
 
我们可以用传输数据，可以传输文本，这个我们需要传什么需要一个协议，这个是应用高层，前面说的是协议传输

## 带Header的WebRequest

```csharp
HttpRequestMessage httpRequestMessage = new HttpRequestMessage(
    HttpMethod.Post, new Uri("http://www.contoso.com"));
httpRequestMessage.Content = new HttpStringContent("hello, world");
httpRequestMessage.Headers.Append("X-My-Client","true");

myWebView.NavigateWithHttpRequestMessage(httpRequestMessage);
```

```csharp
            var url = new Uri("http://www.baidu.com", UriKind.Absolute);
            var httpClient = new System.Net.Http.HttpClient();
            httpClient.DefaultRequestHeaders.Add("apikey", "123456");
            var reponse = await httpClient.GetStreamAsync(url);
            var streamReader = new StreamReader(reponse, Encoding.UTF8);
            var jsonString = streamReader.ReadToEnd();
            var jsonObject = JsonObject.Parse(jsonString);
```

## 获取Buffer

```csharp
                Windows.Web.Http.HttpClient http = new Windows.Web.Http.HttpClient();
                IBuffer buffer = await http.GetBufferAsync(uri);

```

参见：https://msdn.microsoft.com/zh-cn/library/windows/apps/windows.ui.xaml.controls.webview.navigatewithhttprequestmessage.aspx




 
 

## HttpClient 使用 Cookie 

参见：https://stackoverflow.com/questions/41599384/httpclient-cookie-issue

如果不想自己写，请看 https://github.com/chenrensong/WebSocket.UWP

https://github.com/rdavisau/sockets-for-pcl