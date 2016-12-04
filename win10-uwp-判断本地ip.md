# win10 uwp 判断本地ip
【】
本文主要：如何判断一个IP是本地IP

<!--more-->

对于本地`127.0.0.1`就是一个内部IP，之外，还有` 10.0.0.0/24`，`172.16.0.0/16`，
` 192.168.0.0/16`，`169.254.0.0/16`

判断是不是本地，首先判断是不是`127.0.0.1`

```
        private bool IsPrivateIP(IPAddress myIPAddress)
        {
            if (IPAddress.IsLoopback(myIPAddress))
            {
                return true;
            }
        }

```

判断是不是`10.0.0.0/24`

                byte[] ipBytes = myIPAddress.GetAddressBytes();
                // 10.0.0.0/24 
                if (ipBytes[0] == 10)
                {
                    return true;
                }

判断`172.16.0.0/16`

```
                if (ipBytes[0] == 172 && ipBytes[1] == 16)
                {
                    return true;
                }

```

判断`192.168.0.0/16`

```
                if (ipBytes[0] == 192 && ipBytes[1] == 168)
                {
                    return true;
                }

```

判断`169.254.0.0/16`

```
                if (ipBytes[0] == 169 && ipBytes[1] == 254)
                {
                    return true;
                }

```

源代码：

```
        /// <summary>
        /// 判断私有ip
        /// </summary>
        /// <param name="myIPAddress"></param>
        /// <returns></returns>
        private bool IsPrivateIP(IPAddress myIPAddress)
        {
            if (IPAddress.IsLoopback(myIPAddress))
            {
                return true;
            }
            if (myIPAddress.AddressFamily == System.Net.Sockets.AddressFamily.InterNetwork)
            {
                byte[] ipBytes = myIPAddress.GetAddressBytes();
                // 10.0.0.0/24 
                if (ipBytes[0] == 10)
                {
                    return true;
                }
                // 172.16.0.0/16
                else if (ipBytes[0] == 172 && ipBytes[1] == 16)
                {
                    return true;
                }
                // 192.168.0.0/16
                else if (ipBytes[0] == 192 && ipBytes[1] == 168)
                {
                    return true;
                }
                // 169.254.0.0/16
                else if (ipBytes[0] == 169 && ipBytes[1] == 254)
                {
                    return true;
                }
            }
            return false;
        }

```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。








