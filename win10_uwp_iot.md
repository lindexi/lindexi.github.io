# win10 uwp iot

这篇文章主要译：

https://msdn.microsoft.com/magazine/mt694090 有很多都是胡说，随便喷，但我不会理。
https://blogs.msdn.microsoft.com/lucian

今天的科技行业最常用的短语之一就是“物联网”，物联网可以让每个设备使用云而智能。使用云，设备可以分享数据和控制别的设备。我们可以远程控制相机，远程收集分析数据。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

虽然在MSDN杂志有很多文章说如何收集和分析数据，尚未有任何从硬件和布线的角度讨论。转IOT开发需要电子设计、电工、焊接等硬件。开发者一般都是活在虚拟的世界不太想去弄现实的。好像我就是这个样子，在二次元。当然这句话原文没有说。很多开发者难以知道自己该如何做硬件，跨接电缆和电阻。为了解决这问题，本文章介绍了win10 iot如何让开发者不使用硬件。不使用硬件是不可能，但是我们能让开发者很大的不理解硬件就做出好的。

可编程的硬件已经有了很久，在硬件写程序需要对硬件了解， Raspberry Pi 2 Model B可以运行win10，自然和我们电脑的win10不一样。win10系统可以在Iot的可以到[dev.windows.com/iot](https://developer.microsoft.com/zh-cn/windows/iot)下载， Raspberry Pi 2可以运行UWP。

这文章作者将会创建UWP在Raspberry Pi 2虽然这是一个使用天气API根据他闪灯。作者将会介绍Iot概念，如何使用C#。这里的作者自然不是我，作者：Frank La Vigne 。

## 检测霜
春天他们会有霜，我们想要检测如果有霜我们就告诉，告诉使用亮灯。除了软件我们还需要硬件。我需要Raspberry Pi 2 Model B ，MicroSD card，LED light，无焊料的电路试验板，很多线。

Raspberry Pi 2 Model B的介绍可以看下面的博客。我就发一张图，因为翻译不好。

![这里写图片描述](http://img.blog.csdn.net/20160423104230030)

MicroSD Card 可以做Raspberry Pi 2 Model B硬盘，里面安装系统，我们的uwp。因为作者现在有4G的MicroSD Card，就使用了，建议还是8G。MicroSD Card大小根据需求。

无焊料的电路试验板、线 连接Raspberry Pi 2组件。虽然我可以使用随意连接，最快的方式是无焊料的电路试验板。如同名称我不需要焊接。我需要把线连接，使用30行10列。注意列有两个五组，"a-e"，"f-j"，很容易就知道怎么弄。

Led 电阻 我将会连接led到Raspberry Pi 2 ，电压5V，会让LED坏，所以我们需要电阻。

以太网电缆,USB鼠标和键盘,HDMI监视器  Raspberry Pi 2有4个USB我们可以连接键盘，以太网，HDMI，我们可以把Raspberry Pi 2当做电脑。

下载windows10 可以在iot跑，可以到https://developer.microsoft.com/zh-CN/windows/iot/Downloads.htm#Win8

开始项目有点难，很多开发者移动代码不一定适合硬件，为了这个例子我做了一个很简单的LED闪动，实时从网上下载数据。需要下面这些硬件：led灯，无焊料的电路试验板，电缆。
 Raspberry Pi 2 Model B 有很多GPIO ，GPIO 是General Purpose Input/Output，如果想知道可以百度，当然我是觉得google才是比较好，具体翻墙我就不说，相信大家github host很快就可以干了百度。有一些接口是保留我们不能编程，那么不能使用的有标出来，可以看上面的图有哪些被引出不能够编程。

## 设计电路

Led需要的电路可以看下图，电流动从pin1，标着3.3V的见上图的接口，3.3V对Led太大，所以我们需要一个电阻，然后电流流GPIO 5，根据引脚，从上面图我们可以看到是pin29。这个接口是可以编程让led变得“smart“，通过接口高电压和低电压，led亮一下暗一下。

![这里写图片描述](http://img.blog.csdn.net/20160423141642815)

现在作者将会开发来显示上面图的让led暗一下亮一下，作者合并开关连接 Raspberry Pi 2 pin29。作者选择e列7行然后把led长的接到a列8行，短的到a列7行。这里翻译都是在胡说，因为我没有真的弄。

作者把电阻连在c列8行和c列15行，我把正极连在a列15行，负极连在pin1，可以看下面，我也不知道翻译是不是。

![这里写图片描述](http://img.blog.csdn.net/20160423143037852)

作者把Windows IoT Core安装在MicroSD card，插Raspberry Pi 2，连接。启动设备看到下面
![这里写图片描述](http://img.blog.csdn.net/20160423143232400)

硬件设置好，我们开始写软件，首先打开神器，新建一个iot。我们需要新建一个UWP项目，我们把它命名`WeatherBlink`
![这里写图片描述](http://img.blog.csdn.net/20160423143609919)
打开扩展` Windows IoT Extensions for the UWP`

在`MainPage.xaml.cs`，我们需要使用Windows.Devices.Gpio

```csharp
using Windows.Devices.Gpio;
```

我们可以很容易使用pin，下面是我们使用pin电压高，好像电压高不是翻译电压

```csharp
var gpioController = GpioController.GetDefault();
gpioPin = gpioController.OpenPin(5);
gpioPin.Write(GpioPinValue.High);
```

下面代码获取pin电压

```csharp
var currentPinValue = gpioPin.Read();
```

整个app需要 GPIO pins，我们把他写成员

```csharp
private GpioPin gpioPin;
private GpioPinValue gpioPinValue;
```

我们在构造

```csharp
private void InitializeGPIO()
{
  var gpioController = GpioController.GetDefault();
  gpioPin = gpioController.OpenPin(5);
  gpioPinValue = GpioPinValue.High;
  gpioPin.Write(gpioPinValue);
  gpioPin.SetDriveMode(GpioPinDriveMode.Output);
}
```

我们可以使用win10 全部控件，我感觉微软这个没有什么好。很多iot没有好看的界面，对于一个渣，没有界面实在不好，所以我们需要做一个界面。这些话都是我自己的，还没有去弄就觉得把重点放到了一个我们不用的。

我们弄一个简单的UI，如果我们能连接鼠标，使用压缩来更新天气。
![这里写图片描述](http://img.blog.csdn.net/20160423144525530)

作者需要下载天气信息从网上，我的天气可以从[openweathermap.org/api](http://openweathermap.org/api)接收的是json，温度是k，我们可以使用

```csharp
private async void LoadWeatherData()
{
  double minTempDouble = await GetMinTempForecast();
  // 38F/3.3C = 276.483 Kelvin
  if (minTempDouble <= 276.483)
  {
   Blink(500);
   txtStatus.Text = "Freeze Warning!"
  }
  else
  {
    Blink(2000);
    txtStatus.Text = "No freezing weather in forecast."
  }
}
```
如果天气不好我们就很多开始警报，看到的函数就是我们的警报

```csharp
private void Blink(int interval)
{
  blinkingTimer = new DispatcherTimer();
  blinkingTimer.Interval =
    TimeSpan.FromMilliseconds(interval);
  blinkingTimer.Tick += BlinkingTimer_Tick;
}
```

```csharp
private void BlinkingTimer_Tick(
  object sender, object e)
{
  var currentPinValue = gpioPin.Read();
  if (currentPinValue == GpioPinValue.High)
  {
    gpioPin.Write(GpioPinValue.Low);
  }
  else
  {
    gpioPin.Write(GpioPinValue.High);
  }
}
```
我们需要在PC部署，我们需要改变编译ARM
![这里写图片描述](http://img.blog.csdn.net/20160423145206754)
在运行选择远程，我们能看到
![这里写图片描述](http://img.blog.csdn.net/20160423145259361)

我们可以使用我的 Raspberry Pi 2，我听到有一些大神连接不了，如果连接不了自己写ip，一般可以，如果还是不可以不用找我我自己没有去

连接了我们就可以看到我的led在闪，我还没有去弄，作者没有图，我就没法弄一个，随便找一个觉得和这个不合

Iot是一个新的挑战，开发需要运行环境，需要电源和网，大多挑战来自于想要如何，如增加一个防风雨的室外场景的容器，我的iot需要显示，很多挑战决定我的代码。如果我的设备有4G网络，我需要考虑数据传输，他的意思是他需要钱，需要优化设备数据发送。

虽然我们的天气没有使用云，很多iot都是要网络，我们可以做一个简单的可以发送邮件的应用。Iot可以用很多地方，所以有空可以去玩。

代码：https://github.com/ms-iot/samples

中文好的博客：
http://edi.wang/post/2016/3/26/windows-10-iot-gy-30-light-sensor

http://edi.wang/post/2016/3/28/windows-10-iot-moisture-sensor-raspberry-pi3

http://edi.wang/post/2016/4/2/windows-10-iot-hc04-ultra-sonic-distance

http://edi.wang/post/2016/4/3/windows-10-iot-sound-light

http://edi.wang/post/2016/4/4/windows-10-iot-stepper-motor

http://edi.wang/post/2016/4/10/windows-10-iot-azure-remote-light






