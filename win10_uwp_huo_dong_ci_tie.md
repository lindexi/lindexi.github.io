# win10 uwp 活动磁贴


本文翻译：https://mobileprogrammerblog.wordpress.com/2015/12/23/live-tiles-and-notifications-in-universal-windows-10-app/ 我会写很多质量很低文章，文章都是胡说，如果看不懂可以发到邮箱

如下面的图，很多应用都有活动磁贴，活动磁贴就是放在开始菜单，会像是下面图一样显示东西

![这里写图片描述](http://img.blog.csdn.net/20160426165426823)

win10总有很多看起来有用，但实际没什么卵用的东西，我一点不觉得用户觉得这个有用，但是我们能做活动磁贴UWP，微软一直把开发者当成用户。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

做一个UWP当然需要我们打开神器

新建一个项目，空UWP，可以使用快捷键`ctrl+shift+N`

![这里写图片描述](http://img.blog.csdn.net/20160426165458772)

我们打开MainPage.xaml，新建的时候有点慢，我们需要等一下如果放在固态基本不用等。

![这里写图片描述](http://img.blog.csdn.net/20160426165506167)

```xml
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition />
            <RowDefinition />
        </Grid.RowDefinitions>
        <Grid>
            <Grid.RowDefinitions>
                <RowDefinition />
                <RowDefinition />
            </Grid.RowDefinitions>
            <StackPanel Grid.Row="0" Margin="12">
                <TextBlock Text="Adaptive Tiles" FontSize="20" FontWeight="Bold" />
                <Button Click="UpdateBadge" VerticalAlignment="Top" Margin="12" Background="#330070B0">Update Badge Count</Button>
                <Button Click="UpdatePrimaryTile" VerticalAlignment="Top" Background="#330070B0" Margin="12">Update Primary Tile</Button>
            </StackPanel>
            <StackPanel Grid.Row="1" Margin="12">
                <TextBlock Text="Interactive Toast" FontSize="20" FontWeight="Bold" />
                <StackPanel Orientation="Horizontal" Margin="12">
                    <TextBlock x:Name="Description" VerticalAlignment="Center" Text="{x:Bind CurrentToDoTask.Description, Mode=OneWay}" FontWeight="Bold" />
                    <CheckBox Margin="12,0,0,0" IsChecked="{x:Bind CurrentToDoTask.IsComplete, Mode=OneWay}" IsEnabled="False" />
                </StackPanel>
                <Button Click="Notify" Background="#330070B0" Margin="12">Notify</Button>
                <Button Background="#330070B0" Click="{x:Bind Refresh}" Margin="12">Refresh</Button>
            </StackPanel>
        </Grid>
        <Grid Grid.Row="1">
            <Grid.RowDefinitions>
                <RowDefinition />
                <RowDefinition />
            </Grid.RowDefinitions>
            <StackPanel Grid.Row="0" Margin="12">
                <TextBlock Text="我翻译的 本文来自http://blog.csdn.net/lindexi_gd" />
                <TextBlock Text="磁贴" FontSize="20" FontWeight="Bold" />
                <Button Click="UpdateBadge" VerticalAlignment="Top" Margin="12" Background="#330070B0">更新磁贴数</Button>
                <Button Click="UpdatePrimaryTile" VerticalAlignment="Top" Background="#330070B0" Margin="12">更新显示磁贴</Button>
            </StackPanel>
            <StackPanel Grid.Row="1" Margin="12">
                <TextBlock Text="互动吐司" FontSize="20" FontWeight="Bold" />
                <StackPanel Orientation="Horizontal" Margin="12">
                    <TextBlock x:Name="xdescription" VerticalAlignment="Center" Text="{x:Bind CurrentToDoTask.Description, Mode=OneWay}" FontWeight="Bold" />
                    <CheckBox Margin="12,0,0,0" IsChecked="{x:Bind CurrentToDoTask.IsComplete, Mode=OneWay}" IsEnabled="False" />
                </StackPanel>
                <Button Click="Notify" Background="#330070B0" Margin="12">通知</Button>
                <Button Background="#330070B0" Click="{x:Bind Refresh}" Margin="12">更新</Button>
            </StackPanel>
        </Grid>
    </Grid>
```

写完我们可以看到下面的样子

![这里写图片描述](http://img.blog.csdn.net/20160426165524323)

![这里写图片描述](http://img.blog.csdn.net/20160426165532823)

上面一张是作者写的开始我没有去看，以为他写出来就是上面那图，复制了他代码在我写博客，发现他的代码错了，我自己重新写，发现我应该弄个中文，就写了第二张图，我们看到上面代码是第二张图。

我们右击方案新建一个文件夹`DATA`，里面新建一个类`PrimaryTile`，可以看下面图

![这里写图片描述](http://img.blog.csdn.net/20160426165542151)

我们在PrimaryTile

```csharp
    public class PrimaryTile
    {
        public string time
        {
            set;
            get;
        } = "8:15 AM, Saturday";
        public string message
        {
            set;
            get;
        } = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed  do eiusmod tempor incididunt ut labore.";
        public string message2
        {
            set;
            get;
        } = "At vero eos et accusamus et iusto odio dignissimos ducimus  qui blanditiis praesentium voluptatum deleniti atque corrupti quos dolores et quas molestias  excepturi sint occaecati cupiditate non provident.";
        public string branding
        {
            set;
            get;
        } = "name";
        public string appName
        {
            set;
            get;
        } = "UWP";
    }
```

![这里写图片描述](http://img.blog.csdn.net/20160426165607479)

创建一个文件夹`services` 新建`tileservice.cs` `toastservice.cs`

```csharp
    public class TileService
    {
        public static void SetBadgeCountOnTile(int count)
        {
            // Update the badge on the real tile  
            System.Xml.XmlDocument badgeXml = BadgeUpdateManager.GetTemplateContent(BadgeTemplateType.BadgeNumber);

            XmlElement badgeElement = (XmlElement) badgeXml.SelectSingleNode("/badge");
            badgeElement.SetAttribute("value", count.ToString());

            BadgeNotification badge = new BadgeNotification(badgeXml);
            BadgeUpdateManager.CreateBadgeUpdaterForApplication().Update(badge);
        }

        public static XmlDocument CreateTiles(PrimaryTile primaryTile)
        {
            XDocument xDoc = new XDocument(
                new XElement("tile", new XAttribute("version", 3),
                    new XElement("visual",
                        // Small Tile  
                        new XElement("binding", new XAttribute("branding", primaryTile.branding),
                            new XAttribute("displayName", primaryTile.appName), new XAttribute("template", "TileSmall"),
                            new XElement("group",
                                new XElement("subgroup",
                                    new XElement("text", primaryTile.time, new XAttribute("hint-style", "caption")),
                                    new XElement("text", primaryTile.message,
                                        new XAttribute("hint-style", "captionsubtle"), new XAttribute("hint-wrap", true),
                                        new XAttribute("hint-maxLines", 3))
                                    )
                                )
                            ),

                        // Medium Tile  
                        new XElement("binding", new XAttribute("branding", primaryTile.branding),
                            new XAttribute("displayName", primaryTile.appName), new XAttribute("template", "TileMedium"),
                            new XElement("group",
                                new XElement("subgroup",
                                    new XElement("text", primaryTile.time, new XAttribute("hint-style", "caption")),
                                    new XElement("text", primaryTile.message,
                                        new XAttribute("hint-style", "captionsubtle"), new XAttribute("hint-wrap", true),
                                        new XAttribute("hint-maxLines", 3))
                                    )
                                )
                            ),

                        // Wide Tile  
                        new XElement("binding", new XAttribute("branding", primaryTile.branding),
                            new XAttribute("displayName", primaryTile.appName), new XAttribute("template", "TileWide"),
                            new XElement("group",
                                new XElement("subgroup",
                                    new XElement("text", primaryTile.time, new XAttribute("hint-style", "caption")),
                                    new XElement("text", primaryTile.message,
                                        new XAttribute("hint-style", "captionsubtle"), new XAttribute("hint-wrap", true),
                                        new XAttribute("hint-maxLines", 3)),
                                    new XElement("text", primaryTile.message2,
                                        new XAttribute("hint-style", "captionsubtle"), new XAttribute("hint-wrap", true),
                                        new XAttribute("hint-maxLines", 3))
                                    ),
                                new XElement("subgroup", new XAttribute("hint-weight", 15),
                                    new XElement("image", new XAttribute("placement", "inline"),
                                        new XAttribute("src", "Assets/StoreLogo.png"))
                                    )
                                )
                            ),

                        //Large Tile  
                        new XElement("binding", new XAttribute("branding", primaryTile.branding),
                            new XAttribute("displayName", primaryTile.appName), new XAttribute("template", "TileLarge"),
                            new XElement("group",
                                new XElement("subgroup",
                                    new XElement("text", primaryTile.time, new XAttribute("hint-style", "caption")),
                                    new XElement("text", primaryTile.message,
                                        new XAttribute("hint-style", "captionsubtle"), new XAttribute("hint-wrap", true),
                                        new XAttribute("hint-maxLines", 3)),
                                    new XElement("text", primaryTile.message2,
                                        new XAttribute("hint-style", "captionsubtle"), new XAttribute("hint-wrap", true),
                                        new XAttribute("hint-maxLines", 3))
                                    ),
                                new XElement("subgroup", new XAttribute("hint-weight", 15),
                                    new XElement("image", new XAttribute("placement", "inline"),
                                        new XAttribute("src", "Assets/StoreLogo.png"))
                                    )
                                )
                            )
                        )
                    )
                );

            XmlDocument xmlDoc = new XmlDocument();
            xmlDoc.LoadXml(xDoc.ToString());
            //Debug.WriteLine(xDoc);  
            return xmlDoc;
        }
    }

    public static class ToastService
    {
        public static System.Xml.XmlDocument CreateToast()
        {
            XDocument xDoc = new XDocument(
                new XElement("toast",
                    new XElement("visual",
                        new XElement("binding", new XAttribute("template", "ToastGeneric"),
                            new XElement("text", "To Do List"),
                            new XElement("text", "Is the task complete?")
                            ) // binding  
                        ), // visual  
                    new XElement("actions",
                        new XElement("action", new XAttribute("activationType", "background"),
                            new XAttribute("content", "Yes"), new XAttribute("arguments", "yes")),
                        new XElement("action", new XAttribute("activationType", "background"),
                            new XAttribute("content", "No"), new XAttribute("arguments", "no"))
                        ) // actions  
                    )
                );

            System.Xml.XmlDocument xmlDoc = new System.Xml.XmlDocument();
            xmlDoc.LoadXml(xDoc.ToString());
            return xmlDoc;
        }
    }
```

我们创建文件`ToDoTask.cs` `ToDoTaskFileHelper.cs`

![这里写图片描述](http://img.blog.csdn.net/20160426165644698)

```csharp
    public class ToDoTask
    {
        public string Description
        {
            get;
            set;
        }

        public Guid Id
        {
            get;
            set;
        }

        public bool? IsComplete
        {
            get;
            set;
        }

        public string ToJson()
        {
            using (MemoryStream stream = new MemoryStream())
            {
                DataContractJsonSerializer serializer = new DataContractJsonSerializer(typeof (ToDoTask));
                serializer.WriteObject(stream, this);
                stream.Position = 0;
                byte[] jsonBytes = stream.ToArray();
                return Encoding.UTF8.GetString(jsonBytes, 0, jsonBytes.Length);
            }
        }

        public static ToDoTask FromJson(string json)
        {
            // note: throws exception if the json is not valid  
            JsonObject jsonData = JsonObject.Parse(json);

            // exceptions will be thrown if the values do not match the types  
            return new ToDoTask
            {
                Id = Guid.Parse(jsonData["Id"].GetString()),
                Description = jsonData["Description"].GetString(),
                IsComplete = jsonData["IsComplete"].GetBoolean()
            };
        }
    }

    public static class ToDoTaskFileHelper
    {
        public static async Task ReadToDoTaskJsonAsync()
        {
            // declare an empty variable to be filled later  
            string json = null;
            // define where the file resides  
            StorageFolder localfolder = ApplicationData.Current.LocalFolder;
            // see if the file exists  
            if (await localfolder.TryGetItemAsync(Filename) != null)
            {
                // open the file  
                StorageFile textfile = await localfolder.GetFileAsync(Filename);
                // read the file  
                json = await FileIO.ReadTextAsync(textfile);
            }
            // if the file doesn't exist, we'll copy the app copy to local storage  
            else
            {
                StorageFile storageFile =
                    await StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appx:///Assets/task.json"));
                await storageFile.CopyAsync(ApplicationData.Current.LocalFolder);
                json = await FileIO.ReadTextAsync(storageFile);
            }

            return json;
        }

        public static async Task SaveToDoTaskJson(string json)
        {
            StorageFolder localfolder = ApplicationData.Current.LocalFolder;
            StorageFile textfile = await localfolder.GetFileAsync(Filename);
            await FileIO.WriteTextAsync(textfile, json);
        }

        private static readonly string Filename = "task.json";
    }
```
`task.json`

```csharp
{"Description":"A test task","Id":"9d6c3585-d0c2-4885-8fe0-f02727f8e483","IsComplete":true}     

```

我们把刚才写的MainPage的按钮绑定到

![这里写图片描述](http://img.blog.csdn.net/20160426165719213)

```csharp
    public sealed partial class MainPage : Page, INotifyPropertyChanged
    {
        public MainPage()
        {
            InitializeComponent();
            Loaded += MainPage_Loaded;
        }

        #region Delegates

        public event PropertyChangedEventHandler PropertyChanged;

        private void MainPage_Loaded(object sender, RoutedEventArgs e)
        {
            Refresh();
        }

        #endregion

        public ToDoTask CurrentToDoTask
        {
            get
            {
                return _currentToDoTask;
            }
            set
            {
                _currentToDoTask = value;
                PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(CurrentToDoTask)));
            }
        }

        private int _count;
        private ToDoTask _currentToDoTask;

        private async void Refresh()
        {
            void json = await ToDoTaskFileHelper.ReadToDoTaskJsonAsync();
            CurrentToDoTask = ToDoTask.FromJson(json);
        }

        private void UpdateBadge(object sender, RoutedEventArgs e)
        {
            _count++;
            TileService.SetBadgeCountOnTile(_count);
        }

        private void UpdatePrimaryTile(object sender, RoutedEventArgs e)
        {
            XmlDocument xmlDoc = TileService.CreateTiles(new PrimaryTile());

            TileUpdater updater = TileUpdateManager.CreateTileUpdaterForApplication();
            TileNotification notification = new TileNotification(xmlDoc);
            updater.Update(notification);
        }

        private void Notify(object sender, RoutedEventArgs e)
        {
            System.Xml.XmlDocument xmlDoc = ToastService.CreateToast();
            ToastNotifier notifier = ToastNotificationManager.CreateToastNotifier();
            ToastNotification toast = new ToastNotification(xmlDoc);
            notifier.Show(toast);
        }
    }
```

写完自己运行就可以知道，更新磁贴，更新界面，提示通知，每个对应的代码自己可以看到，这个国内很多教程

![这里写图片描述](http://img.blog.csdn.net/20160426165745276)

![这里写图片描述](http://img.blog.csdn.net/20160426165752073)

http://blog.csdn.net/lindexi_gd

https://mobileprogrammerblog.wordpress.com/2015/12/23/live-tiles-and-notifications-in-universal-windows-10-app/


