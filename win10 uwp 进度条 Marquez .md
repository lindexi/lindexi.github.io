# win10 uwp 进度条 Marquez 

本文将告诉大家，如何做一个带文字的进度条，这个进度条可以用在游戏，现在我做的挂机游戏就使用了他。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2FMarquez.gif)

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


<div id="toc"></div>

如何做上图的效果，实际需要的是两个控件，一个是显示文字 的 TextBlock  一个是进度条。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201764111545.jpg)

那么如何让 文字和左边的距离变化？使用 TranslateTransform

看起来 Marquez 的界面就是：


```csharp
         <ProgressBar x:Name="Mcdon" Maximum="100" Minimum="0" Value="20"
                     VerticalAlignment="Stretch"></ProgressBar>
        <TextBlock x:Name="scrohn" Text="100/100"
                   VerticalAlignment="Center">
            <TextBlock.RenderTransform>
                <TranslateTransform  X="0"></TranslateTransform>
            </TextBlock.RenderTransform>
        </TextBlock>
```

进度条的名字就是 Marquez ，写完界面，后台也不难

需要使用几个依赖属性设置最大值、当前值、最小值


```csharp
    
        /// <summary>
        ///     标识 <see cref="Maximum" /> 的依赖项属性。
        /// </summary>
        public static readonly DependencyProperty MaximumProperty = DependencyProperty.Register(
            "Maximum", typeof(double), typeof(Marquez), new PropertyMetadata(100d, (s, e) =>
            {
                var t = s as Marquez;
                if (t == null)
                {
                    return;
                }

                Scrhrentran(t.scrohn, t.ActualWidth, t.Value, (double) e.NewValue, t.Mcdon);
            }));


        /// <summary>
        ///     标识 <see cref="Minimum" /> 的依赖项属性。
        /// </summary>
        public static readonly DependencyProperty MinimumProperty = DependencyProperty.Register(
            "Minimum", typeof(double), typeof(Marquez), new PropertyMetadata(default(double)));


        /// <summary>
        ///     标识 <see cref="Value" /> 的依赖项属性。
        /// </summary>
        public static readonly DependencyProperty ValueProperty = DependencyProperty.Register(
            "Value", typeof(double), typeof(Marquez), new PropertyMetadata(20d, (s, e) =>
            {
                var t = s as Marquez;
                if (t == null)
                {
                    return;
                }

                Scrhrentran(t.scrohn, t.ActualWidth, (double) e.NewValue, t.Maximum, t.Mcdon);
            }));

        /// <summary>
        ///     获取或设置
        /// </summary>
        public double Value
        {
            get { return (double) GetValue(ValueProperty); }
            set { SetValue(ValueProperty, value); }
        }

        /// <summary>
        ///     获取或设置最小值
        /// </summary>
        public double Minimum
        {
            get { return (double) GetValue(MinimumProperty); }
            set { SetValue(MinimumProperty, value); }
        }

        /// <summary>
        ///     获取或设置最大值
        /// </summary>
        public double Maximum
        {
            get { return (double) GetValue(MaximumProperty); }
            set { SetValue(MaximumProperty, value); }
        }
```

所有值变化时，需要修改文字和进度条，因为进度条没有绑定值到代码，Scrhrentran 函数修改所有值。

为什么不使用绑定，因为绑定容易重复，而且有些值不是简单绑定就可以，这个控件使用绑定还是可以做到，如果自己感兴趣，可以修改他绑定。

从属性可以看到，值变化自动调用 Scrhrentran 于是函数需要修改进度条的值，修改进度条很简单，只需要使用下面代码

```csharp
            private static void Scrhrentran(TextBlock scrohn, double w, double v, double t, ProgressBar mcdon)
            {
                        mcdon.Value = v;
                        mcdon.Maximum = t;
            }
```
可以看到，上面的代码没修改最小值，因为最小值没有在依赖属性写，我不写最小值因为我想讲下如何获得依赖属性修改。

依赖属性是很好用的，他自己就带了绑定，如果想用绑定，那么可以使用依赖属性，依赖属性可以使用 dep 和tab打出来，一般的依赖属性是比较长的，最小值用的就是 vs 自带的依赖属性，也就是经常这样写。


```csharp

        /// <summary>
        ///     标识 Minimum 的依赖项属性。
        /// </summary>
        public static readonly DependencyProperty MinimumProperty = DependencyProperty.Register(
            "Minimum", typeof(double), typeof(Marquez), new PropertyMetadata(default(double)));

         /// <summary>
        ///     获取或设置最小值
        /// </summary>
        public double Minimum
        {
            get { return (double) GetValue(MinimumProperty); }
            set { SetValue(MinimumProperty, value); }
        }
```
实际依赖属性是上面的静态属性，他使用了注册，注册的第一个参数表示变量的名字，因为是自己生成的，就是字符串，但是字符串有问题，如果我修改了 Minimum 名称，那么字符串就无法使用，为了在修改名称可以使用，我建议使用 nameof 这个可以获得变量名称。

其中第二个参数是 类型，第三个是类，这个参数指定是哪个类，如果复制了别人的 依赖属性，容易出错，因为他的类没有修改为自己的类。最后一个属性是指定默认值，在这个属性可以指定属性修改时的函数。


```csharp
            public static readonly DependencyProperty MinimumProperty = DependencyProperty.Register(
            "Minimum", typeof(double), typeof(Marquez), new PropertyMetadata(default(double), (s, e) =>
            {
                
            } ));
```
现在就可以在里面写属性修改的函数，第一个参数 s 是哪个触发，也就是 Marquez ，使用第一个参数就可以获得 Marquez，第二个参数是获得之前的值和当前的值，通过`e.NewValue`可以获得修改后的值。

但是不可以通过这个函数修改 `e.NewValue` 的值。

于是这个控件比较难的地方就是修改文字，下面来开始做这部分。

显示文字可以使用下面代码

```csharp
                scrohn.Text = v.ToString("F") + "/" + t.ToString("F");

```
可以看到，只看代码是不知道 v 是什么， t 是什么，所以在命名时最好不要这样写，建议写为 value 和 maximum，这样看代码就可以知道两个值。

修改文字之前，判断RenderTransform


```csharp
                var sc = scrohn.RenderTransform as TranslateTransform;

```

在value为最大值，文字显示在中间，于是文字最大的就是 ` w / 2` ，w就是控件宽度。但是还需要乘以现在的 `v / t`

于是算法就是 `sc.X = w / 2 * v / t` ，但是因为文字有宽度，显示的是文字左边，所以需要减去文字，但是可能让文字在控件看不到，因为`sc.X < 0`，于是代码就是


```csharp
                 sc.X = w / 2 * v / t - scrohn.ActualWidth / 2;
                if (sc.X < 0)
                {
                    sc.X = 0;
                }
```

总的代码放在github：https://github.com/lindexi/UWP/tree/master/uwp/control/Progress

如果 想写一个控件，建议先在我的库找找，可能我做了，所以可以让你省点时间。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


