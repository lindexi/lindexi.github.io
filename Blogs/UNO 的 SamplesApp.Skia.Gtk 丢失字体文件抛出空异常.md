在运行 UNO 的 SamplesApp.Skia.Gtk 例子程序时，如果没有拷贝字体文件夹，导致字体丢失，将会在运行的时候抛出 NullReferenceException 空异常

<!--more-->


<!-- CreateTime:2023/9/20 15:07:49 -->

<!-- 发布 -->
<!-- 博客 -->

抛出的异常堆栈大概如下

```
>	Uno.UI.dll!Windows.UI.Xaml.Documents.Inline.GetFont(string name = "ms-appx:///Assets/Fonts/uno-fluentui-assets.ttf#Symbols", Windows.UI.Text.FontWeight weight = {Windows.UI.Text.FontWeight}, Windows.UI.Text.FontStretch stretch = Undefined, Windows.UI.Text.FontStyle style = Normal) line 152	C#
 	Uno.UI.dll!Windows.UI.Xaml.Documents.Inline..cctor.AnonymousMethod__23_0(string nm = "ms-appx:///Assets/Fonts/uno-fluentui-assets.ttf#Symbols", Windows.UI.Text.FontWeight wt = {Windows.UI.Text.FontWeight}, Windows.UI.Text.FontStretch wh = Undefined, Windows.UI.Text.FontStyle sl = Normal) line 26	C#
 	Uno.Foundation.dll!Uno.Extensions.FuncMemoizeExtensions.AsMemoized.AnonymousMethod__0(string arg1 = "ms-appx:///Assets/Fonts/uno-fluentui-assets.ttf#Symbols", Windows.UI.Text.FontWeight arg2 = {Windows.UI.Text.FontWeight}, Windows.UI.Text.FontStretch arg3 = Undefined, Windows.UI.Text.FontStyle arg4 = Normal) line 159	C#
 	Uno.UI.dll!Windows.UI.Xaml.Documents.Inline.FontInfo.get() line 53	C#
 	Uno.UI.dll!Windows.UI.Xaml.Documents.Run.GetSegments() line 30	C#
 	Uno.UI.dll!Windows.UI.Xaml.Documents.Run.Segments.get() line 21	C#
 	Uno.UI.dll!Windows.UI.Xaml.Documents.InlineCollection.Measure(Windows.Foundation.Size availableSize = 20.0x32.0) line 68	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.TextBlock.MeasureOverride(Windows.Foundation.Size availableSize = 20.0x32.0) line 37	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 20.0x32.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 20.0x32.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 20.0x32.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 20.0x32.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.TextBlock}, Windows.Foundation.Size availableSize = 20.0x32.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = 20.0x32.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 20.0x32.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 20.0x32.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 20.0x32.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 20.0x32.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.FontIcon}, Windows.Foundation.Size availableSize = 20.0x32.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.InnerMeasureOverride(Windows.Foundation.Size availableSize = 20.0x32.0) line 1095	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureOverride(Windows.Foundation.Size availableSize = 20.0x32.0) line 1045	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 20.0x49.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 20.0x49.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 20.0x49.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 20.0x49.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.Grid}, Windows.Foundation.Size availableSize = 20.0x49.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureCell(Windows.UI.Xaml.UIElement child = {Windows.UI.Xaml.Controls.Grid}, Windows.UI.Xaml.Controls.Grid.CellUnitTypes rowHeightTypes = Star, Windows.UI.Xaml.Controls.Grid.CellUnitTypes columnWidthTypes = Pixel, bool forceRowToInfinity = false, double rowSpacing = 0, double columnSpacing = 0) line 498	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureCellsGroup(int cellsHead = 0, int cellCount = 2, double rowSpacing = 0, double columnSpacing = 0, bool ignoreColumnDesiredSize = false, bool forceRowToInfinity = false, ref Windows.UI.Xaml.Controls.Grid.CellCacheStackVector cellCacheVector = {Windows.UI.Xaml.Controls.Grid.CellCacheStackVector}) line 373	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.InnerMeasureOverride(Windows.Foundation.Size availableSize = ∞x49.0) line 1326	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureOverride(Windows.Foundation.Size availableSize = ∞x49.0) line 1045	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = ∞x49.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = ∞x49.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = ∞x49.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = ∞x49.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.Grid}, Windows.Foundation.Size availableSize = ∞x49.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = ∞x49.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = ∞x49.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = ∞x49.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = ∞x49.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = ∞x49.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.CheckBox}, Windows.Foundation.Size availableSize = ∞x49.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureCell(Windows.UI.Xaml.UIElement child = {Windows.UI.Xaml.Controls.CheckBox}, Windows.UI.Xaml.Controls.Grid.CellUnitTypes rowHeightTypes = Star, Windows.UI.Xaml.Controls.Grid.CellUnitTypes columnWidthTypes = Auto, bool forceRowToInfinity = false, double rowSpacing = 0, double columnSpacing = 0) line 498	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureCellsGroup(int cellsHead = 8, int cellCount = 10, double rowSpacing = 0, double columnSpacing = 0, bool ignoreColumnDesiredSize = false, bool forceRowToInfinity = false, ref Windows.UI.Xaml.Controls.Grid.CellCacheStackVector cellCacheVector = {Windows.UI.Xaml.Controls.Grid.CellCacheStackVector}) line 373	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.InnerMeasureOverride(Windows.Foundation.Size availableSize = 300.0x50.0) line 1272	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x50.0) line 1045	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x∞) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x∞) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x∞) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x∞) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.Grid}, Windows.Foundation.Size availableSize = 300.0x∞) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureCell(Windows.UI.Xaml.UIElement child = {Windows.UI.Xaml.Controls.Grid}, Windows.UI.Xaml.Controls.Grid.CellUnitTypes rowHeightTypes = Auto, Windows.UI.Xaml.Controls.Grid.CellUnitTypes columnWidthTypes = Star, bool forceRowToInfinity = false, double rowSpacing = 0, double columnSpacing = 0) line 498	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureCellsGroup(int cellsHead = 0, int cellCount = 2, double rowSpacing = 0, double columnSpacing = 0, bool ignoreColumnDesiredSize = false, bool forceRowToInfinity = false, ref Windows.UI.Xaml.Controls.Grid.CellCacheStackVector cellCacheVector = {Windows.UI.Xaml.Controls.Grid.CellCacheStackVector}) line 373	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.InnerMeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 1294	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 1045	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.Grid}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Border.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 35	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.Border}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.InnerMeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 1095	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 1045	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.Grid}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureCell(Windows.UI.Xaml.UIElement child = {Windows.UI.Xaml.Controls.Grid}, Windows.UI.Xaml.Controls.Grid.CellUnitTypes rowHeightTypes = Star, Windows.UI.Xaml.Controls.Grid.CellUnitTypes columnWidthTypes = Star | Pixel, bool forceRowToInfinity = false, double rowSpacing = 0, double columnSpacing = 0) line 498	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureCellsGroup(int cellsHead = 0, int cellCount = 2, double rowSpacing = 0, double columnSpacing = 0, bool ignoreColumnDesiredSize = false, bool forceRowToInfinity = false, ref Windows.UI.Xaml.Controls.Grid.CellCacheStackVector cellCacheVector = {Windows.UI.Xaml.Controls.Grid.CellCacheStackVector}) line 373	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.InnerMeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 1326	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Grid.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 1045	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.Grid}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.SplitView.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 279	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.SplitView}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Uno.UI.Samples.Controls.SampleChooserControl}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {SamplesApp.MainPage}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.ContentPresenter.MeasureOverride(Windows.Foundation.Size size = 300.0x200.0) line 1100	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.ContentPresenter}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureElement(Windows.UI.Xaml.UIElement view = {Windows.UI.Xaml.Controls.Frame}, Windows.Foundation.Size availableSize = 300.0x200.0) line 308	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 228	C#
 	Uno.UI.dll!Windows.UI.Xaml.Controls.Border.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 35	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 184	C#
 	Uno.UI.dll!Uno.UI.Xaml.Core.RootVisual.MeasureOverride(Windows.Foundation.Size availableSize = 300.0x200.0) line 90	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.InnerMeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 99	C#
 	Uno.UI.dll!Windows.UI.Xaml.FrameworkElement.MeasureCore(Windows.Foundation.Size availableSize = 300.0x200.0) line 78	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.DoMeasure(Windows.Foundation.Size availableSize = 300.0x200.0) line 245	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.MeasureVisualTreeRoot(Windows.Foundation.Size availableSize = 300.0x200.0) line 199	C#
 	Uno.UI.dll!Windows.UI.Xaml.UIElement.Measure(Windows.Foundation.Size availableSize = 300.0x200.0) line 179	C#
 	Uno.UI.dll!Windows.UI.Xaml.XamlRoot.RunMeasureAndArrange() line 95	C#
 	Uno.UI.dll!Windows.UI.Xaml.XamlRoot.ScheduleInvalidateMeasureOrArrange.AnonymousMethod__34_0() line 72	C#
 	Uno.UI.Dispatching.dll!Uno.UI.Dispatching.CoreDispatcher.InvokeOperation(Uno.UI.Dispatching.UIAsyncOperation operation = {Uno.UI.Dispatching.UIAsyncOperation}) line 373	C#
 	Uno.UI.Dispatching.dll!Uno.UI.Dispatching.CoreDispatcher.InvokeOperationSafe(Uno.UI.Dispatching.UIAsyncOperation operation = {Uno.UI.Dispatching.UIAsyncOperation}) line 337	C#
 	Uno.UI.Dispatching.dll!Uno.UI.Dispatching.CoreDispatcher.DispatchItems() line 299	C#
 	Uno.UI.Dispatching.dll!Uno.UI.Dispatching.CoreDispatcher.EnqueueNative.AnonymousMethod__41_0() line 40	C#
 	Uno.UI.Runtime.Skia.Gtk.dll!Uno.UI.Runtime.Skia.GtkHost.Run.AnonymousMethod__19() line 139	C#
 	GLibSharp.dll!GLib.Idle.IdleProxy.Handler()	
 	GtkSharp.dll!Gtk.Application.Run()	
 	Uno.UI.Runtime.Skia.Gtk.dll!Uno.UI.Runtime.Skia.GtkHost.Run() line 203	C#
 	SamplesApp.Skia.Gtk.dll!SkiaSharpExample.MainClass.Main(string[] args = {string[0]}) line 24	C#
```

此问题已在新的 UNO 版本修复，需要更新代码就可以自动修复
