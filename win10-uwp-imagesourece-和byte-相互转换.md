# win10 uwp ImageSourece 和Byte[] 相互转换

<!--more-->

我们先写一个简单的xaml

```
       <Image x:Name="Img" Height="200" Width="200" 
              HorizontalAlignment="Center" Source="Assets/SplashScreen.png" ></Image>
        
        <Button Margin="10,300,10,10" Content="确定" Click="Button_OnClick" ></Button>
   
```

![](http://7xqpl8.com1.z0.glb.clouddn.com/fc7733af-8526-44d2-84b9-99b41ef99f4a2016121293717.jpg)

## 保存WriteableBitmap到文件





## ImageSource 转byte[]

ImageSource可以转为WriteableBitmap

## BitmapImage 转 WriteableBitmap

我使用http://www.cnblogs.com/cjw1115/p/5164327.html 大神的，直接转`WriteableBitmap bitmap = imageSource as WriteableBitmap;`bitmap为null，于是我在网上继续找。




如果需要保存网络图片到本地，请到[win10 uwp 存放网络图片到本地](http://lindexi.oschina.io/lindexi/post/win10-uwp-存放网络图片到本地/)

参见：http://www.cnblogs.com/cjw1115/p/5164327.html

http://www.cnblogs.com/yuanforprogram/p/4819307.html