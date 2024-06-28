本文将通过阅读 lib x11 代码告诉大家，调用 XRootWindow 函数是不耗时的，没有成本的

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

在我阅读 Avalonia 和 CPF 和 UNO 框架的代码的时候，我发现了很多时候都是在需要用到 RootWindow 时，调用 XRootWindow 或 XDefaultRootWindow 获取 RootWindow 的值。此时我想着是否将 RootWindow 存放起来，这样可以稍微提升一点性能

在对某个函数调用进行性能测量考虑时，不仅可以使用基准性能测试工具进行测试，还可以通过阅读代码的方式了解实现原理从而了解其性能

通过阅读 lib x11 的代码，我发现了 XRootWindow 方法只是从结构体里面将值取出来，性能损耗其实和自己将 RootWindow 存起来可以认为是等价的

在 Macros.c 文件的对 XRootWindow 方法的定义代码如下

```csharp
Window XRootWindow (Display *dpy, int scr)
{
    return (RootWindow(dpy,scr));
}
```

以上代码的 RootWindow 是一个宏定义，定义在 Xlib.h 文件中，代码如下

```c
#define RootWindow(dpy, scr) 	(ScreenOfDisplay(dpy,scr)->root)
```

从以上的代码可以看到，实现就是将传入的 Display 和 screen number 传入到 ScreenOfDisplay 里面。最后取出来一个 root 字段

再继续看看 ScreenOfDisplay 这个宏的定义，代码如下

```c
#define ScreenOfDisplay(dpy, scr)(&((_XPrivDisplay)(dpy))->screens[scr])
```

可以看到实现的逻辑十分简单，那就是将传入的 Display 转换为 `_XPrivDisplay` 结构体类型。接着获取其 screens 字段，这个字段是一个数组，或者准确说是一个指向 Screen 数组类型的指针。再取其第 scr 项，也就是取传入的 screen number 项

回到咱 C# 代码，如以下的代码定义

```csharp
var display = XOpenDisplay(IntPtr.Zero);
var screen = XDefaultScreen(display);
```

以上代码拿到的 display 就是以上的 Display 的值，也就是实际的 `_XPrivDisplay` 结构体类型指针。以上的 screen 准确来说是 screen number 的意思，大部分情况下返回 0 的值

在咱 C# 代码调用 XRootWindow 方法时，如以下代码，其实等同于在 display 里面先取 Screen 再取其 root 字段

```csharp
var xRootWindow = XRootWindow(display, screen);
```

那这么说是否可以绕过 XRootWindow 方法，直接不安全使用 `_XPrivDisplay` 结构体类型指针获取 RootWindow 内容？答案是可以的

开始之前必须说明的是，这样的方式是不安全的，强依赖 xlib 的实现。好在这部分逻辑好久都没有变更了，大概在你的设备上，我以下的代码也能跑起来

先阅读 `_XPrivDisplay` 结构体，大概代码如下

```c
typedef struct
   #ifdef XLIB_ILLEGAL_ACCESS
   _XDisplay
   #endif
   {
    	XExtData *ext_data;	/* hook for extension to hang data * /
    	struct _XPrivate *private1;
    	int fd;			/* Network socket. * /
    	int private2;
    	int proto_major_version;/* major version of server's X protocol * /
    	int proto_minor_version;/* minor version of servers X protocol * /
    	char *vendor;		/* vendor of the server hardware * /
            XID private3;
    	XID private4;
    	XID private5;
    	int private6;
    	XID (*resource_alloc)(	/* allocator function * /
   		struct _XDisplay*
    	);
    	int byte_order;		/* screen byte order, LSBFirst, MSBFirst * /
    	int bitmap_unit;	/* padding and data requirements * /
    	int bitmap_pad;		/* padding requirements on bitmaps * /
    	int bitmap_bit_order;	/* LeastSignificant or MostSignificant * /
    	int nformats;		/* number of pixmap formats in list * /
    	ScreenFormat *pixmap_format;	/* pixmap format list * /
    	int private8;
    	int release;		/* release of the server * /
    	struct _XPrivate *private9, *private10;
    	int qlen;		/* Length of input event queue * /
    	unsigned long last_request_read; /* seq number of last event read * /
    	unsigned long request;	/* sequence number of last request. * /
    	XPointer private11;
    	XPointer private12;
    	XPointer private13;
    	XPointer private14;
    	unsigned max_request_size; /* maximum number 32 bit words in request* /
    	struct _XrmHashBucketRec *db;
    	int (*private15)(
    		struct _XDisplay*
    		);
    	char *display_name;	/* "host:display" string used on this connect* /
    	int default_screen;	/* default screen for operations * /
    	int nscreens;		/* number of screens on this server* /
    	Screen *screens;	/* pointer to list of screens * /
    	unsigned long motion_buffer;	/* size of motion buffer * /
    	unsigned long private16;
    	int min_keycode;	/* minimum defined keycode * /
    	int max_keycode;	/* maximum defined keycode * /
    	XPointer private17;
    	XPointer private18;
    	int private19;
    	char *xdefaults;	/* contents of defaults from server * /
    	/* there is more to this structure, but it is private to Xlib * /
   }
   #ifdef XLIB_ILLEGAL_ACCESS
   Display,
   #endif
   *_XPrivDisplay;
```

