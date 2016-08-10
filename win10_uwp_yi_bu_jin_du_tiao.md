# win10 uwp 异步进度条

 进度条可以参见：http://edi.wang/post/2016/2/25/windows-10-uwp-modal-progress-dialog
 
 进度条其实异步就是使用后台变化，然后value绑定
 
 
 ##圆形进度条
 
 参见：http://www.cnblogs.com/ms-uap/p/4641419.html
 
 先说怎么用我的，首先去我源代码https://github.com/lindexi/UWP，打开我的进度条文件夹，里面
 有View文件夹
 
 我在View有一个控件`RountProgress`复制他到你的解决方案，如果我的控件大小和你不一样，很简单调整，我就不说。
 
 那么我的控件只需要指定Value就好啦，Value其实是从0到100，如果叫别的应该好，但是我不改，如果你觉得不想要，自己改
 
 ```
 
     xmlns:view="using:lindexi.uwp.control.RountProgress.View"

     <view:RountProgress Value="{x:Bind Value,Mode=OneWay}"></view:RountProgress>
```

 ![这里写图片描述](http://img.blog.csdn.net/20160810164207135)
 
 我来说下怎么做
 
 
 
 