----------

Title: win10 uwp 存放网络图片到本地

----------

有时候我们的网络很垃圾，我的的UWP要在第一次打开网络图片，就把图片存放到本地，下次可以从本地打开。

这就是先把图片下载，然后显示出来，存放到本地，接着下次要使用就可以从本地获取。

最好这个和我们用户是透明，我们不知道图片在哪，是本地还是网络，只要给一个Uri就有一个图片。

这里图片我用BitmapImage，Uri是输入网络的

## 下载图片

图片也是和其他一样，我们可以简单用系统给的网络web下载。

我们需要输入Uri，然后把图片下载。

图片要显示，需要SetSourceAsync，他需要的参数IRandomAccessStream，而这个需要Buffer写数据，不能用byte，我开始用的`System.Net.Http`没有获取Buffer方法，于是我查了垃圾wr，最后用`Windows.Web.Http`

先获取图片

```
                Windows.Web.Http.HttpClient http = new Windows.Web.Http.HttpClient();
                IBuffer buffer = await http.GetBufferAsync(uri);

                BitmapImage img = new BitmapImage();
                using (IRandomAccessStream stream = new InMemoryRandomAccessStream())
                {
                    await stream.WriteAsync(buffer);
                    stream.Seek(0);
                    await img.SetSourceAsync(stream);
                    await StorageImageFolder(stream, uri);
                    return img;
                }

```

StorageImageFolder就是保存图片

## 保存图片





