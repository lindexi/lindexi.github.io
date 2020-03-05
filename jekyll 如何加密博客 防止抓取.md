# jekyll 如何加密博客 防止抓取

经常会发现自己的博客被一些垃圾网站抓取，我就在博客进行加密，在访问的时候进行解密，于是爬虫如果不执行js就无法获得内容。

本文告诉大家如何加密博客。

<!--more-->
<!-- CreateTime:2019/1/29 16:26:17 -->


加密使用把文章内容转换为 Html 之后转换为 base64 然后在加载完成之后把 base64 转换为 html ，这个方式就可以解密文章。

文章的摘要可以不加密，但是文章内容可以使用这方法进行加密。

我使用 Pandoc 转换 html ，推荐使用这个方法。然后把我的文章转换得到的 html 在 [base64 转图片 在线解码编码](http://base64.xpcha.com/ )转换得到 base64 。然后把这些代码放在一个 div 内，在页面加载完成就把他转换为 html 

我下面会把 js 放在文章最后，大家可以复制去自己博客使用，使用只需要把转换 html 后的代码放在下面的 div 里

```csharp
<div class="src">
base64
</div>
```

在页面加载完成就会把 base64 转换为 html 。

这时会发现，打开页面看到的是base64，所以可以先把他隐藏，设置 css 隐藏 src ，请看下面的代码

```csharp
.src
{
    display: none;
}
```


关键 js 代码

```csharp
      $(document).ready(function() 
       {
             var src = document.getElementsByClassName('src');
             for (var i = 0; i < src.length; i++) 
             {
                 src[i].innerHTML = utf8to16(base64decode(src[i].innerText));
                 src[i].style.display = "inline";
             }
         });
```

但是很多爬虫还是会执行一下代码，于是他还是可以获得源代码，如何让他无法获得源代码？实际上我还想到另一个方法，先把 html 转换为 二进制， 然后从 0-1000 选一个数作为密码，对他加密。得到的内容转换为 base64 然后放在上面的 div 这样在页面加载完成之后，执行代码，尝试从 0-1000 进行解密，于是就可以获得一个正确 html 这样就可以让js执行时间变长，一般的垃圾网站不会让自己的网站爬这样的文章。

1. 转换 html

1. html 转 二进制

1. 随机从 0-1000 选一个数字

1. 把 "lindexi" 转二进制，并且把他进行加密，加密的密码就是上面选的数字。

1. 把 html 转换得到的二进制进行加密

1. 把上面加密的内容转换为 base64 放在博客里

打开网页时的算法

1. 获得从 lindexi 转换二进制加密的 base64 从 0-1000 选数字，看哪个数字可以解密得到 "lindexi"

1. 从上一步拿到的数字对 html 加密后的二进制进行解密

1. 显示解密后的html

我想的这个算法感觉比较好，但是不会写，不知道有哪位大神可以帮我写。

当然这样也可以做博客加密，就是把随机选一个数字变为自己设置一个字符串。然后打开网页弹出窗口输入字符串解密。

全部代码

```csharp
var base64EncodeChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
var base64DecodeChars = new Array(
    -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
    -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
    -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 62, -1, -1, -1, 63,
    52, 53, 54, 55, 56, 57, 58, 59, 60, 61, -1, -1, -1, -1, -1, -1,
    -1,  0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14,
    15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, -1, -1, -1, -1, -1,
    -1, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
    41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, -1, -1, -1, -1, -1);

function base64encode(str) {
    var out, i, len;
    var c1, c2, c3;

    len = str.length;
    i = 0;
    out = "";
    while(i < len) {
    c1 = str.charCodeAt(i++) & 0xff;
    if(i == len)
    {
        out += base64EncodeChars.charAt(c1 >> 2);
        out += base64EncodeChars.charAt((c1 & 0x3) << 4);
        out += "==";
        break;
    }
    c2 = str.charCodeAt(i++);
    if(i == len)
    {
        out += base64EncodeChars.charAt(c1 >> 2);
        out += base64EncodeChars.charAt(((c1 & 0x3)<< 4) | ((c2 & 0xF0) >> 4));
        out += base64EncodeChars.charAt((c2 & 0xF) << 2);
        out += "=";
        break;
    }
    c3 = str.charCodeAt(i++);
    out += base64EncodeChars.charAt(c1 >> 2);
    out += base64EncodeChars.charAt(((c1 & 0x3)<< 4) | ((c2 & 0xF0) >> 4));
    out += base64EncodeChars.charAt(((c2 & 0xF) << 2) | ((c3 & 0xC0) >>6));
    out += base64EncodeChars.charAt(c3 & 0x3F);
    }
    return out;
}

function base64decode(str) {
    var c1, c2, c3, c4;
    var i, len, out;

    len = str.length;
    i = 0;
    out = "";
    while(i < len) {
    /* c1 */
    do {
        c1 = base64DecodeChars[str.charCodeAt(i++) & 0xff];
    } while(i < len && c1 == -1);
    if(c1 == -1)
        break;

    /* c2 */
    do {
        c2 = base64DecodeChars[str.charCodeAt(i++) & 0xff];
    } while(i < len && c2 == -1);
    if(c2 == -1)
        break;

    out += String.fromCharCode((c1 << 2) | ((c2 & 0x30) >> 4));

    /* c3 */
    do {
        c3 = str.charCodeAt(i++) & 0xff;
        if(c3 == 61)
        return out;
        c3 = base64DecodeChars[c3];
    } while(i < len && c3 == -1);
    if(c3 == -1)
        break;

    out += String.fromCharCode(((c2 & 0XF) << 4) | ((c3 & 0x3C) >> 2));

    /* c4 */
    do {
        c4 = str.charCodeAt(i++) & 0xff;
        if(c4 == 61)
        return out;
        c4 = base64DecodeChars[c4];
    } while(i < len && c4 == -1);
    if(c4 == -1)
        break;
    out += String.fromCharCode(((c3 & 0x03) << 6) | c4);
    }
    return out;
}

function utf16to8(str) {
    var out, i, len, c;

    out = "";
    len = str.length;
    for(i = 0; i < len; i++) {
    c = str.charCodeAt(i);
    if ((c >= 0x0001) && (c <= 0x007F)) {
        out += str.charAt(i);
    } else if (c > 0x07FF) {
        out += String.fromCharCode(0xE0 | ((c >> 12) & 0x0F));
        out += String.fromCharCode(0x80 | ((c >>  6) & 0x3F));
        out += String.fromCharCode(0x80 | ((c >>  0) & 0x3F));
    } else {
        out += String.fromCharCode(0xC0 | ((c >>  6) & 0x1F));
        out += String.fromCharCode(0x80 | ((c >>  0) & 0x3F));
    }
    }
    return out;
}

function utf8to16(str) {
    var out, i, len, c;
    var char2, char3;

    out = "";
    len = str.length;
    i = 0;
    while(i < len) {
    c = str.charCodeAt(i++);
    switch(c >> 4)
    { 
      case 0: case 1: case 2: case 3: case 4: case 5: case 6: case 7:
        // 0xxxxxxx
        out += str.charAt(i-1);
        break;
      case 12: case 13:
        // 110x xxxx   10xx xxxx
        char2 = str.charCodeAt(i++);
        out += String.fromCharCode(((c & 0x1F) << 6) | (char2 & 0x3F));
        break;
      case 14:
        // 1110 xxxx  10xx xxxx  10xx xxxx
        char2 = str.charCodeAt(i++);
        char3 = str.charCodeAt(i++);
        out += String.fromCharCode(((c & 0x0F) << 12) |
                       ((char2 & 0x3F) << 6) |
                       ((char3 & 0x3F) << 0));
        break;
    }
    }

    return out;
}

function CharToHex(str) {
    var out, i, len, c, h;
    out = "";
    len = str.length;
    i = 0;
    while(i < len) 
    {
	    c = str.charCodeAt(i++);
	    h = c.toString(16);
	    if(h.length < 2)
	    	h = "0" + h;
	    
	    out += "\\x" + h + " ";
	    if(i > 0 && i % 8 == 0)
	    	out += "\r\n";
    }

    return out;
}

function doEncode() {
	var src = document.getElementById('src').value;
	document.getElementById('dest').value = base64encode(utf16to8(src));
}

function doDecode() {
	var src = document.getElementById('src').value;
	var opts = document.getElementById('opt');

	if(opts.checked)
	{
		document.getElementById('dest').value = CharToHex(base64decode(src));
	}
	else
	{
		document.getElementById('dest').value = utf8to16(base64decode(src));
	}
}

       $(document).ready(function() 
       {
             var src = document.getElementsByClassName('src');
             for (var i = 0; i < src.length; i++) 
             {
                 src[i].innerHTML = utf8to16(base64decode(src[i].innerText));
                 src[i].style.display = "inline";
             }
         });



```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。