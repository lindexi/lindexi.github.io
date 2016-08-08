# win10 uwp 简单MasterDetail

本文主要讲实现一个简单的界面，可以在窗口比较大显示列表和内容，窗口比较小时候显示列表或内容。也就是在窗口比较小的时候，点击列表会显示内容，点击返回会显示列表。

先放图，很简单。

开始的窗口是大，显示列表，因为开始没有点击列表就显示图片，点击列表显示内容，就是下面的图。

![这里写图片描述](http://img.blog.csdn.net/20160806130421310)

![这里写图片描述](http://img.blog.csdn.net/20160806130438076)

如果屏幕小，那么显示列表或内容

当然可以看下垃圾wr的

![这里写图片描述](https://msdn.microsoft.com/zh-cn/windows/uwp/controls-and-patterns/images/patterns-md-stacked.png)

我的

![这里写图片描述](http://img.blog.csdn.net/20160806131345316)

![这里写图片描述](http://img.blog.csdn.net/20160806131357113)

https://msdn.microsoft.com/windows/uwp/controls-and-patterns/master-details

国内晓迪文章很好，但是对我渣渣很难。

本文是很简单的，一般和我一样渣都能大概知道。

我在很大的压力会议写的，不到一个钟，写完修改，和大家说，可以修改我代码，可以自己写

我们首先一个Grid，分为两栏，其中一栏为List，一栏为Content

在大屏，也就是我们可以把Grid两栏显示，基本就是Frame导航就好了。

如果屏幕小，我们合并为一个Grid，使用顺序，对，List和Content的Zindex设置他们的位置，Zindex比较大的会显示，也就是判断是否存在Content，存在就显示他，不存在，显示List。

应该可以看懂。

现在来说Frame导航。

##UWP 导航

Content是一个Frame和一个Image的Grid

```
            <Grid Grid.Column="{x:Bind View.GridInt,Mode=OneWay}" x:Name="Img" 
                  Canvas.ZIndex="{x:Bind View.ZFrame,Mode=OneWay}">
                <Image  Source="../Assets/images.jpg"
                       ></Image>
                <Frame x:Name="frame"
                       ></Frame>
            </Grid>
```

先不要Grid的属性，我会在后面说。

我们没Frame会显示图片，Frame有页面就不会显示，因为ZIndex Frame比Image大，很简单

页面传参数很简单，首先是Frame

```
FrameNavigate(typeof(页), 参数);
```

我们在参数写我们要传页面

在页面

```
        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            var 参数= e.Parameter as 传输的参数;
            base.OnNavigatedTo(e);
        }
```

##List点击

我们创建数据Model，我们使用MVVM

```
    public class AddressBook
    {
        public string Id { set; get; }
        public string Name { set; get; }
        public string Str { set; get; }
    }
```

随便的，可以根据你需要修改

我们在ViewModel，我在View新建两个`DetailPage.xaml` `MasterDetailPage.xaml`，所以在ViewModel DetailMasterModel.cs

我们在里面

```
        public ObservableCollection<AddressBook> EccryptAddress { set; get; }
```

记住要修改列的数量需要使用的

然后我们需要在View写，让我们的数据显示

```
                <ListView ItemClick="{x:Bind View.MasterClick}"
                      IsItemClickEnabled="True"
                      ItemsSource="{x:Bind View.EccryptAddress}"
                      >
                    <ListView.ItemTemplate>
                        <DataTemplate x:DataType="view:AddressBook">
                            <Grid>
                                <TextBlock Text="{x:Bind Name}"></TextBlock>
                            </Grid>
                        </DataTemplate>
                    </ListView.ItemTemplate>
                </ListView>
```

当然需要我们在view.xaml.cs

```
        public MasterDetailPage()
        {
            View = new DetailMasterModel();
            this.InitializeComponent();
        }

        private DetailMasterModel View { set; get; }
```

我们给ListView我们ViewModel的数据，这样就可以显示，我们使用ItemClick可以得到ListView被点击，当然要`IsItemClickEnabled="True"`

```
        public void MasterClick(object o, ItemClickEventArgs e)
        {
            AddressBook temp = e.ClickedItem as AddressBook;
            if (temp == null)
            {
                return;
            }
            HasFrame = true;
            Detail.Navigate(typeof(DetailPage), temp.Str);
            Narrow();
        }

```

我们拿到点击传给Frame，在ViewModel，把Frame叫Detail

因为点击所以我们的Frame有内容 HasFrame=true;

##后退按钮

在App写

```
            Windows.UI.Core.SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility = Windows.UI.Core.AppViewBackButtonVisibility.Visible;

```

我们在ViewModel 

```
            SystemNavigationManager.GetForCurrentView().BackRequested += BackRequested;

```

如果不知道我说的是什么，可以去下我源代码https://github.com/lindexi/UWP

然后在按后退按钮，就把我们的hasFrame=false;

大概我们就把一个页面做好，Detail就显示我们点击传的str

##页面更改大小

我们获得页面大小修改，可以简单

```
       <VisualStateManager.VisualStateGroups >
            <VisualStateGroup CurrentStateChanged="{x:Bind View.NarrowVisual}">
                <VisualState>
                    <VisualState.StateTriggers>
                        <AdaptiveTrigger MinWindowWidth="720"/>
                    </VisualState.StateTriggers>
                    <VisualState.Setters >
                        <!--<Setter Target="Img.Visibility" Value="Collapsed"></Setter>-->
                    </VisualState.Setters>
                </VisualState>
                <VisualState>
                    <VisualState.StateTriggers>
                        <AdaptiveTrigger MinWindowHeight="200">

                        </AdaptiveTrigger>

                    </VisualState.StateTriggers>
                    <VisualState.Setters >

                    </VisualState.Setters>
                </VisualState>
            </VisualStateGroup>
        </VisualStateManager.VisualStateGroups>
```

```
        public void NarrowVisual(object sender, VisualStateChangedEventArgs e)
        {
            Narrow();
        }
```

Window.Current.Bounds.Width放在函数，就可以得到我们的窗口大小。

当然我们可以给我们VisualState名，从e.NewState拿到Name就很简单，我们使用Narrow，判断显示屏是小还是可以显示两个

##修改显示

我们先判断我们现在屏幕，显示两个还是显示List一个，如果是显示两个，那么我们不需要什么，当然我们需要给默认。

默认Grid左边Auto，右边*，分两个，然后左边是List，如果没有Frame，那么显示图片

如果屏小，那么就显示List，这时我们修改Grid为左边*，右边auto，然后把我们Grid，有Frame，修改为左边，这样我们右边就没有，左边有List和Grid

如果我们HasFrame，还记得hasFrame在哪？就是我们Frame存在内容就是true，那么我们把Frame的ZIndex>List的ZIndex，我们就显示Frame，如果我们按返回，那么把List的ZIndex大于Frame

可以看到我们需要设置一个ZIndex就好

我们就在界面变化，和点击后悔，点击列表，使用判断，我们判断写成一个函数，函数判断现在窗口，判断HasFrame，很简单。


