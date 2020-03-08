# WPF 好看的矢量图标

本文告诉大家一个好用的网站，里面提供很多好看的图标。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->
<!-- 标签：WPF，UI，xaml -->

本文介绍的网站是 [Xamalot](http://www.xamalot.com/ ) 里面有很多好看的图标。

例如我找到了一个好看的图标

![](http://image.acmx.xyz/lindexi%2F20185161419393396.jpg)

我只需要点击下面的下载就可以了

![](http://image.acmx.xyz/lindexi%2F20185161420355419.jpg)

推荐使用 Brush 的方式，或者直接 Canvas 的方式。有大神说，显示图片的性能会比显示 Brush Path 好，但是显示图片需要关注dpi等，而且作为静态的资源，实际上 Brush 占用内存和图片是不能直接相比的。

这里尝试使用 http://www.xamalot.com/asset/3577c503-c4a3-477a-8204-e33f6739c196 这个图标

点击了 Brush 然后点击 Download XAML Brush as text 可以看到有很多代码，把他复制放在 Grid 的资源，然后写另一个 Grid 引用他

```csharp
    <Grid>

        <Grid.Resources>
            <SolidColorBrush x:Key="Brush1" Color="#FF868A84" ></SolidColorBrush>
            <SolidColorBrush x:Key="Brush2" Color="#8EFFFFFF" ></SolidColorBrush>
            <DrawingBrush x:Key="xofficecalendar" Stretch="Uniform">
                <DrawingBrush.Drawing>
                    <DrawingGroup>
                        <DrawingGroup>
                            <DrawingGroup Transform="0.0210546087473631,0,0,0.0208675786852837,42.6749382019043,38.5018997192383">
                                <GeometryDrawing Geometry="F1M-1559.252,-150.697L-219.619,-150.697 -219.619,327.660 -1559.252,327.660z">
                                    <GeometryDrawing.Brush>
                                        <LinearGradientBrush StartPoint="-1051.935,-150.697" EndPoint="-1051.935,327.66" MappingMode="Absolute" SpreadMethod="Pad">
                                            <GradientStop Color="#00000000" Offset="0" ></GradientStop>
                                            <GradientStop Color="Black" Offset="0.5" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </LinearGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                                <GeometryDrawing Geometry="F1M-219.619,-150.680C-219.619,-150.680 -219.619,327.650 -219.619,327.650 -76.745,328.551 125.781,220.481 125.781,88.454 125.781,-43.572 -33.655,-150.680 -219.619,-150.680z">
                                    <GeometryDrawing.Brush>
                                        <RadialGradientBrush Center="605.714,486.648" RadiusX="117.143" RadiusY="117.143" GradientOrigin="605.714,486.648" MappingMode="Absolute" Transform="2.77438902854919,0,0,1.9697060585022,-1891.63305664063,-872.885375976563">
                                            <GradientStop Color="Black" Offset="0" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </RadialGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                                <GeometryDrawing Geometry="F1M-1559.252,-150.680C-1559.252,-150.680 -1559.252,327.650 -1559.252,327.650 -1702.127,328.551 -1904.653,220.481 -1904.653,88.454 -1904.653,-43.572 -1745.216,-150.680 -1559.252,-150.680z">
                                    <GeometryDrawing.Brush>
                                        <RadialGradientBrush Center="605.714,486.648" RadiusX="117.143" RadiusY="117.143" GradientOrigin="605.714,486.648" MappingMode="Absolute" Transform="-2.77438902854919,0,0,1.9697060585022,112.762298583984,-872.885375976563">
                                            <GradientStop Color="Black" Offset="0" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </RadialGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <GeometryDrawing Geometry="F1M7.190,30.619L6.958,41.500 41.036,41.500 40.504,28.828 7.190,30.619z">
                                <GeometryDrawing.Brush>
                                    <LinearGradientBrush StartPoint="25.283,37.548" EndPoint="25.283,39.751" MappingMode="Absolute" SpreadMethod="Pad">
                                        <GradientStop Color="#FF383936" Offset="0" ></GradientStop>
                                        <GradientStop Color="#FF555753" Offset="1" ></GradientStop>
                                    </LinearGradientBrush>
                                </GeometryDrawing.Brush>
                                <GeometryDrawing.Pen>
                                    <Pen Brush="#FF2E3436" Thickness="1" LineJoin="Round" MiterLimit="4" ></Pen>
                                </GeometryDrawing.Pen>
                            </GeometryDrawing>
                            <GeometryDrawing Brush="#3BFFFFFF" Geometry="F1M7.530,37.614L7.331,41.000 41.314,41.000 7.828,40.502 7.530,37.614z" ></GeometryDrawing>
                            <GeometryDrawing Brush="#FFCACACA" Geometry="F1M6.812,11.277L39.686,11.277 43.019,36.964 5.031,36.964 6.812,11.277z">
                                <GeometryDrawing.Pen>
                                    <Pen Brush="#FF686868" Thickness="1" MiterLimit="4" ></Pen>
                                </GeometryDrawing.Pen>
                            </GeometryDrawing>
                            <GeometryDrawing Geometry="F1M6.182,8.500L39.921,8.500 43.342,35.500 4.354,35.500 6.182,8.500z">
                                <GeometryDrawing.Brush>
                                    <LinearGradientBrush StartPoint="37.798,27.006" EndPoint="36.223,21.153" MappingMode="Absolute" SpreadMethod="Pad">
                                        <GradientStop Color="White" Offset="0" ></GradientStop>
                                        <GradientStop Color="#FFE6E6E6" Offset="1" ></GradientStop>
                                    </LinearGradientBrush>
                                </GeometryDrawing.Brush>
                                <GeometryDrawing.Pen>
                                    <Pen Brush="{StaticResource Brush1}" Thickness="1" MiterLimit="4" ></Pen>
                                </GeometryDrawing.Pen>
                            </GeometryDrawing>
                            <GeometryDrawing Geometry="F1M7.128,9.500L39.029,9.500 42.074,34.500 5.423,34.500 7.128,9.500z">
                                <GeometryDrawing.Pen>
                                    <Pen Brush="White" Thickness="1" MiterLimit="4" ></Pen>
                                </GeometryDrawing.Pen>
                            </GeometryDrawing>
                            <GeometryDrawing Brush="#35000000" Geometry="F1M42.811,35.039L42.090,29.152 35.827,35.062 42.811,35.039z" ></GeometryDrawing>
                            <DrawingGroup Transform="1.62031602859497,0,0,1.12674701213837,0.837298989295959,-7.68939590454102">
                                <GeometryDrawing Geometry="M15.645,17.904A1.812,3.668,0.000,1,1,12.021,17.904A1.812,3.668,0.000,1,1,15.645,17.904z">
                                    <GeometryDrawing.Brush>
                                        <RadialGradientBrush Center="13.833,17.904" RadiusX="1.812" RadiusY="1.812" GradientOrigin="13.833,17.904" MappingMode="Absolute" Transform="1,0,0,2.02438998222351,1.46367200641314E-16,-18.3404407501221">
                                            <GradientStop Color="Black" Offset="0" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </RadialGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="1.12598204612732,0,0,0.958072006702423,19.3195896148682,-5.28820180892944">
                                <GeometryDrawing Geometry="M15.645,17.904A1.812,3.668,0.000,1,1,12.021,17.904A1.812,3.668,0.000,1,1,15.645,17.904z">
                                    <GeometryDrawing.Brush>
                                        <RadialGradientBrush Center="13.833,17.904" RadiusX="1.812" RadiusY="1.812" GradientOrigin="13.833,17.904" MappingMode="Absolute" Transform="1,0,0,2.02438998222351,1.66255999581571E-14,-18.3404407501221">
                                            <GradientStop Color="Black" Offset="0" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </RadialGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="1.12598204612732,0,0,1.05445802211761,-4.56608915328979,-6.66031312942505">
                                <GeometryDrawing Geometry="M15.645,17.904A1.812,3.668,0.000,1,1,12.021,17.904A1.812,3.668,0.000,1,1,15.645,17.904z">
                                    <GeometryDrawing.Brush>
                                        <RadialGradientBrush Center="13.833,17.904" RadiusX="1.812" RadiusY="1.812" GradientOrigin="13.833,17.904" MappingMode="Absolute" Transform="1,0,0,2.02438998222351,-3.66395324521399E-15,-18.3404407501221">
                                            <GradientStop Color="Black" Offset="0" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </RadialGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <GeometryDrawing Geometry="F1M19.719,29.324L17.369,29.324 17.369,18.735 14.384,19.636 14.384,17.647 19.719,16.180 19.719,29.324 M32.023,29.324L23.968,29.324 23.968,27.441C26.600,24.778 28.093,23.260 28.446,22.889 28.805,22.512 29.091,22.117 29.303,21.705 29.521,21.286 29.630,20.832 29.630,20.343 29.630,19.778 29.444,19.324 29.073,18.982 28.708,18.634 28.196,18.461 27.536,18.461 27.106,18.461 26.632,18.576 26.114,18.805 25.596,19.029 25.128,19.345 24.710,19.751L24.710,17.373C25.870,16.702 26.968,16.366 28.004,16.366 29.235,16.366 30.198,16.696 30.893,17.356 31.593,18.010 31.944,18.914 31.944,20.069 31.944,20.782 31.744,21.504 31.343,22.235 30.949,22.966 30.351,23.702 29.550,24.445L26.432,27.353 32.023,27.353 32.023,29.324">
                                <GeometryDrawing.Brush>
                                    <LinearGradientBrush StartPoint="23.204,24.078" EndPoint="30.742,36.054" MappingMode="Absolute" SpreadMethod="Pad">
                                        <GradientStop Color="#FF2E3436" Offset="0" ></GradientStop>
                                        <GradientStop Color="#002E3436" Offset="1" ></GradientStop>
                                    </LinearGradientBrush>
                                </GeometryDrawing.Brush>
                            </GeometryDrawing>
                            <DrawingGroup Transform="1.18130004405975,0,0,1.1820479631424,-4.86464309692383,-6.42248916625977">
                                <GeometryDrawing Geometry="M14.584,16.711A1.591,1.591,0.000,1,1,11.402,16.711A1.591,1.591,0.000,1,1,14.584,16.711z">
                                    <GeometryDrawing.Pen>
                                        <Pen Thickness="0.846" MiterLimit="4">
                                            <Pen.Brush>
                                                <RadialGradientBrush Center="12.993,18.4" RadiusX="2.162" RadiusY="2.162" GradientOrigin="12.993,18.4" MappingMode="Absolute">
                                                    <GradientStop Color="White" Offset="0" ></GradientStop>
                                                    <GradientStop Color="Transparent" Offset="1" ></GradientStop>
                                                </RadialGradientBrush>
                                            </Pen.Brush>
                                        </Pen>
                                    </GeometryDrawing.Pen>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="0.875764012336731,0,0,0.876358985900879,-0.916889011859894,-1.31427001953125">
                                <GeometryDrawing Geometry="M14.584,16.711A1.591,1.591,0.000,1,1,11.402,16.711A1.591,1.591,0.000,1,1,14.584,16.711z">
                                    <GeometryDrawing.Brush>
                                        <RadialGradientBrush Center="12.887,16.771" RadiusX="1.591" RadiusY="1.591" GradientOrigin="12.887,16.771" MappingMode="Absolute" Transform="1.94444394111633,4.46604093022108E-15,-4.46604093022108E-15,1.94444394111633,-12.772120475769,-16.783899307251">
                                            <GradientStop Color="Black" Offset="0" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </RadialGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="0.875764012336731,0,0,0.876358985900879,23.0914096832275,-1.31427001953125">
                                <GeometryDrawing Geometry="M14.584,16.711A1.591,1.591,0.000,1,1,11.402,16.711A1.591,1.591,0.000,1,1,14.584,16.711z">
                                    <GeometryDrawing.Brush>
                                        <RadialGradientBrush Center="12.887,16.771" RadiusX="1.591" RadiusY="1.591" GradientOrigin="12.887,16.771" MappingMode="Absolute" Transform="1.94444394111633,4.46604093022108E-15,-4.46604093022108E-15,1.94444394111633,-12.772120475769,-16.783899307251">
                                            <GradientStop Color="Black" Offset="0" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </RadialGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="-0.783151984214783,0,0,1.12674701213837,45.8329086303711,-5.19961404800415">
                                <GeometryDrawing Geometry="M14.411,16.252A1.458,4.331,0.000,1,1,15.998,11.967">
                                    <GeometryDrawing.Pen>
                                        <Pen Thickness="1.597" StartLineCap="Round" EndLineCap="Round" DashCap="Round" MiterLimit="4">
                                            <Pen.Brush>
                                                <LinearGradientBrush StartPoint="10.998,11.929" EndPoint="14.54,13.486" MappingMode="Absolute" SpreadMethod="Pad">
                                                    <GradientStop Color="#FF999A98" Offset="0" ></GradientStop>
                                                    <GradientStop Color="#FFCACCC8" Offset="0.5" ></GradientStop>
                                                    <GradientStop Color="#FF616161" Offset="1" ></GradientStop>
                                                </LinearGradientBrush>
                                            </Pen.Brush>
                                        </Pen>
                                    </GeometryDrawing.Pen>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="1.12598204612732,0,0,1.12674701213837,-5.88420486450195,-5.19961404800415">
                                <GeometryDrawing Geometry="M14.411,16.252A1.458,4.331,0.000,1,1,15.998,11.967">
                                    <GeometryDrawing.Pen>
                                        <Pen Thickness="1.332" StartLineCap="Round" EndLineCap="Round" DashCap="Round" MiterLimit="4">
                                            <Pen.Brush>
                                                <LinearGradientBrush StartPoint="10.998,11.929" EndPoint="14.54,13.486" MappingMode="Absolute" SpreadMethod="Pad">
                                                    <GradientStop Color="#FF999A98" Offset="0" ></GradientStop>
                                                    <GradientStop Color="#FFCACCC8" Offset="0.5" ></GradientStop>
                                                    <GradientStop Color="#FF616161" Offset="1" ></GradientStop>
                                                </LinearGradientBrush>
                                            </Pen.Brush>
                                        </Pen>
                                    </GeometryDrawing.Pen>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <GeometryDrawing Brush="{StaticResource Brush1}" Geometry="M10.418,8.000L35.000,8.000 35.000,9.000 10.418,9.000z" ></GeometryDrawing>
                            <DrawingGroup Transform="0.875764012336731,0,0,0.876358985900879,11.6174201965332,-1.36672604084015">
                                <GeometryDrawing Geometry="M14.584,16.711A1.591,1.591,0.000,1,1,11.402,16.711A1.591,1.591,0.000,1,1,14.584,16.711z">
                                    <GeometryDrawing.Brush>
                                        <RadialGradientBrush Center="12.887,16.771" RadiusX="1.591" RadiusY="1.591" GradientOrigin="12.887,16.771" MappingMode="Absolute" Transform="1.94444394111633,4.46604093022108E-15,-4.46604093022108E-15,1.94444394111633,-12.772120475769,-16.783899307251">
                                            <GradientStop Color="Black" Offset="0" ></GradientStop>
                                            <GradientStop Color="#00000000" Offset="1" ></GradientStop>
                                        </RadialGradientBrush>
                                    </GeometryDrawing.Brush>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <GeometryDrawing Geometry="M21.818,3.341C21.818,2.772,22.280,2.310,22.849,2.310L23.109,2.310C23.679,2.310,24.140,2.772,24.140,3.341L24.140,12.758C24.140,13.327,23.679,13.789,23.109,13.789L22.849,13.789C22.280,13.789,21.818,13.327,21.818,12.758z">
                                <GeometryDrawing.Brush>
                                    <LinearGradientBrush StartPoint="22.979,3.12" EndPoint="22.979,12.134" MappingMode="Absolute" SpreadMethod="Pad">
                                        <GradientStop Color="#FF999A98" Offset="0" ></GradientStop>
                                        <GradientStop Color="#FFCACCC8" Offset="0.5" ></GradientStop>
                                        <GradientStop Color="#FF616161" Offset="1" ></GradientStop>
                                    </LinearGradientBrush>
                                </GeometryDrawing.Brush>
                            </GeometryDrawing>
                            <GeometryDrawing Geometry="F1M34.918,35.178C37.249,35.331 42.536,32.353 42.047,28.569 40.899,30.340 38.552,29.421 35.534,29.537 35.534,29.537 35.912,34.636 34.918,35.178z">
                                <GeometryDrawing.Brush>
                                    <LinearGradientBrush StartPoint="39.589,33.551" EndPoint="37.876,31.586" MappingMode="Absolute" SpreadMethod="Pad">
                                        <GradientStop Color="#FF7C7C7C" Offset="0" ></GradientStop>
                                        <GradientStop Color="#FFB8B8B8" Offset="1" ></GradientStop>
                                    </LinearGradientBrush>
                                </GeometryDrawing.Brush>
                                <GeometryDrawing.Pen>
                                    <Pen Brush="{StaticResource Brush1}" Thickness="1" MiterLimit="4" ></Pen>
                                </GeometryDrawing.Pen>
                            </GeometryDrawing>
                            <GeometryDrawing Geometry="F1M36.500,33.500C37.506,33.000 39.753,31.931 40.707,30.556 39.491,30.832 38.586,30.621 36.563,30.607 36.563,30.607 36.638,32.934 36.500,33.500z">
                                <GeometryDrawing.Pen>
                                    <Pen Thickness="1" MiterLimit="4">
                                        <Pen.Brush>
                                            <LinearGradientBrush StartPoint="37.886,30.747" EndPoint="38.454,31.587" MappingMode="Absolute" SpreadMethod="Pad">
                                                <GradientStop Color="White" Offset="0" ></GradientStop>
                                                <GradientStop Color="Transparent" Offset="1" ></GradientStop>
                                            </LinearGradientBrush>
                                        </Pen.Brush>
                                    </Pen>
                                </GeometryDrawing.Pen>
                            </GeometryDrawing>
                            <DrawingGroup Transform="0.592592000961304,0,0,0.592592000961304,9.0763988494873,1.51851999759674">
                                <GeometryDrawing Brush="{StaticResource Brush2}" Geometry="M23.813,5.031A0.844,0.844,0.000,1,1,22.125,5.031A0.844,0.844,0.000,1,1,23.813,5.031z" ></GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="0.629630029201508,0,0,0.629630029201508,-4.99306392669678,1.36342394351959">
                                <GeometryDrawing Brush="{StaticResource Brush2}" Geometry="M23.813,5.031A0.844,0.844,0.000,1,1,22.125,5.031A0.844,0.844,0.000,1,1,23.813,5.031z" ></GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="0.629630029201508,0,0,0.629630029201508,20.8819408416748,1.36342394351959">
                                <GeometryDrawing Brush="{StaticResource Brush2}" Geometry="M23.813,5.031A0.844,0.844,0.000,1,1,22.125,5.031A0.844,0.844,0.000,1,1,23.813,5.031z" ></GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="1.18130004405975,0,0,1.1820479631424,7.66440486907959,-6.42248916625977">
                                <GeometryDrawing Geometry="M14.584,16.711A1.591,1.591,0.000,1,1,11.402,16.711A1.591,1.591,0.000,1,1,14.584,16.711z">
                                    <GeometryDrawing.Pen>
                                        <Pen Thickness="0.846" MiterLimit="4">
                                            <Pen.Brush>
                                                <RadialGradientBrush Center="12.993,18.4" RadiusX="2.162" RadiusY="2.162" GradientOrigin="12.993,18.4" MappingMode="Absolute">
                                                    <GradientStop Color="White" Offset="0" ></GradientStop>
                                                    <GradientStop Color="Transparent" Offset="1" ></GradientStop>
                                                </RadialGradientBrush>
                                            </Pen.Brush>
                                        </Pen>
                                    </GeometryDrawing.Pen>
                                </GeometryDrawing>
                            </DrawingGroup>
                            <DrawingGroup Transform="1.18130004405975,0,0,1.1820479631424,19.1769905090332,-6.42248916625977">
                                <GeometryDrawing Geometry="M14.584,16.711A1.591,1.591,0.000,1,1,11.402,16.711A1.591,1.591,0.000,1,1,14.584,16.711z">
                                    <GeometryDrawing.Pen>
                                        <Pen Thickness="0.846" MiterLimit="4">
                                            <Pen.Brush>
                                                <RadialGradientBrush Center="12.993,18.4" RadiusX="2.162" RadiusY="2.162" GradientOrigin="12.993,18.4" MappingMode="Absolute">
                                                    <GradientStop Color="White" Offset="0" ></GradientStop>
                                                    <GradientStop Color="Transparent" Offset="1" ></GradientStop>
                                                </RadialGradientBrush>
                                            </Pen.Brush>
                                        </Pen>
                                    </GeometryDrawing.Pen>
                                </GeometryDrawing>
                            </DrawingGroup>
                        </DrawingGroup>
                    </DrawingGroup>
                </DrawingBrush.Drawing>
            </DrawingBrush>

        </Grid.Resources>

        <Grid Margin="10,10,10,50" Background="{StaticResource xofficecalendar}"></Grid>
    </Grid>

```

按下 F5 就可以看到下面的图片

![](http://image.acmx.xyz/lindexi%2F20185161425559526.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
