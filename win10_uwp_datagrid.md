# win10 uwp dataGrid

本文告诉大家如何在 UWP 使用 DataGrid ，提供两个方法使用。
 


<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


<div id="toc"></div>
<!-- csdn -->

## Microsoft.Toolkit.Uwp.UI.Controls.DataGrid

这是比较推荐的库，使用也很简单

### 安装

首先需要通过 Nuget 搜索 Microsoft.Toolkit.Uwp.UI.Controls.DataGrid 安装

### 界面

xaml：

先引用库

```csharp
    xmlns:controls="using:Microsoft.Toolkit.Uwp.UI.Controls"

```

然后写 DataGrid，需要的代码很少

```csharp
        <controls:DataGrid x:Name="DataGrid" Margin="100,10,10,10" >         

        </controls:DataGrid>
```

这时就可以尝试按 F5 运行代码，虽然只有什么都没有的表

### 设置数据

在设置数据之前，需要先定义一个类作为数据，下面定义 Foo ，里面只有两个属性

```csharp

    public class Foo
    {
        public string Name { get; set; }

        public string Url { get; set; }
    }
```

通过 DataGrid.ItemsSource 可以给数据

```csharp
	        DataGrid.ItemsSource = new List<Foo>()
            {
                new Foo()
                {
                    Name = "lindexi",
                    Url = "lindexi.gitee.io"
                }
            };
```

这句代码需要写在构造，但是需要在 InitializeComponent 之后

```csharp
	     public MainPage()
        {
            this.InitializeComponent();

            DataGrid.ItemsSource = new List<Foo>()
            {
                new Foo()
                {
                    Name = "lindexi",
                    Url = "lindexi.gitee.io"
                }
            };
        }
```

尝试按下F5，可以看到这个界面

<!-- ![](image/win10_uwp_datagrid/win10_uwp_datagrid0.png) -->

![](http://image.acmx.xyz/lindexi%2F20185271922537551.jpg)

也就是不需要写代码就可以自动创建表格，因为默认的 AutoGenerateColumns 就是 true ，如果需要自定义表头，请看下面

### 自定义

因为大家都不希望显示表头就是属性名，所以需要定义表格

首先需要关闭自动生成

```csharp
        <controls:DataGrid x:Name="DataGrid" Margin="100,10,10,10" AutoGenerateColumns="False" d:DataContext="{d:DesignInstance local:Foo}">
        </controls:DataGrid>
```

然后在使用 DataGridTextColumn 写出一行

```csharp
        <controls:DataGrid x:Name="DataGrid" Margin="100,10,10,10" AutoGenerateColumns="False" d:DataContext="{d:DesignInstance local:Foo}">
            <controls:DataGrid.Columns>
                <controls:DataGridTextColumn Header="名字" Binding="{Binding Name}"/>
                <controls:DataGridTextColumn Header="网站" Binding="{Binding Url}"/>
            </controls:DataGrid.Columns>

        </controls:DataGrid>
```

现在按下 F5 ，可以看到下面界面

<!-- ![](image/win10_uwp_datagrid/win10_uwp_datagrid1.png) -->

![](http://image.acmx.xyz/lindexi%2F2018527192719520.jpg)

### 属性

下面是一些其他的设置

#### GridLinesVisibility

是否显示表格线，如果设置为 None ，那么除了表头，其他地方都不显示表格线

<!-- ![](image/win10_uwp_datagrid/win10_uwp_datagrid2.png) -->

![](http://image.acmx.xyz/lindexi%2F20185271936574099.jpg)

如果设置为 Horizontal 就显示水平的表格线，如下图

<!-- ![](image/win10_uwp_datagrid/win10_uwp_datagrid3.png) -->

![](http://image.acmx.xyz/lindexi%2F2018527193719194.jpg)

还可以设置为 Vertical 只显示水平表格线，和设置 All 显示水平和垂直的表格线

#### 交替行

通过 AlternateRowBackground 可以设置交替行的背景，下面会设置`AlternatingRowBackground="LightGray"`让第二行背景修改

```csharp
        <controls:DataGrid x:Name="DataGrid" Margin="100,10,10,10" AutoGenerateColumns="False" d:DataContext="{d:DesignInstance local:Foo}" GridLinesVisibility="Vertical" AlternatingRowBackground="LightGray">
            <controls:DataGrid.Columns>
                <controls:DataGridTextColumn Header="名字" Binding="{Binding Name}"/>
                <controls:DataGridTextColumn Header="网站" Binding="{Binding Url}"/>
            </controls:DataGrid.Columns>

        </controls:DataGrid>
```

<!-- ![](image/win10_uwp_datagrid/win10_uwp_datagrid4.png) -->

![](http://image.acmx.xyz/lindexi%2F20185271940583087.jpg)

#### 多选

通过设置 SelectionMode = Extended 可以支持多选，通过设置 SelectionMode 可以设置单选

其他的属性，大家试试就知道

请看：[DataGrid XAML Control - Windows Community Toolkit](https://docs.microsoft.com/en-us/windows/uwpcommunitytoolkit/controls/datagrid )

[DataGrid](https://docs.microsoft.com/en-us/previous-versions/windows/silverlight/dotnet-windows-silverlight/cc189753(v=vs.95) )

## 表格控件

我们先要知道我说的是哪个？

其实DataGrid就是表格控件，本文就是告诉大家如何做一个UWP 表格控件

一开始我是改ListView，ListView有个问题，就是你设置他的宽度实际是很小，这个如何做？

其实简单UWP ListView宽度过小，可以通过下面代码修改

```xml
                <ListView.ItemContainerStyle>
                    <Style TargetType="ListViewItem">
                        <Setter Property="HorizontalContentAlignment"
                                Value="Stretch"></Setter>
                    </Style>
                </ListView.ItemContainerStyle>

```

我们这个问题还可以做ListView对齐，ListBox内容对齐，ListBox宽度过小的解决

这样我们手动写表格，手动写表格宽度不好做，因为我们需要都是固定宽度

参见：http://www.cnblogs.com/FaDeKongJian/p/5860148.html

看到国内一个大神写的：https://github.com/zmtzawqlp/UWP-master/commits/master 
 
现在可以使用： https://github.com/MyToolkit/MyToolkit/wiki/DataGrid
 
国外 https://liftcodeplay.com/2015/10/24/datagrid-alternatives-in-uwp/
 
需要钱的：https://www.syncfusion.com/products/uwp/sfdatagrid

![](http://i.wotula.com/wp.png)
 
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

