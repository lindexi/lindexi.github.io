俄罗斯方块是一个很经典的游戏，做一个UWP俄罗斯方块没有什么用，我想说的是移植，把经典游戏移植到UWP。

我们之前有很多游戏，很多软件使用C或者C++之类的来写，或者C#，其实我们可以把之前的算法拿出来，转换为UWP的C#，这时大家会说，界面。对，界面我们没法直接移植，但是用XAML做一个界面很快的，那么现在的问题就是，我们如何使用之前的算法来用现在的界面。

简单的一个，我们可以使用绑定。

MVVM的知识，我觉得看到一篇文章，忘了出错，希望知道的小伙伴提醒。他说，MVVM的ViewModel作用是界面的抽象。我们不用理界面，因为界面总是改，所以我们需要一个抽象的界面，就是我们做的ViewModel，那么model做的就是算法，数据。Model不知道界面怎样，他需要知道数据。ViewModel不知道界面怎样，他知道界面需要什么。

MVVM的知识我说的不算对，也不算错，但从这个看也是可以。

为什么要分开view？

其实可以看下面的：

假设我们需要做一个软件，这个软件是举报恶意转载的功能，他能够在网上搜，找到那些恶意转载的网站。
先吐槽下中国学网那些垃圾网站，全部东西都是转载的。吐槽下百度，搜索到的转载的都是前，找不到我的博客。

