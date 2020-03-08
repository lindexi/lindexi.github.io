# win10 uwp 解决 SerialDevice.FromIdAsync 返回空

<!--more-->
<!-- CreateTime:2019/6/23 11:54:04 -->


调用 SerialDevice.FromIdAsync 可能返回空，因为没有设置 package.appmanifest 可以使用端口

打开 package.appmanifest 文件添加下面代码

```csharp
    <Capabilities>
     <DeviceCapability Name="serialcommunication">
      <Device Id="any">
       <Function Type="name:serialPort" />
      </Device>
     </DeviceCapability>
    </Capabilities>
```

尝试使用特定的端口访问

```csharp
string aqs = SerialDevice.GetDeviceSelector("COM3");
DeviceInformationCollection dlist = await DeviceInformation.FindAllAsync(aqs);

if (dlist.Any())
{
    deviceId = dlist.First().Id;
}

using (SerialDevice serialPort = await SerialDevice.FromIdAsync(deviceId))
{

}
```

[https://stackoverflow.com/q/37505107/6116637](https://stackoverflow.com/q/37505107/6116637)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
