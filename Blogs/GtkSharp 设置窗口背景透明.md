本文告诉大家如何在 GTK Sharp 里面设置窗口背景透明

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

在 GTK 里面设置窗口背景透明十分简单，只需使用如下代码即可

```csharp
        this.AppPaintable = true;
        var screen = this.Screen;
        var visual = screen.RgbaVisual;
        if (visual is not null && screen.IsComposited)
        {
            this.Visual = visual;
        }
```

感谢 [walterlv](https://github.com/walterlv) 大佬提供此方法，我只是代为记录的工具人

上面代码一般是放在窗口的构造函数里面，如以下示例

```csharp
internal sealed class DemoWindow : global::Gtk.Window
{
    public DemoWindow() : base(global::Gtk.WindowType.Toplevel)
    {
        Title = "Walterlv Blank Gtk App";
        SetDefaultSize(800, 600);
        Add(new Area());

        this.AppPaintable = true;
        var screen = this.Screen;
        var visual = screen.RgbaVisual;
        if (visual is not null && screen.IsComposited)
        {
            this.Visual = visual;
        }
    }

    protected override bool OnDeleteEvent(Event evnt)
    {
        global::Gtk.Application.Quit();
        return base.OnDeleteEvent(evnt);
    }
}
```

如果你运行代码没有看到窗口背景透明，那可能是你的系统里面的桌面窗口合成管理不正确或没安装，请自行解决，如安装 [compiz](https://en.wikipedia.org/wiki/Compiz) 窗口合成管理器

本文代码放在 <https://github.com/walterlv/Walterlv.BlankUnoApp> 仓库里，欢迎大家拉取代码运行
