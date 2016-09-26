#win10 uwp 九幽图床

本文主要是图片加水印自动上传，代码已经上传github。

图片加水印，我的方法，简单，一个好的方法是毒逆天大神的图片加水印方法或http://daily.zhihu.com/story/8812028

我们现在用我的方法，就是加一个TextBlock，然后获得屏幕，很简单，然后我们把水印图保存。

我们可能有很多地方需要上传，我现在使用是九幽，之前有smms的图床，我们做一个通用任务。

 - Guid 上传的Guid，为了识别任务

 - File 要上传的图片

 - Name 上传图片的名字，没有用

 - OnUploaded 上传完成EventHandler<bool> 成功true

 - Url 上传成功返回图片

 - Scale 缩放比例，如果没有设置，默认-1，上传图片不缩放

 - Width 默认-1，图片宽度，有设置上传为设置宽度

 - Height 默认-1，图片高度

 - public abstract void UploadImage() 上传图片，不同的类可以有不同上传

我们类需要传StorageFile，这个必须，因为我们上传必须有文件。

九幽上传很简单，首先是Nuget下载

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=2a5bef95bb3678d2836d9d2e0753e754)

我们搜索Jiuyou，有很多九幽的，很好用，但是我们这里下载JyCloud

然后我们写九幽上传，我们基层我们通用任务，我们需要写UploadImage

`public class JyUploadImage : UploadImageTask`

九幽上传可以有多个函数。

- sendFileScale 上传图片可以缩放。我们判断Scale》0，如果是，我们就用这函数上传。

- sendFileCustom 上传图片设置图片宽度和高度，我们判断(uploadImageTask.Width > 0) && (uploadImageTask.Height > 0)

- sendFileOriginal上传原图

- sendFileDefault 当图片的宽或高大于某个值时，将高或宽等比缩放到这个值

  九幽上传开始需要appKey，这个可以在`http://www.windows.sc`创建应用然后得到。

我们先登录[http://www.windows.sc](http://www.windows.sc),选应用。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=cc9dbe08cc1697a37b5ebf29a101a66d)

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=a042475a5c7e36609567708ba9904f43)

我们可以创建应用，我们可以用我们创建的应用，可以看到key

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=5ead83cda56807f1502d939630fb6419)

我们创建一个类来放我们的key，我们上传还需要我们的图床密钥。

图床，九幽叫云存储。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=6cfff56cd26ce5c74a07969517f122d6)

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=5538bd7846ed0ad0caab77fc93dd4c1a)

我们可以看到我们的key，这个是和应用没关。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=4607f51cd9435b25a7602ca0318dff8c)

我们上传的是我们的StorageFile，上传完可以有ResponseInfo

```
        private async Task UploadImage(UploadImageTask uploadImageTask)
        {
            //Appid 为静态，有_appid 应用的appid
            //_secretId 九幽的
            ResponseInfo responseInfo;
            if (uploadImageTask.Scale > 0)
            {
                responseInfo = await JyCloudTool.JyCloudTool.sendFileScale(
                    AppId._appId, AppId._secretId, uploadImageTask.File,
                    uploadImageTask.Scale);
            }
            else if ((uploadImageTask.Width > 0) && (uploadImageTask.Height > 0))
            {
                responseInfo = await JyCloudTool.JyCloudTool.sendFileCustom(
                    AppId._appId, AppId._secretId, uploadImageTask.File,
                    (uint) uploadImageTask.Width, (uint) uploadImageTask.Height);
            }
            else
            {
                responseInfo = await JyCloudTool.JyCloudTool.sendFileOriginal(
                    AppId._appId, AppId._secretId, uploadImageTask.File);
            }
            if (responseInfo.respose_Status == 0)
            {
                uploadImageTask.Url = responseInfo.ImgUrl;
                uploadImageTask.OnUploaded?.Invoke(uploadImageTask, true);
            }
            else
            {
                uploadImageTask.OnUploaded?.Invoke(uploadImageTask, false);
            }
        }


```

respose_Status是上传的状态

|0|成功|
|--|--|
|1|参数缺失|
|2|服务冻结|
|3|sdk版本过低|
|4|密钥错误|
|5|签名错误(请检查SerectId是否正确）|
|-1|文件为空|
|-2|文件大小超出限制（8MB）|
|-3|接口命令错误|
|-4|服务器报错|
|-5|文件类型不支持|
|-6|积分不足上传失败|
|-7|未检测到网络连接|
|-8|文件格式出错，暂时只支持png,jpg格式|

源代码：https://github.com/lindexi/UWP/tree/master/uwp/control/BitStamp 

 <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。