# Xamarin.Forms 选取文件 让用户选择本地文件

在 Xamarin 中，使用文件存储或打开某个特定文件都是很常用的做法，而在跨平台中，每个平台都有自己的 IO 的坑。如何在 Xamarin.Froms 里面让用户可以选择打开哪个本文文件，需要照顾多个不同的平台的文件访问方式

<!--more-->
<!-- CreateTime:2020/2/23 16:43:08 -->

<!-- 发布 -->

在 Xamarin.Forms 右击管理 NuGet 程序包，搜寻 Xamarin.Plugin.FilePicker 进行安装，或在 csproj 上添加下面代码

```csharp
    <PackageReference Include="Xamarin.Plugin.FilePicker" Version="2.1.36-beta" />
```

如果不是让用户选取文件内容，那么在 Xamarin.Essentials 这个提供了 Xamarin 原生 API 交互的库就完全足够使用了

在界面上添加一个按钮，用来让用户选取文件内容

```xml
    <StackLayout>
        <Label x:Name="FileText" Margin="10,10,10,10"></Label>
        <Button HorizontalOptions="Center" Text="选取文件" Clicked="Button_OnClicked"></Button>
    </StackLayout>
```

在后台代码添加按钮点击时让用户选择文件的代码

```csharp
using Plugin.FilePicker;

        private async void Button_OnClicked(object sender, EventArgs e)
        {
            var pickFile = await CrossFilePicker.Current.PickFile();
            if (pickFile is null)
            {
                // 用户拒绝选择文件
            }
            else
            {
                FileText.Text = $@"选取文件路径 :{pickFile.FilePath}";
            }
        }
```

此时用户可以不选择文件，如果用户不选择文件，那么将拿到一个空值

![](http://image.acmx.xyz/lindexi%2FScreenshot_1582444733.png)

现在 CrossFilePicker 的 Open 和 Save 方法都过时了，请使用 `Xamarin.Essentials.FileSystem` 代替，或者用 `Xamarin.Essentials.ShareFile` 将文件分享给其他应用打开

这个项目所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a94fb1c9361e7a7ed80b4d7df2b587b2c0d57045/FecawjearwhalljearWugeweenere) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