如上文，咱核心需要的就是拿到 `Screen *screens;	/* pointer to list of screens * /` 字段的内容

继续先看看 Screen 的定义，代码如下

```c
typedef struct {
   	XExtData *ext_data;	/* hook for extension to hang data * /
   	struct _XDisplay *display;/* back pointer to display structure * /
   	Window root;		/* Root window id. * /
   	int width, height;	/* width and height of screen * /
   	int mwidth, mheight;	/* width and height of  in millimeters * /
   	int ndepths;		/* number of depths possible * /
   	Depth *depths;		/* list of allowable depths on the screen * /
   	int root_depth;		/* bits per pixel * /
   	Visual *root_visual;	/* root visual * /
   	GC default_gc;		/* GC for the root root visual * /
   	Colormap cmap;		/* default color map * /
   	unsigned long white_pixel;
   	unsigned long black_pixel;	/* White and Black pixel values * /
   	int max_maps, min_maps;	/* max and min color maps * /
   	int backing_store;	/* Never, WhenMapped, Always * /
   	Bool save_unders;
   	long root_input_mask;	/* initial root input mask * /
   } Screen;
```

由于咱只想从 `_XPrivDisplay` 拿到 screens 字段，于是在 C# 代码定义里面，可以使用 StructLayout 加 Explicit 方式跳过结构体定义，只定义核心的字段

```csharp
[StructLayout(LayoutKind.Explicit)]
struct Display
{
    [FieldOffset(228)]
    public int nscreens;

    [FieldOffset(232)]
    public IntPtr Screens;
}
```

以上代码的 `[FieldOffset(228)]` 是根据上文的 `_XPrivDisplay` 算到的在 nscreens 字段在整个结构体里面的 byte 量，换句话说就是在第几个 byte 就是 nscreens 字段

同样对 Screen 结构体做相同的定义，只定义其中我需要用到的属性

```csharp
[StructLayout(LayoutKind.Explicit)]
struct Screen
{
    [FieldOffset(8)]
    public IntPtr _XDisplay;

    [FieldOffset(8 + 8)]
    public int RootWindow;

    [FieldOffset(8 + 8 + 8)]
    public int Width;
    [FieldOffset(8 + 8 + 8 + 4)]
    public int Height;
}
```

尝试通过自己定义的结构体和 XRootWindow 获取 RootWindow 的值，看是否相同，代码如下

```csharp
var display = XOpenDisplay(IntPtr.Zero);
var screen = XDefaultScreen(display);

var xRootWindow = XRootWindow(display, screen);

unsafe
{
    var pDisplay = (Display*)display;

    var firstScreen = pDisplay->Screens;

    Screen* pScreen = (Screen*) firstScreen;

    var rootWindowFromPScreen = pScreen->RootWindow;

    Console.WriteLine($"{rootWindowFromPScreen} xRootWindow={xRootWindow}");
}
```

尝试运行以上代码，可以在控制台里输出以下内容

```
1729 xRootWindow=1729
```

可以看到两个值是相同的，证明咱这个获取方式是正确的，也证明了 xlib 实现确实如此

如此也可以证明 XRootWindow 方法获取是不耗时没成本的，只是从结构体将值取出而已。不需要自己存着，自己存着和调用方法拿从业务角度没有性能上的差异

那 XDefaultRootWindow 呢？通过阅读 Macros.c 代码，可以发现和 XRootWindow 差不多，代码定义如下

```csharp
Window XRootWindow (Display *dpy, int scr)
{
    return (RootWindow(dpy,scr));
}

Window XDefaultRootWindow (Display *dpy)
{
    return (RootWindow(dpy,DefaultScreen(dpy)));
}
```

可以看到不同点仅仅只是 XDefaultRootWindow 传入 RootWindow 的 scr 是从 DefaultScreen 拿的

<!-- 如下面代码可以看到 DefaultScreen 的定义

```csharp
int XDefaultScreen(Display *dpy) { return (DefaultScreen(dpy)); }
``` -->

继续进入 Macros.c 代码，可以看到 DefaultScreen 宏的定义如下

```c
#define DefaultScreen(dpy) 	(((_XPrivDisplay)(dpy))->default_screen)
```

如以上代码可以看到 XDefaultScreen 也只是从 `_XPrivDisplay` 结构体取出 `default_screen` 字段而已，也可以忽略

如此即可了解到调用 XRootWindow 和 XDefaultRootWindow 都只是从 Display 结构体取出字段而已，可以在业务端随意调用，没有成本

本文以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7ad18fcc3b99003e0864e54e1ea6e696909b4b3b/X11/LabajaycolibearLuleacearewearjaykee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7ad18fcc3b99003e0864e54e1ea6e696909b4b3b/X11/LabajaycolibearLuleacearewearjaykee) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7ad18fcc3b99003e0864e54e1ea6e696909b4b3b
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7ad18fcc3b99003e0864e54e1ea6e696909b4b3b
```

获取代码之后，进入 X11/LabajaycolibearLuleacearewearjaykee 文件夹，即可获取到源代码
