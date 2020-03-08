# win10 uwp 进度条 WaveProgressControl

昨天看到了有个大神做出好看的进度条样式，于是我就去抄袭他的代码，但是发现看不懂，于是本文主要翻译就是大神说这个控件如何做。

![](http://image.acmx.xyz/0f822922-f86b-98e3-4682-30bbe3160e6a%2Fenyb.gif)

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


本文翻译 https://stackoverflow.com/a/46057193/6116637 来这 liu xin 大神的控件。

上面的控件实际就是两个圆，然后 Compositor 让背景显示在里面的圆。因为可以使用下面图片的方式，看起来就是从一个圆里出现背景。实际就是背景移动图片，可以看到图片移动的时候，看里面的圆的背景，就是上面那张图的样子。

![](http://image.acmx.xyz/0f822922-f86b-98e3-4682-30bbe3160e6a%2Fbrmx.gif)

也就是在图片的上移就是进度，可以用 Percent 来知道现在的进度，然后计算显示的高度，很容易就计算出上移。然后图片可以通过 Adobe Illustrator 工具来做，打开 Zig Zag 效果就可以做出这个图片。

![](http://image.acmx.xyz/0f822922-f86b-98e3-4682-30bbe3160e6a%2F201791016335.jpg)

注意图片从左到右播放再重新播放，看起来不会出现断的图片。

下面就是代码，如果现在 UWP 可以做出随意裁剪，就不需要使用 Compositor 为了使用 Compositor 需要使用字段 Compositor ，而且需要一个 double 的属性，用于做进度。

因为使用 LoadedImageSurface 下面的代码需要在 15063 才可以跑，如果你的代码是跑在 14393 那么无法使用。

界面代码


```csharp
<UserControl x:Class="WaveProgressControlRepo.WaveProgressControl"
             Height="160"
             Width="160">

    <Grid x:Name="Root">
        <Ellipse x:Name="ClippedImageContainer"
                 Fill="White"
                 Margin="6" /> 这个圆白色，里面背景就是放图片
        <Ellipse x:Name="CircleBorder"
                 Stroke="#FF0289CD"
                 StrokeThickness="3" />
        <TextBlock Foreground="#FF0289CD"
                   FontSize="36"
                   FontWeight="SemiBold"
                   TextAlignment="Right"
                   VerticalAlignment="Center"
                   Width="83"
                   Margin="0,0,12,0">
                   显示现在进度
            <Run Text="{x:Bind Percent, Mode=OneWay}" />
            <Run Text="%"
                 FontSize="22" />
        </TextBlock>
    </Grid>
</UserControl> 
```


```csharp
 using System;
using System.Numerics;
using Windows.UI.Composition;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Hosting;
using Windows.UI.Xaml.Media;

namespace WaveProgressControlRepo
{
	public sealed partial class WaveProgressControl : UserControl
	{
		private readonly Compositor _compositor;
		private readonly CompositionPropertySet _percentPropertySet;

		public WaveProgressControl()
		{
			InitializeComponent();

			_compositor = Window.Current.Compositor;

			_percentPropertySet = _compositor.CreatePropertySet();
			_percentPropertySet.InsertScalar("Value", 0.0f);

			Loaded += OnLoaded;
		}

		public double Percent
		{
			get => (double)GetValue(PercentProperty);
			set => SetValue(PercentProperty, value);
		}
		public static readonly DependencyProperty PercentProperty =
			DependencyProperty.Register("Percent", typeof(double), typeof(WaveProgressControl),
				new PropertyMetadata(0.0d, (s, e) =>
				{
					var self = (WaveProgressControl)s;
					var propertySet = self._percentPropertySet;
					propertySet.InsertScalar("Value", Convert.ToSingle(e.NewValue) / 100);
				}));

		private void OnLoaded(object sender, RoutedEventArgs e)
		{
			CompositionSurfaceBrush imageSurfaceBrush;

			SetupClippedWaveImage();//裁剪图片，显示圆
			SetupEndlessWaveAnimationOnXAxis();//图片从左到右，这样看起来就不会断
			SetupExpressionAnimationOnYAxisBasedOnPercentValue();//如果进度修改了，那么移动图片

            //把背景设置到控件
			void SetupClippedWaveImage()//
            {
				// Note LoadedImageSurface is only available in 15063 onward.
				var imageSurface = LoadedImageSurface.StartLoadFromUri(new Uri(BaseUri, "ms-appx:///Assets/wave.png"));
                //LoadedImageSurface 在 15063 所以如果代码在 14393 无法使用
                imageSurfaceBrush = _compositor.CreateSurfaceBrush(imageSurface);
				imageSurfaceBrush.Stretch = CompositionStretch.None;
				imageSurfaceBrush.Offset = new Vector2(120, 248);

				var maskBrush = _compositor.CreateMaskBrush();
				var maskSurfaceBrush = ClippedImageContainer.GetAlphaMask(); // CompositionSurfaceBrush
				maskBrush.Mask = maskSurfaceBrush;
				maskBrush.Source = imageSurfaceBrush;

				var imageVisual = _compositor.CreateSpriteVisual();
				imageVisual.RelativeSizeAdjustment = Vector2.One;
				ElementCompositionPreview.SetElementChildVisual(ClippedImageContainer, imageVisual);

				imageVisual.Brush = maskBrush;
			}

			void SetupEndlessWaveAnimationOnXAxis()
			{
                //水平动画
				var waveOffsetXAnimation = _compositor.CreateScalarKeyFrameAnimation();
				waveOffsetXAnimation.InsertKeyFrame(1.0f, -80.0f, _compositor.CreateLinearEasingFunction());
				waveOffsetXAnimation.Duration = TimeSpan.FromSeconds(1);//一秒重复一次
				waveOffsetXAnimation.IterationBehavior = AnimationIterationBehavior.Forever;
				imageSurfaceBrush.StartAnimation("Offset.X", waveOffsetXAnimation);
			}

			void SetupExpressionAnimationOnYAxisBasedOnPercentValue()
			{
                //_percentPropertySet 可以拿到 进度 变化，移动背景
                var waveOffsetYExpressionAnimation = _compositor.CreateExpressionAnimation("Lerp(248.0f, 120.0f, Percent.Value)");
				waveOffsetYExpressionAnimation.SetReferenceParameter("Percent", _percentPropertySet);
				imageSurfaceBrush.StartAnimation("Offset.Y", waveOffsetYExpressionAnimation);
			}
		}
	}
}

```

如果觉得上面的代码还是不懂，那么从 github 下载代码来运行 https://github.com/JustinXinLiu/WaveProgressControlRepo

