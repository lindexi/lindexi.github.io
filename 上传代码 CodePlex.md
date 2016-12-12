# 上传代码 CodePlex

CodePlex是微软开源项目网站，有很多人都在上面传代码，我们也可以上传自己的代码


<!--more-->


## 注册

我们可以用微软账号注册，填写用户名、密码，很快就好。

## 新建项目

去https://www.codeplex.com/ 选`Creat Project`

![](http://7xqpl8.com1.z0.glb.clouddn.com/fc7733af-8526-44d2-84b9-99b41ef99f4a20161212135425.jpg)

填写标题、域名、选择git上传

![](http://7xqpl8.com1.z0.glb.clouddn.com/fc7733af-8526-44d2-84b9-99b41ef99f4a20161212135510.jpg)

创建

## 上传代码

点击 `Source Code`

![](http://7xqpl8.com1.z0.glb.clouddn.com/fc7733af-8526-44d2-84b9-99b41ef99f4a20161212135552.jpg)

点击`Clone`

就得到Git地址，在自己项目，上传

		
```
git remote add codeplex https://git01.codeplex.com/lindexiuwp   这里的地址是你自己地址

git push codeplex master

```

就可以在代码看到

## Page

我们需要让别人知道我们项目，需要写一个Page，他的语法垃圾，我不想说，大家自己去学。
		
```
! Uwp

常用代码和控件

!! 控件

 - [url:白天黑夜按钮|https://github.com/lindexi/UWP/blob/master/uwp/control/NightDayThemeToggleButton]

 [image:http://7xqpl8.com1.z0.glb.clouddn.com/fc7733af-8526-44d2-84b9-99b41ef99f4a20161212105727.jpg]

 - [url:进度条|https://github.com/lindexi/UWP/blob/master/uwp/control/Progress]

 [image:http://img.blog.csdn.net/20160815151046014]

 - [url:变大数字颜色按钮|https://github.com/lindexi/UWP/blob/master/uwp/control/RountGradualFigure]

 [image:http://7xqpl8.com1.z0.glb.clouddn.com/RountGradual.gif]

!!! 图

 - DataGrid（没做）

!! 软件

 - [url:win10 uwp 水印图床|https://github.com/lindexi/UWP/blob/master/uwp/control/BitStamp]

   参见 ：[url:win10 uwp 水印图床|https://github.com/lindexi/UWP/blob/master/uwp/control/BitStamp/%E3%80%90%E5%B9%BF%E5%91%8A%E3%80%91win10%20uwp%20%E6%B0%B4%E5%8D%B0%E5%9B%BE%E5%BA%8A%20%E5%90%AB%E4%BB%A3%E7%A0%81.md]

   [url:安装|ms-windows-store://pdp/?productid=9nblggh562r2]https://www.microsoft.com/store/apps/9nblggh562r2

!! 代码

 - [url:DetailMaster|https://github.com/lindexi/UWP/blob/master/uwp/src/DetailMaster]

 [image:http://img.blog.csdn.net/20160806130438076]

 - [url:图床|https://github.com/lindexi/UWP/blob/master/uwp/src/Imageshack]

   图床是把图片上传到云，然后获取图片链接的开发包，我将繁琐的过程写成一个简单的类。
   上传的服务器现在有[url:sm.ms|https://sm.ms/]和[url:七牛图床|http://www.qiniu.com/]。其中[url:七牛sdk UWP|https://github.com/lindexi/UWP/blob/master/uwp/src/Imageshack/cloundes]，
   我只有简单文件上传，好多还没写。代码是从其他大神改出

   七牛图床上传到Nuget，搜索`lindexi.uwp.ImageShack.Thirdqiniucs`或
   控制台`Install-Package lindexi.uwp.ImageShack.Thirdqiniucs`


 - [url:显示svg|https://github.com/lindexi/UWP/blob/master/uwp/src/ScalableVectorGraphic]

 - [url:SplitView|https://github.com/lindexi/UWP/blob/master/uwp/src/SplitView]
   
   汉堡菜单

 - [url:ViewModel|https://github.com/lindexi/UWP/blob/master/uwp/src/ViewModel]

 - [url:隐私策略|https://github.com/lindexi/UWP/blob/master/uwp/src/%E9%9A%90%E7%A7%81%E7%AD%96%E7%95%A5]

 - [url:径向规|https://github.com/lindexi/UWP/blob/master/uwp/src/RadialGauge]

 - 图片存放本地
   
   输入Uri打开，第一次从网络打开，之后在本地打开。

   先判断本地存在图片，不存在就从网络下载

   `BitmapImage img = await ImageStorage.GetImage(uri);`

   上传到Nuget，可以搜索`lindexi.uwp.src.ImageStorage `或控制台
   `Install-Package lindexi.uwp.src.ImageStorage`

! English

Some controls and common codes

```

http://www.cnblogs.com/gossip/archive/2012/06/26/2563587.html 

参见 http://www.cnblogs.com/aspnet_csharp/archive/2012/04/20/2459099.html

