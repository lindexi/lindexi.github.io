# WPF 修复 ContextMenu 在开启 PerMonitorV2 后所用 DPI 错误

本文告诉大家如何修复 WPF 的 ContextMenu 在开启 PerMonitorV2 之后，在双屏不同的 DPI 的设备上，在副屏弹出的 ContextMenu 使用了主屏的 DPI 导致缩放错误的问题

<!--more-->
<!-- CreateTime:2022/8/10 20:06:40 -->

<!-- 发布 -->

关于什么是 PerMonitorV2 请参阅 [支持 Windows 10 最新 PerMonitorV2 特性的 WPF 多屏高 DPI 应用开发 - walterlv](https://blog.walterlv.com/post/windows-high-dpi-development-for-wpf.html )

开启 PerMonitorV2 的 WPF 应用的 ContextMenu 将在多屏下，需要找到一个关联的屏幕来辅助计算所要显示的坐标。然而在 ContextMenu 创建出来时，是无法知道应该选用哪个屏幕。这就是导致 ContextMenu 视觉效果的 DPI 缩放不对的原因

修复方法就是给 ContextMenu 一个参考的控件，通过此参考控件，可以让 ContextMenu 进行多屏幕不同的 DPI 计算。给 ContextMenu 一个参考的控件的方法有两个

第一个方法是通过将 ContextMenu 设置给所要关联的控件的 ContextMenu 属性上，如此即可让 ContextMenu 弹出的坐标可以根据此关联控件计算。要求关联的控件是在界面布局

```csharp
            var menu = new ContextMenu
            {
                Name = menuName,
                Style = contextMenuStyle,
                ItemsSource = menuItems,
            };
            canvas.ContextMenu = menu;
```

但是以上方法存在缺点，那就是对相同的业务逻辑，在 ContextMenu 关闭之前重新赋值，将存在重入问题，重入问题也许导致了某个过程的 ContextMenu 依然由于找不到关联的控件，弹出在左上角。例如以上代码被快速进入两次，第一次的 ContextMenu 对象还没完成弹出，第二次就进入，第二次的 ContextMenu 将会覆盖 `canvas` 的 ContextMenu 属性，从而让第一次的 ContextMenu 找不到关联的控件，让第一次的 ContextMenu 弹出到左上角，或者计算 DPI 不对 

如果采用第一个方法，可以通过缓存 ContextMenu 的方式，代替每次都创建。或者判断当前正在准备弹出 ContextMenu 就不继续创建

第二个方法是设置 ContextMenu 的 PlacementTarget 属性，通过此属性可以让 ContextMenu 关联控件，如以下代码

```csharp
            var menu = new ContextMenu
            {
                Name = menuName,
                Style = contextMenuStyle,
                ItemsSource = menuItems,
                // Popup 内部不处理显示过程中的 DPI 改变，依赖于创建时要能找到正确的屏幕，
                // 如果什么都不指定，那么创建会创建到主屏，如果实际显示在副屏了，那就会因为 DPI 缩放导致尺寸不对。
                // 
                // 寻找创建时的屏幕时，会寻找 PlacementTarget 和 VisualTreeHelper.GetContainingVisual2D(VisualTreeHelper.GetParent(this))，
                // 这里通过指定 PlacementTarget 确保创建的屏幕正确
                PlacementTarget = canvas,
            };
```