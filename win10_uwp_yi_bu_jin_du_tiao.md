# win10 uwp 异步进度条

 进度条可以参见：http://edi.wang/post/2016/2/25/windows-10-uwp-modal-progress-dialog
 
 进度条其实异步就是使用后台变化，然后value绑定
 
 我使用一个
 
 ```
         <ProgressBar Maximum="100" Value="{x:Bind View.Value,Mode=OneWay}" Height="20" Width="100"></ProgressBar>
```

绑定到我们的ViewModel，一般如果后台线程操作界面是不能直接，但是我用了

 ```
             await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal,
                () =>
                {
                     
                });
```
 
代码参见：https://github.com/lindexi/UWP，项目所有代码都会发出

我们使用Task异步，我们因为没有什么耗时的，就`Task.Delay(1000).Wait();`

ViewModel

```
        public ViewModel()
        {
            new Task(() =>
            {
                while (Value < 90)
                {
                    Value += 10;
                    Task.Delay(1000).Wait();
                }
            }).Start();
        }

        public double Value
        {
            set
            {
                _value = value;
                OnPropertyChanged();
            }
            get
            {
                return _value;
            }
        }

        private double _value;
```

我还自己的控件，一个值从0到100的圆形的，可以看下面
 
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
 
 我们可以用第一个为一个值，然后第二个为一个比总长度还大的值，这样会让宽度显示为我们第一个的值，而其他为空，因为第二个比最大还大
 
 我们要做一个`30%`，我们需要算
 
 `长=圆*30%/宽度`
 
 `圆=PI*(总长度-宽度)`
 
 ```
         <Ellipse x:Name="Rount" Stroke="DeepSkyBlue" Height="100" Width="100" 
                 StrokeThickness="3" 
                 RenderTransformOrigin="0.5,0.5"/>
                 
```

那么我们第一个值 (总长度100 - 宽度3) \* PI / 宽度3

因为我们需要算我们的宽度不是直接总长度，是总长度-宽度

第二个最好是Double.Max

我们想要一个可以用户进度，那么可以绑定一个属性，在我们控件

我们需要这个为double，然后绑定

因为我们需要两个值，所以转换

假如我们的转换是固定的总长度，宽度，那么可以使用

```

        public object Convert(object value, Type targetType, object parameter, string language)
        {
            double thine = 3;
            double w = 100 - thine;
            double n = Math.PI * w/thine * (double)value / 100;
            DoubleCollection temp = new DoubleCollection()
            {
               n,
                1000
            };

            return temp;
        }
```

如果觉得固定不好，可以在我们转换写属性，然后在界面把我们的宽度给属性，然后换为我们的宽度算，这个简单

代码在https://github.com/lindexi/UWP/tree/master/uwp/control/Progress/Progress/View/RountProgress.xaml

那么进度条如果不需要进度，那么我有一些好的，例如我之前的博客有说的，还有一个简单，也是上面改，我们一个值是显示一个值是不显示，那么我们可以做