![这里写图片描述](http://img.blog.csdn.net/20160505152213804)

还是360好，能找到

![这里写图片描述](http://img.blog.csdn.net/20160505152243696)

我们软件开始界面

![这里写图片描述](http://img.blog.csdn.net/20160505152318099)

发现我们需要改

![这里写图片描述](http://img.blog.csdn.net/20160505153432025)

接着发现还是需要改

![这里写图片描述](http://img.blog.csdn.net/20160505153448697)

如果我们和界面有联系，一改界面就需要改，那么这样我们开发将会很慢。

如果我们能使用抽象，那么界面怎么改，我们修改的也就界面。

上面图片来自：http://my.oschina.net/Android1989/blog/296850 

我们需要做一个游戏，我们有了之前的算法，我拿到了一位大神：http://www.cnblogs.com/china_x01/p/5253556.html

看不懂他写的，问了一位大神，他帮我改了UWP，最后我也看不懂，他写的没有注释。

做一个俄罗斯方块算法简单，我们放在后面，现在先和大家说界面。

[后面](#俄罗斯方块)说的有些小白。



我们程序：

- view:MainPage.xaml
- viewModel.cs
- model.cs

我们在界面

放一个Canvas

里面就是游戏

![这里写图片描述](http://img.blog.csdn.net/20160505153731872)

因为我们需要游戏按键，所以我们需要一个TextBox

```
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <TextBox Margin="10,10,10,10" Width="1" Height="1" KeyDown="keydown"></TextBox>
        <Canvas x:Name="canvas" Margin="10,10,10,10">

        </Canvas>
    </Grid>
```

每个方块
Rectangle[,] _rectangle
我们需要设计高度宽度 `size = 10;`
现在就是我们重要的绑定，我们有200个Rectangle，如果每个在xaml，写我觉得我没有这么时间，也容易错

所以我们在

```
            for (int i = 0; i < view.row; i++)
            {
                for (int j = 0; j < view.col; j++)
                {
                    _rectangle[i, j] = new Rectangle()
                    {
                        Width = size,
                        Height = size,
                        Fill = new SolidColorBrush(Colors.Gray),
                        Stroke = new SolidColorBrush(Colors.LightCoral),                      
                        AllowDrop = false,
                        CanDrag = false,
                        Margin = new Thickness(j * size, i * size, 0, 0)
                    };  
                    canvas.Children.Add(_rectangle[i, j]);                 
                }
            }
```

我们有的model，有俄罗斯方块的初始方块、移动、变形、向下

他把所有的数据保存在一个数组`grid_observable`，类型grid里面有个`rectangle`，如果为0表示这个地方没有方块，如果为1表示有方块。

类型grid
* 长
* 宽
* 是否有方块

我们界面根据rectangle显示，如果有，那么显示灰色，没有显示白色。

因为我们view是不知道后台，所以这个显示需要viewModel把后台的rectangle变为颜色。

我们ViewModel把颜色放`ObservableCollection<solid> solid_collection`

需要把rectangle变为颜色

```

            foreach (grid temp in _model.grid_observable)
            {
                if (temp.rectangle == 0)
                {
                    solid_collection[temp.row * col + temp.col].solids = new SolidColorBrush(Colors.Gray);
                }
                else 
                {
                    solid_collection[temp.row * col + temp.col].solids = new SolidColorBrush(Colors.White);
                }
            }
            
```

为了让solid一修改就可以告诉view

```
    public class solid : notify_property
    {
        public solid(SolidColorBrush solid)
        {
            _solid = solid;
        }

        public SolidColorBrush solids
        {
            set
            {
                _solid = value;
                OnPropertyChanged();
            }
            get
            {
                return _solid;
            }
        }
        private SolidColorBrush _solid;
    }
```

因为每次写INotifyPropertyChanged要写很多，我们需要通知有很多 ，所以写notify_property

ViewModel能把后台的rectangle变颜色，那么我们view把颜色显示

我们刚才new 了200个`Rectangle`我们把他的颜色绑定ViewModel

如果使用xaml，我觉得我没法

那么我们在代码

```
                    _rectangle[i, j] = new Rectangle()
                    {
                        Width = size,
                        Height = size,
                        Fill = new SolidColorBrush(Colors.Gray),
                        Stroke = new SolidColorBrush(Colors.LightCoral),                      
                        AllowDrop = false,
                        CanDrag = false,
                        Margin = new Thickness(j * size, i * size, 0, 0)
                    };      
                    Binding bind = new Binding()
                    {
                        Path = new PropertyPath("solid_collection[" + (i * view.col + j) + "].solids"),
                        Mode = BindingMode.OneWay
                    };
                    _rectangle[i, j].DataContext = view;
                    _rectangle[i, j].SetBinding(Shape.FillProperty, bind);
```

绑定可以` Binding bind = new Binding()`
里面写路径，可以数组中`Path = new PropertyPath("solid_collection[" + (i * view.col + j) + "].solids"),`
其实Path写在new Binding(string Path)

我们可以设置`Source = view`

```
                    Binding bind = new Binding()
                    {
                        Path = new PropertyPath("solid_collection[" + (i * view.col + j) + "].solids"),
                        Mode = BindingMode.OneWay,
                        Source = view
                    };
```
也可以`_rectangle[i, j].DataContext = view;`
写完我们需要

```
_rectangle[i, j].SetBinding(Shape.FillProperty, bind);
```

如果我们后台是可以的，那么我们就能看到

![这里写图片描述](http://img.blog.csdn.net/20160505153644699)

我想说的不是写俄罗斯，而是把之前的软件移植，我们可以把二维表，bool，表示为颜色，把颜色显示，我们有很多游戏都是可以这样，那么移植UWP简单，需要使用绑定，一个转换。

大神：可以直接绑定转换。

其实我是不喜欢直接绑定就转换，因为这样类很多，我们需要文件夹
Convert里面是转换类

写到这，后面都是小白

#俄罗斯方块

我们先打开vs神器，之前下载vs安装，需要sdk，这个在安装自己弄。

新建项目，我叫tetris

新建一个类叫viewModel，一个model

再新建一个类notify_property，接口INotifyPropertyChanged

```
    /// <summary>
    /// 提供继承通知UI改变值
    /// </summary>
    public class notify_property : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;
        public void UpdateProper<T>(ref T properValue , T newValue , [System.Runtime.CompilerServices.CallerMemberName] string properName = "")
        {
            if (object.Equals(properValue , newValue))
                return;

            properValue = newValue;
            OnPropertyChanged(properName);
        }
        public void OnPropertyChanged([System.Runtime.CompilerServices.CallerMemberName] string name="")
        {
            PropertyChangedEventHandler handler = PropertyChanged;
            handler?.Invoke(this , new PropertyChangedEventArgs(name));
        }
    }
```
这个类是我们每次需要INotifyPropertyChanged都需要写PropertyChanged，觉得还是放成类，让需要的继承

俄罗斯方块有

- 新建方块
- 方块移动
- 方块向下
- 碰到下面方块
- 清除

我们把算法写model

方块有

-         straight, 
![这里写图片描述](http://img.blog.csdn.net/20160505154803058)
-         square,
![这里写图片描述](http://img.blog.csdn.net/20160505154810261)
-         t,
![这里写图片描述](http://img.blog.csdn.net/20160505154818235)
-         bent 
![这里写图片描述](http://img.blog.csdn.net/20160505154825152)

我们需要做一个来保存

```
    public enum block
    {
        straight, 
        square,
        t,
        bent
    }
```

那么我们需要一个来放我们的方块

```
    public class grid : notify_property
    {
        public grid()
        {
            _rectangle = 1;
        }

        public grid(int col, int row)
        {
            _col = col;
            _row = row;
            _rectangle = 0;
        }

        public grid clone()
        {
            return new grid(col, row);
        }

        public int row
        {
            set
            {
                _row = value;
                OnPropertyChanged();
            }
            get
            {
                return _row;
            }
        }

        public int col
        {
            set
            {
                _col = value;
                OnPropertyChanged();
            }
            get
            {
                return _col;
            }
        }

        public int rectangle
        {
            set
            {
                _rectangle = value;
            }
            get
            {
                return _rectangle;
            }
        }

        private int _col;

        private int _rectangle;

        private int _row;
    }
```
行列，是否有方块

我们发现这个只能放一个方块，所以我们写

![这里写图片描述](http://img.blog.csdn.net/20160505164603757)

放着`grid[] _grid;`

新建方块：
square(block block, int center)
我们需要方块是什么，中心

我们先做直线

```
        public square(block block, int center)
        {
            _block = block;
            int n = 4;
            _grid = new grid[n];
            for (int i = 0; i < n; i++)
            {
                _grid[i] = new grid();
                switch (block)
                {
                    case block.straight:
                        _grid[i].col = center;
                        _grid[i].row = -i;
                        break;
                    default:
                        throw new ArgumentOutOfRangeException(nameof(block), block, null);
                }
            }
        }
```

![这里写图片描述](http://img.blog.csdn.net/20160505164614054)

我们来做t

```
                    case block.t:
                        _grid[0].col = center;
                        _grid[0].row = 0;
                        if (i > 0)
                        {
                            _grid[i].col = center + i - 3;
                            _grid[i].row = -1;
                        }
```

![这里写图片描述](http://img.blog.csdn.net/20160505164621304)

square

```
                   case block.square:
                        if (i <= 1)
                        {
                            _grid[i].col = center + i;
                            _grid[i].row = 0;
                        }
                        else
                        {
                            _grid[i].col = center + i - 2;
                            _grid[i].row = -1;
                        }
```

![这里写图片描述](http://img.blog.csdn.net/20160505164628461)

bent

```
                    case block.bent:
                        if (i <= 1)
                        {
                            _grid[i].col = center + i;
                            _grid[i].row = 0;
                        }
                        else
                        {
                            _grid[i].col = center + i - 3;
                            _grid[i].row = -1;
                        }
```

![这里写图片描述](http://img.blog.csdn.net/20160505164635199)

```
        public square(block block, int center)
        {
            _block = block;
            int n = 4;
            _grid = new grid[n];
            for (int i = 0; i < n; i++)
            {
                _grid[i] = new grid();
                switch (block)
                {
                    case block.straight:
                        _grid[i].col = center;
                        _grid[i].row = -i;
                        break;
                    case block.t:
                        _grid[0].col = center;
                        _grid[0].row = 0;
                        if (i > 0)
                        {
                            _grid[i].col = center + i - 3;
                            _grid[i].row = -1;
                        }
                        break;
                    case block.square:
                        if (i <= 1)
                        {
                            _grid[i].col = center + i;
                            _grid[i].row = 0;
                        }
                        else
                        {
                            _grid[i].col = center + i - 2;
                            _grid[i].row = -1;
                        }
                        break;
                    case block.bent:
                        if (i <= 1)
                        {
                            _grid[i].col = center + i;
                            _grid[i].row = 0;
                        }
                        else
                        {
                            _grid[i].col = center + i - 3;
                            _grid[i].row = -1;
                        }
                        break;
                    default:
                        throw new ArgumentOutOfRangeException(nameof(block), block, null);
                }
            }
        }
```

这样看起来代码很多，这样不好，我们需要把每个放在一个函数

```
        public square(block block, int center)
        {
            _block = block;
            int n = 4;
            _grid = new grid[n];
            for (int i = 0; i < n; i++)
            {
                _grid[i] = new grid();
                switch (block)
                {
                    case block.straight:
                        block_straight(center, i);
                        break;
                    case block.t:
                        block_t(center, i);
                        break;
                    case block.square:
                        block_square(center, i);
                        break;
                    case block.bent:
                        block_bent(center, i);
                        break;
                    default:
                        throw new ArgumentOutOfRangeException(nameof(block), block, null);
                }
            }
        }

        private void block_straight(int center, int i)
        {
            _grid[i].col = center;
            _grid[i].row = -i;
        }

        private void block_t(int center, int i)
        {
            _grid[0].col = center;
            _grid[0].row = 0;
            if (i > 0)
            {
                _grid[i].col = center + i - 3;
                _grid[i].row = -1;
            }
        }

        private void block_bent(int center, int i)
        {
            if (i <= 1)
            {
                _grid[i].col = center + i;
                _grid[i].row = 0;
            }
            else
            {
                _grid[i].col = center + i - 3;
                _grid[i].row = -1;
            }
        }
```

我们model还没写东西
我们先做新建方块

我们需要最大值

```
        public int row
        {
            set
            {
                _row = value;
            }
            get
            {
                return _row;
            }
        }
        public int col
        {
            set
            {
                _col = value;
            }
            get
            {
                return _col;
            }
        }

        private int _col = 10;
        private int _row = 20;
```

当前方块

```
        public square _square
        {
            set;
            get;
        }
```

新建方块

```
        private void new_block()
        {
            block _block = (block)ran.Next(4);
            int center = _col / 2;
            _square = new square(_block, center);
        }
```

我们现在没有想着去什么，我们需要显示

每次下降

```
        public void down()
        {
            if (_square == null)
            {
                new_block();
            }
        }
```

我们在ViewModel

```
        public viewModel()
        {
            solid_collection = new ObservableCollection<solid>();
            for (int i = 0; i < col * row; i++)
            {
                solid_collection.Add(new solid(new SolidColorBrush(Colors.Gray)));
            }
            DispatcherTimer time = new DispatcherTimer
            {
                Interval = TimeSpan.FromSeconds(0.5)
            };
            time.Tick += tick;
            time.Start();
        }
        public int col
        {
            set
            {
                value = 0;
            }
            get
            {
                return _model.col;
            }
        }

        public int row
        {
            set
            {
                value = 0;
            }
            get
            {
                return _model.row;
            }
        }
        public ObservableCollection<solid> solid_collection
        {
            set;
            get;
        }
        private void tick(object sender, object e)
        {
            DispatcherTimer time = sender as DispatcherTimer;
            time?.Stop();
            down();
            time?.Start();
        }

        public void down()
        {
            _model.down();
        }
```

我们需要DispatcherTimer，给他时间`Interval = TimeSpan.FromSeconds(0.5)`就向下

如果model 

```
            if (_square == null)
            {
                new_block();
            }
```

我们现在新建出来，还没有显示

我们需要把`_square `显示

viewModel

```
        public void down()
        {
            _model.down();
            foreach (grid temp in _model._square._grid.Where(temp => temp.col >= 0 && temp.row >= 0))
            {
                solid_collection[temp.row * col + temp.col].solids = new SolidColorBrush(Colors.White);
            }
        }
```

我们现在除了界面，都做好了。

打开MainPage

```
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <TextBox Margin="10,10,10,10" Width="1" Height="1" KeyDown="keydown"></TextBox>
        <Canvas x:Name="canvas" Margin="10,10,10,10">

        </Canvas>
    </Grid>
```

TextBox 我们需要按键

```
        private viewModel view;
        public MainPage()
        {
            InitializeComponent();
            rectangle();
        }
```

rectangle绑定我们的界面
