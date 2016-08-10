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
 
 我们要知道StrokeDashArray，这个是一个数组，是循环的，也就是依此读取，知道超过长度。
 
 首先我们需要有Thickness，宽度，StrokeDashArray的每一个都是宽度的倍数
 
 首先取第一个元素，把这个元素乘以宽度，作为显示的大小，然后取第二个元素，乘以宽度，作为不显示的大小
 
 然后循环获取第三个……，如果不存在第三个，那么循环拿第一做第三，n=n==max?0:n+1，n就是第n个元素
 
 一个显示一个不显示，循环
 
 记得长度乘以是`值*宽度`
 
 那么我们如果有一个`值*宽度`的到大小比我们的宽度还大，那么就会截断。
 
 加入我们宽度 3，StrokeDashArray 1,2,0.5，总长度为5，那么
 
 
 第一个是大小 `1*3`显示，然后是`2*3`不显示，因为到第一个只有长度为2，第二个大小为6，所以会截断，3显示然后2不显示
 
 
 
 