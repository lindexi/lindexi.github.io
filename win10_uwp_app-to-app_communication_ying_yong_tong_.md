# win10 uwp App-to-app communication 应用通信

这篇文章都是乱说的，如果觉得有不好的，可以发我邮箱

本文主要讲如何让两个应用之间传输消息，也就是我们经常用的分享。我们可以使用的有剪辑版、UWP分享、Uri启动应用多个方式。

![这里写图片描述](http://img.blog.csdn.net/20160404102715815)

如果有个人看到一个网页很好，于是就希望把这个网页发送到邮件，那么这样的话就是使用应用通信。

因为每个应用都是不能访问其他应用数据，所以需要通信可以使用启动内置应用，文件关联应用。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


<div id="toc"></div>

## 发送数据

创建一个event 可以在用户发送，共享发送

```csharp
            DataTransferManager data_transfer_manager = DataTransferManager.GetForCurrentView();
            data_transfer_manager.DataRequested += DataTransferManager_DataRequested;
```
当DataRequested，应用收到一个DataRequest，这个是DataPackage可以在里面写你要发送的信息。DataPackage必须写标题和数据，如果有描述也写

```csharp
        private static void DataTransferManager_DataRequested(DataTransferManager sender, DataRequestedEventArgs args)
        {
            DataRequest request = args.Request;
        }
```

可以共享数据：

 - 纯文本
 - url
 - HTML
 - 文本
 - 图片
 - 文件
 - 自己弄的我也不知道是什么的可以共享的

```csharp
            //文本
            request.Data.SetText(text);
            //uri
            //request.Data.SetUri(uri);过时
            request.Data.SetWebLink(uri);    
            //html                 
            request.Data.SetHtmlFormat(html);
            request.Data.SetRtf(text);
            //文件
            request.Data.SetStorageItems(file);
            //图片
            request.Data.SetBitmap(bitmap);
```

我们需要和用户说我们在做的数据

```csharp
            request.Data.Properties.Title = "标题";
            request.Data.Properties.Description = "我的博客blog.csdn.net/lindexi_gd";
```
![这里写图片描述](http://img.blog.csdn.net/20160404105455138)

开始通信

```csharp
DataTransferManager.ShowShareUI();
```
有时候我们需要等待一些操作需要时间，不能马上就分享，我们可以使用

```csharp
            request.Data.Properties.Title = "标题";
            request.Data.Properties.Description = "我的博客blog.csdn.net/lindexi_gd";

            request.Data.SetDataProvider(StandardDataFormats.Bitmap, (data_provider_request) =>
            {
                DataProviderDeferral deferral = data_provider_request.GetDeferral();
                //做时间比较长的操作
                //一般可以把操作内容放try，因为操作内容主要是io，有出错
                //如果放在try，把deferral.Complete();finally
                //try
                //{
                //    //操作
                //}
                //finally
                //{
                //    //deferral.Complete();
                //}
                deferral.Complete();
            });
```



要接受其他的app我们需要设置`requestData.Properties.ContentSourceApplicationLink = ApplicationLink;`
ApplicationLink是`new Uri("ms-sdk-sharesourcecs:navigate?page=" + 页面名);`

要接受其他的app我们需要设置

![](http://image.acmx.xyz/16-4-5/70888377.jpg)

![](http://image.acmx.xyz/16-4-5/65763757.jpg)
我们在说明写：林德熙博客

但说明其实没有什么用，主要是数据格式才是需要我们选择，在上也看到我们可以分享的数据有多种格式，那么满足格式的分享就会在分享看到我们的应用。

![](http://image.acmx.xyz/16-4-5/25742257.jpg)
新建一个页面接分享，因为我想不到这个叫什么，我就放在MainPage

导航到MainPage就是分享打开

页面传参数可以使用，`Frame frame.Navigate`(页面，参数)


```csharp
        protected override void OnNavigatedTo(NavigationEventArgs e)
        {

        }
```

在App.xaml.cs

```csharp
        protected override void OnShareTargetActivated(ShareTargetActivatedEventArgs args)
        {
            Frame rootFrame = Window.Current.Content as Frame;
            if (rootFrame == null)
            {
                rootFrame=new Frame();
                Window.Current.Content = rootFrame;//http://blog.csdn.net/lindexi_gd
            }
            rootFrame.Navigate(typeof (MainPage), args.ShareOperation);
            Window.Current.Activate();
        }
```
我们可以在OnNavigatedTo拿分享

```csharp
        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            ShareOperation share_operation = e.Parameter as ShareOperation;
            if (share_operation == null)
            {
                return;
            }
            //标题
            string shared_data_title = share_operation.Data.Properties.Title;
            string shared_data_description = share_operation.Data.Properties.Description;
            Uri url = share_operation.Data.Properties.ContentSourceWebLink;
            Uri application_link = share_operation.Data.Properties.ContentSourceApplicationLink;
            //图像
            RandomAccessStreamReference thumbnail = share_operation.Data.Properties.Thumbnail;
            //应用名称
            string application_name = share_operation.Data.Properties.ApplicationName;
            //数据
            //判断存在，如果不存在我们
            if (share_operation.Data.Contains(StandardDataFormats.WebLink))
            {
                Uri web_link =await share_operation.Data.GetWebLinkAsync();
            }
        }
```

当我们做完可以告诉` share_operation.ReportCompleted();`

如果错了可以告诉发送我们接受错

分享成功经常返回一个链接，我们把一个东西分享到百度云，那么我们可以拿到一个链接百度云，可以发送，这个`QuickLink`


`QuickLink `·我们需要标题，图标，id

```csharp
            QuickLink quickLinkInfo = new QuickLink()
            {
                Id = QuickLinkId,
                Title = QuickLinkTitle,
                SupportedFileTypes = { "*" },
                SupportedDataFormats =
                    {
                        StandardDataFormats.Text,
                        StandardDataFormats.WebLink,
                        StandardDataFormats.ApplicationLink,
                        StandardDataFormats.Bitmap,//http://blog.csdn.net/lindexi_gd
                        StandardDataFormats.StorageItems,
                        StandardDataFormats.Html
                    },
                Thumbnail = thumbnail,
            };
            share_operation.ReportCompleted(quickLinkInfo);
```
## 文件启动

我们需要关联
![这里写图片描述](http://img.blog.csdn.net/20160405185522977)
在app.xaml.cs

```csharp
        protected override void OnFileActivated(FileActivatedEventArgs args)
        {
           // args.Files
        }
```
Files包含文件可以拿来

博客：http://blog.csdn.net/lindexi_gd

原文：https://msdn.microsoft.com/en-us/windows/uwp/app-to-app/index

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


