# WPF 省市县3级联动

本文告诉大家如何使用绑定做省市县3级联动，代码从网上找的。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

首先定义显示的类，包括 id 和 名称 

```csharp
    public class CodeView
    {
        public string Id { get; set; }

        public string Name { get; set; }
    }
```

然后定义省市县的数据

```csharp
    public class Province: CodeView
    {
        public List<City> Child { get; set; }
    }

    public class City: CodeView
    {
        public List<County> Child { get; set; }
    }

    public class County:CodeView
    {

    }
```

因为可以通过 xaml 绑定 选择的元素，所以可以绑定选择的列

请看前台代码，最重要的是通过省选择的元素来作为下一级的数据，于是选择第一个修改时，就会自动联动

```csharp
    <Grid Name="Content">
        <Grid.Resources>
            <Style TargetType="ComboBox" BasedOn="{StaticResource {x:Type ComboBox}}">
                <Setter Property="Margin" Value="0 0 10 0"></Setter>
                <Setter Property="MinWidth" Value="60"></Setter>
            </Style>
        </Grid.Resources>
        <Grid.ColumnDefinitions>
            <ColumnDefinition></ColumnDefinition>
            <ColumnDefinition></ColumnDefinition>
            <ColumnDefinition></ColumnDefinition>
        </Grid.ColumnDefinitions>
        <ComboBox Grid.Column="0" ItemsSource="{Binding Path=Provinces}" x:Name="ComboBoxProvince" 
                  DisplayMemberPath="Name" SelectedValuePath="Id" SelectedValue="{Binding Path=Province,Mode=TwoWay}">
        </ComboBox>
        <ComboBox Grid.Column="1" x:Name="ComboBoxCity" ItemsSource="{Binding Path=SelectedItem.Child,ElementName=ComboBoxProvince}" 
                  DisplayMemberPath="Name" SelectedValuePath="Id" SelectedValue="{Binding Path=City,Mode=TwoWay}" ></ComboBox>
        <ComboBox Grid.Column="2" x:Name="ComboBoxCounty" ItemsSource="{Binding Path=SelectedItem.Child,ElementName=ComboBoxCity}"
                  DisplayMemberPath="Name" SelectedValuePath="Id" SelectedValue="{Binding Path=County,Mode=TwoWay}"></ComboBox>
    </Grid>
```


可以看到`ItemsSource="{Binding Path=SelectedItem.Child,ElementName=ComboBoxProvince}"`绑定了上一级选择的元素，所以就可以联动。

DisplayMemberPath 就是显示的值，所以就可以显示列表是城市的名称。

后台代码需要定义几个属性

```csharp
   public partial class AreaSelect : UserControl, INotifyPropertyChanged
    {

        private IList<Province> _provinces = new List<Province>();

        public static readonly DependencyProperty ProvinceProperty = DependencyProperty.Register("Province", typeof(string), typeof(AreaSelect), new FrameworkPropertyMetadata(string.Empty));
        public static readonly DependencyProperty CityProperty = DependencyProperty.Register("City", typeof(string), typeof(AreaSelect), new FrameworkPropertyMetadata(string.Empty));
        public static readonly DependencyProperty CountyProperty = DependencyProperty.Register("County", typeof(string), typeof(AreaSelect), new FrameworkPropertyMetadata(string.Empty));


        public IList<Province> Provinces
        {
            get
            {
                return _provinces;
            }
            set
            {
                _provinces.Clear();
                if (value != null)
                {
                    foreach (Province province in value)
                    {
                        _provinces.Add(province);
                    }
                }
            }
        }
        
        public AreaSelect()
        {
            InitializeComponent();
            this.Content.DataContext = this;
            this.PropertyChanged += SelectedArea_PropertyChanged;
        }

        private void SelectedArea_PropertyChanged(object sender, PropertyChangedEventArgs e)
        {
            Console.WriteLine(Province + " " + City + " " + County);
        }


        private string _province;
        private string _city;
        private string _county;

        public string Province
        {
            get { return _province; }
            set
            {
                _province = value;
                _city = string.Empty;
                if (PropertyChanged != null)
                {
                    PropertyChanged.Invoke(this, new PropertyChangedEventArgs("Province"));
                }
            }
        }
        public string City
        {
            get { return _city; }
            set
            {
                _city = value;
                _county = string.Empty;
                if (PropertyChanged != null)
                {
                    PropertyChanged.Invoke(this, new PropertyChangedEventArgs("City"));
                }
            }
        }
        public string County
        {
            get { return _county; }
            set
            {
                _county = value;
                if (PropertyChanged != null)
                {
                    PropertyChanged.Invoke(this, new PropertyChangedEventArgs("County"));
                }
            }
        }

        public event PropertyChangedEventHandler PropertyChanged;
    }
```

数据：[省市区](http://image.acmx.xyz/%E7%9C%81%E5%B8%82%E5%8C%BA.7z )

感谢 Baolaitong 提供代码

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。