![这里写图片描述](http://img.blog.csdn.net/20160810190545145)

```
<UserControl
    x:Class="lindexi.uwp.control.RountProgress.View.IndeterminateProgress"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:lindexi.uwp.control.RountProgress.View"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    d:DesignHeight="300"
    d:DesignWidth="400">

    <UserControl.Resources>

        <Style TargetType="ProgressRing">
            <Setter Property="Background" Value="Transparent"/>
            <Setter Property="Foreground" Value="{ThemeResource SystemControlHighlightAccentBrush}"/>
            <Setter Property="IsHitTestVisible" Value="False"/>
            <Setter Property="HorizontalAlignment" Value="Center"/>
            <Setter Property="VerticalAlignment" Value="Center"/>
            <Setter Property="MinHeight" Value="20"/>
            <Setter Property="MinWidth" Value="20"/>
            <Setter Property="IsTabStop" Value="False"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="ProgressRing">
                        <Grid x:Name="Ring" BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}" Background="{TemplateBinding Background}" FlowDirection="LeftToRight" MaxWidth="{Binding TemplateSettings.MaxSideLength, RelativeSource={RelativeSource Mode=TemplatedParent}}" MaxHeight="{Binding TemplateSettings.MaxSideLength, RelativeSource={RelativeSource Mode=TemplatedParent}}" Padding="{TemplateBinding Padding}" RenderTransformOrigin=".5,.5" >
                            <Grid.Resources>
                                <Style x:Key="ProgressRingEllipseStyle" TargetType="Ellipse">
                                    <Setter Property="Opacity" Value="0"/>
                                    <Setter Property="HorizontalAlignment" Value="Left"/>
                                    <Setter Property="VerticalAlignment" Value="Top"/>
                                </Style>
                            </Grid.Resources>
                            <VisualStateManager.VisualStateGroups>
                                <VisualStateGroup x:Name="SizeStates">
                                    <VisualState x:Name="Large">
                                        <Storyboard>
                                      
                                        </Storyboard>
                                    </VisualState>
                                    <VisualState x:Name="Small"/>
                                </VisualStateGroup>
                                <VisualStateGroup x:Name="ActiveStates">
                                    <VisualState x:Name="Inactive"/>
                                    <VisualState x:Name="Active">
                                        <Storyboard RepeatBehavior="Forever">
                                            <DoubleAnimation Storyboard.TargetName="Rount" Storyboard.TargetProperty="Angle"
                                                           BeginTime="0:0:0" Duration="0:0:5" From="0" To="360"  >

                                            </DoubleAnimation>
                                            
                                        </Storyboard>
                                    </VisualState>
                                </VisualStateGroup>
                            </VisualStateManager.VisualStateGroups>
                           
                            <Ellipse  Stroke="DeepSkyBlue" Height="100" Width="100" 
                                      StrokeThickness="3"  
                                      RenderTransformOrigin="0.5,0.5"/>
                            <Ellipse  Stroke="DeepSkyBlue" Height="200" Width="200" 
                                      StrokeThickness="3" StrokeDashArray="50 50" 
                                      RenderTransformOrigin="0.5,0.5" >
                                <Ellipse.RenderTransform>
                                    <RotateTransform  x:Name="Rount" Angle="0"/>
                                </Ellipse.RenderTransform>
                            </Ellipse>
                        </Grid>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>

    </UserControl.Resources>
    <Grid>
        <ProgressRing Width="200" Height="200" 
                      IsActive="True"></ProgressRing>
    </Grid>

</UserControl>
```

我们使用一个简单的修改，因为我们可以使用`<RotateTransform  x:Name="Rount" Angle="0"/>`

我们使用

```
                                    <VisualState x:Name="Active">
                                        <Storyboard RepeatBehavior="Forever">
                                            <DoubleAnimation Storyboard.TargetName="Rount" Storyboard.TargetProperty="Angle"
                                                             Duration="0:0:5" From="0" To="360"  >

                                            </DoubleAnimation>
                                            
                                        </Storyboard>
                                    </VisualState>
```



修改我们旋转，时间0:0:5，5秒，从0到360，循环

因为是修改，所以可以放在Resource

```
<ProgressRing Width="200" Height="200" 
                      IsActive="True"></ProgressRing>
 ```

我觉得匀速不好，修改速度

- BackEase

 缓动函数，它在部分持续时间内向反方向更改主函数的值
 
- BounceEase
 
 弹跳

- CircleEase 

  加速
  
- PowerEase 

  次方
  
- SineEase 

  sin加速
  
- QuadraticEase 

  `^2`

 
 ##动画
 
 移动元素
 
 我们可以看到我们的元素位置可以修改Margin，那么如何在动画修改Margin
 
 UWP动画Margin可以
 
 ```
 <Storyboard TargetName="Rount">
    <ObjectAnimationUsingKeyFrames Storyboard.TargetProperty="Margin"
                                   BeginTime="00:00:00" EnableDependentAnimation="True"
                                   Duration="0:0:2" >
                 <DiscreteObjectKeyFrame KeyTime="00:00:00"  >
                     <DiscreteObjectKeyFrame.Value >
                         <Thickness>10,1,10,10</Thickness>
                     </DiscreteObjectKeyFrame.Value>
                   </DiscreteObjectKeyFrame>
                 <DiscreteObjectKeyFrame KeyTime="00:00:02">
                     <DiscreteObjectKeyFrame.Value >
                         <Thickness>10,200,10,10</Thickness>
                 </DiscreteObjectKeyFrame.Value>
         </DiscreteObjectKeyFrame>
    </ObjectAnimationUsingKeyFrames>
 </Storyboard>
```
Rount就是我们要修改的控件，我们看到这是在2就直接修改，没有从1到200，这样其实并不是我们直接就想从1然后两秒200

我们定义

                                <local:IndeterminateProgress  Margin="0,10,0,0" Width="200" Height="200" >
                                    <local:IndeterminateProgress.RenderTransform>
                                        <TranslateTransform x:Name="Rount" Y="0"></TranslateTransform>
                                    </local:IndeterminateProgress.RenderTransform>
                                </local:IndeterminateProgress>
                                
```                                
 <DoubleAnimation Storyboard.TargetName="Rount" Storyboard.TargetProperty="Y"
        From="0" To="100" Duration="0:0:2"></DoubleAnimation>
```

我们要让我们的进度弹起来，如果不知道我说什么，简单我有图

![这里写图片描述](http://img.blog.csdn.net/20160815151046014) 

其实我们要让我们的元素移动，可以看林政大神的书

```
 
    <local:IndeterminateProgress Margin="0,10,0,0" Width="200" Height="200" >
       <local:IndeterminateProgress.RenderTransform>
               <TranslateTransform x:Name="Rount" Y="10" />
       </local:IndeterminateProgress.RenderTransform>
    </local:IndeterminateProgress>

```

在动画

```

                                              <DoubleAnimation Storyboard.TargetName="Rount"
                                                             Storyboard.TargetProperty="Y"
                                                             Duration="0:0:2" From="0" To="300">
                                                <DoubleAnimation.EasingFunction>
                                                    <BounceEase Bounces="2"></BounceEase>
                                                </DoubleAnimation.EasingFunction>
                                            </DoubleAnimation>

```

我们使用Rount，x，记得要给名字，然后两秒，从0到300，下面就是弹跳，我上面有说，这个在官方有说比我写还好，但是官方的我没法拿来




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。