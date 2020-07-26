# WPF 列表右键菜单比较符合 MVVM 的命令绑定方法

群里小伙伴问我如何在 ListView 的右击的时候知道右击的是哪一项，他想要获取对应的行信息。他使用的是 GridView 做的，于是我告诉他需要自己写 ItemContainerStyle 在 GridViewRowPresenter 里添加右键的逻辑。但是这样写不清真，我问到了他的本质问题其实只是想要做右键菜单。刚好我在写一个测试应用，用于测试我的文件下载库，此时需要用到在右击某一个下载项的时候，拿到当前下载项的信息，给出右键菜单。我不使用小伙伴的逻辑，就按照我自己会采用的写法，我认为这样写比较符合 WPF 框架的设计，下面让我告诉大家我的用法，十分简单

<!--more-->
<!-- 发布 -->

我开源了一个文件下载库，原因是我的几个项目里面都有自己的文件下载库，我想要统一这些文件下载库。开源出去可以让更多小伙伴帮我踩坑，开源项目是 [https://github.com/dotnet-campus/dotnetCampus.FileDownloader](https://github.com/dotnet-campus/dotnetCampus.FileDownloader) 欢迎小伙伴使用

我需要写一个简单的界面程序用来测试我这个库，我计划替换掉我现在自己使用的FDM工具，这样我如果自己下载炸了，我就会去修我的库

在使用的时候我发现我需要这样的一个功能，我需要在下载完成之后，自己去找下载到哪个文件夹，因此我期望能右击对应的下载项的时候，可以给出右键菜单，点击一下就能打开下载的文件所在的文件夹或者打开下载的文件

刚好我的下载界面用了 GridView 用来显示所有的下载项，代码如下

```xml
<ListView ItemsSource="{Binding Path=DownloadFileInfoList}">
    <ListView.View>
        <GridView>
            <GridViewColumn Width="200" Header="文件名" DisplayMemberBinding="{Binding FileName}" />
            <GridViewColumn Width="100" Header="大小" DisplayMemberBinding="{Binding FileSize}"/>
            <GridViewColumn Width="200" Header="进度" DisplayMemberBinding="{Binding DownloadProcess}"/>
            <GridViewColumn Width="100" Header="下载速度" DisplayMemberBinding="{Binding DownloadSpeed}"/>
            <GridViewColumn Width="200" Header="添加日期" DisplayMemberBinding="{Binding AddedTime}"/>
        </GridView>
    </ListView.View>
```

而此时如果我想要先获取所点击的 GridView 是哪一行，然后弹出右键菜单，设置对应的属性，此时的代码逻辑相对来说很复杂

在 WPF 如此优秀的框架里面怎么也需要提供更清真的方法

先忽略绑定的数据是什么，因为没什么意义。按照需求，咱需要一个右键菜单，好那么先创建一个右键菜单

```xml
        <ContextMenu x:Key="DownloadFileContextMenu">
            <MenuItem Header="Open File"></MenuItem>
            <MenuItem Header="Open Folder"></MenuItem>
        </ContextMenu>
```

右键菜单内容十分简单，通过 Header 给定显示的文本，创建右键菜单之后，那么如何让右键菜单绑定到 ListView 上？只需要通过 ItemContainerStyle 设置给 ListView 的每一项就可以了，如下面代码

```xml
<ListView Style="{x:Null}" ItemsSource="{Binding Path=DownloadFileInfoList}">
    <ListView.ItemContainerStyle>
        <Style TargetType="{x:Type ListViewItem}">
            <Setter Property="ContextMenu" Value="{StaticResource DownloadFileContextMenu}"/>
        </Style>
    </ListView.ItemContainerStyle>
    <ListView.View>
        <GridView>
            <GridViewColumn Width="200" Header="文件名" DisplayMemberBinding="{Binding FileName}" />
            <GridViewColumn Width="100" Header="大小" DisplayMemberBinding="{Binding FileSize}"/>
            <GridViewColumn Width="200" Header="进度" DisplayMemberBinding="{Binding DownloadProcess}"/>
            <GridViewColumn Width="100" Header="下载速度" DisplayMemberBinding="{Binding DownloadSpeed}"/>
            <GridViewColumn Width="200" Header="添加日期" DisplayMemberBinding="{Binding AddedTime}"/>
        </GridView>
    </ListView.View>
</ListView>
```

可以看到，主要的代码如下

```xml
    <ListView.ItemContainerStyle>
        <Style TargetType="{x:Type ListViewItem}">
            <Setter Property="ContextMenu" Value="{StaticResource DownloadFileContextMenu}"/>
        </Style>
    </ListView.ItemContainerStyle>
```

通过 ItemContainerStyle 设置一个样式，在样式里面更改 ContextMenu 的内容就可以了，代码量十分少

还有一个问题是如何让右键菜单知道当前点的哪一项？让右键菜单知道当前选中的是哪个 GridView 的 Row 是很逗比的，因为咱可以使用 WPF 的 DataContext 绑定的方法，让数据一层层分发。在每一个 GridView 的 Row 项里面都会使用 ListView 的 ItemSource 的数据的某一项，而咱按照 MVVM 的思想，应该变更的是数据而不是界面本身

而 DataContext 是在视觉树继承的，也就是在对应的元素的右键菜单也会拿到相同的 DataContext 的值。而我的业务是要右击打开下载项的文件夹或文件，此时的数据可以通过对应行的数据拿到

在 ContextMenu 的菜单里面需要绑定命令，而默认的命令不够好用，咱先磨一下刀，新建一个类，请看代码

```csharp
    public class DelegateCommand : ICommand
    {
        public Func<object, bool>? CanExecuteDelegate { set; get; }

        public Action<object>? ExecuteDelegate { set; get; }

        public bool CanExecute(object parameter)
        {
            return CanExecuteDelegate?.Invoke(parameter) ?? true;
        }

        public void Execute(object parameter)
        {
            ExecuteDelegate?.Invoke(parameter);
        }

        public event EventHandler? CanExecuteChanged;
    }
```

通过这个类就可以在 XAML 写绑定命令的资源和代码，请看代码

```xml
  <local:DelegateCommand x:Key="OpenFileCommand" ExecuteDelegate="OpenFileCommand_OnExecute" ></local:DelegateCommand>
  <local:DelegateCommand x:Key="OpenFolderCommand" ExecuteDelegate="OpenFolderCommand_OnExecute" ></local:DelegateCommand>
```

记得在 cs 代码里面添加对应的 `OpenFileCommand_OnExecute` 和 `OpenFolderCommand_OnExecute` 方法

```csharp
        private void OpenFileCommand_OnExecute(object parameter)
        {
            if (!(parameter is DownloadFileInfo downloadFileInfo))
            {
                return;
            }

            // 这里的 parameter 就是对应的右击 ListViewItem 的绑定数据
        }

        private void OpenFolderCommand_OnExecute(object parameter)
        {
            if (!(parameter is DownloadFileInfo downloadFileInfo))
            {
                return;
            }

            // 忽略代码
        }
```

可以看到对应的行的数据就是通过 parameter 参数传入到后台代码的方法，也就是通过命令的参数可以拿到当前右击的 ListViewItem 的数据

那么如何让命令拿到 DataContext 的参数？刚才咱也说到了右键菜单是放在 ListViewItem 的，而 DataContext 是会在视觉树继承的，所以右键菜单的 DataContext 和右击的行的是相同的

```xml
<MenuItem Header="Open File" Command="{StaticResource OpenFileCommand}" CommandParameter="{Binding}"></MenuItem>
```

通过 Command 绑定资源定义的命令，然后让 CommandParameter 命令参数使用 `{Binding}` 绑定到菜单的 DataContext 就可以将数据给到命令的参数，也就给到了后台代码的方法参数，所以后台代码就可以通过参数拿到右击所在行的数据

这样的代码就不需要去后台代码处理右击的事件，也不需要去找当前右键到哪一项，也不需要去找到对应的右击数据。通过绑定的方法和 DataContext 是视觉树继承的，就可以做到自动拿到当前的右击项的数据，传到后台方法

本文的更改放在 [github](https://github.com/dotnet-campus/dotnetCampus.FileDownloader/commit/5eab15a06a22d287b5622dba55315c0adae2f6e0) 上，小伙伴可以通过对比更改内容，就能知道本文修改的代码

如果在右击的本身是需要修改 ListViewItem 的界面的，如果这个界面更改和数据无关，那么可以通过修改 Style 的方法修改界面，而不是通过后台代码修改属性的方式

上面的代码在我实际的测试项目里面是存在一定的更改，本文的代码只是给大家演示，欢迎小伙伴参与开源项目 [https://github.com/dotnet-campus/dotnetCampus.FileDownloader](https://github.com/dotnet-campus/dotnetCampus.FileDownloader) 的开发

本文不属于入门博客，如果小伙伴还没入门，我推荐小伙伴看[豪哥的 bilibili 免费入门视频](https://space.bilibili.com/32497462)用项目带你入门 WPF 开发

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
