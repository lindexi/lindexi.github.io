本文记录使用 PulseAudio 在 Linux 系统上进行设置和获取当前音量，以及是否静音。当系统音量发生变更时，收到事件通知

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文使用的工具类由 [lsj](https://blog.sdlsj.net) 工具人提供，我只是代为记录

演示的使用方法的代码如下

```csharp
if (!OperatingSystem.IsLinux())
{
    return;
}

var pulseAudioVolumeManager = new PulseAudioVolumeManager();
await pulseAudioVolumeManager.Init();

pulseAudioVolumeManager.VolumeChanged += (sender, volume) =>
{
    Console.WriteLine($"音量变化，当前音量：{volume}");
};

pulseAudioVolumeManager.MuteChanged += (sender, isMute) =>
{
    Console.WriteLine($"静音变化，当前是否静音：{isMute}");
};

while (true)
{
    Console.WriteLine($"是否静音：{await pulseAudioVolumeManager.GetMute()}; 音量：{await pulseAudioVolumeManager.GetVolume()}");

    Console.WriteLine($"输入数字修改音量，输入 y/n 设置是否静音");
    var line = Console.ReadLine();
    if (int.TryParse(line, out var n))
    {
        Console.WriteLine($"设置音量为：{n}");
        await pulseAudioVolumeManager.SetVolume(n);
    }
    else if(line is not null)
    {
        var text = line.ToLowerInvariant();
        if (text == "y")
        {
            Console.WriteLine($"设置是否静音：是");
            await pulseAudioVolumeManager.SetMute(true);
        }
        else if (text == "n")
        {
            Console.WriteLine($"设置是否静音：否");
            await pulseAudioVolumeManager.SetMute(false);
        }
    }
}
```

此代码是完全 C# dotnet 系列的，意味着不挑 UI 框架，可以在 Avalonia 或 UNO 或 CPF 等上层 UI 框架里使用

以上代码用到的 PulseAudioVolumeManager 封装代码如下

```csharp
    /// <summary>
    /// “脉冲”音量管理，这是基于 PulseAudio 的封装
    /// </summary>
    [SupportedOSPlatform("linux")]
    public partial class PulseAudioVolumeManager
    {
        private TaskCompletionSource<bool>? _initTaskCompletionSource;
        private IntPtr _mainLoop;
        private IntPtr _context;
        private readonly pa_context_subscribe_cb_t _contextSubscribeCallback;

        private int? _volume;
        private bool? _mute;

        private readonly string _applicationName;

        public event EventHandler<int>? VolumeChanged;
        public event EventHandler<bool>? MuteChanged;

        /// <summary>
        /// 创建“脉冲”音量管理
        /// </summary>
        /// <param name="applicationName">应用名，可选，只是用于调用 pa_context_new 时传入，无特别含义和作用</param>
        public PulseAudioVolumeManager(string? applicationName = null)
        {
            _applicationName = applicationName ?? Path.GetRandomFileName().Replace('.', '_');
            _contextSubscribeCallback = ContextSubscribeCallback;
        }

        public async Task<bool> Init()
        {
            if (_initTaskCompletionSource == null)
            {
                bool isReady = false;
                _initTaskCompletionSource = new TaskCompletionSource<bool>();

                _mainLoop = pa_threaded_mainloop_new();
                if (_mainLoop != IntPtr.Zero)
                {
                    var mainloopApi = pa_threaded_mainloop_get_api(_mainLoop);
                    if (mainloopApi != IntPtr.Zero)
                    {
                        var context = pa_context_new(mainloopApi, _applicationName);
                        if (context != IntPtr.Zero)
                        {
                            pa_context_set_state_callback(context, ContextStateCallback, IntPtr.Zero);

                            await Task.Run(() =>
                            {
                                var result = pa_context_connect(context, IntPtr.Zero, 0, IntPtr.Zero);
                                if (result < 0)
                                {
                                    return;
                                }

                                result = pa_threaded_mainloop_start(_mainLoop);
                                if (result < 0)
                                {
                                    return;
                                }

                                while (true)
                                {
                                    var state = pa_context_get_state(context);

                                    if (state == pa_context_state_t.PA_CONTEXT_READY)
                                    {
                                        isReady = true;
                                        _context = context;
                                        break;
                                    }

                                    if (!PA_CONTEXT_IS_GOOD(state))
                                    {
                                        return;
                                    }

                                    pa_threaded_mainloop_wait(_mainLoop);
                                }
                            });
                        }
                    }
                }
                _initTaskCompletionSource.SetResult(isReady);
                return isReady;
            }
            else
            {
                return await _initTaskCompletionSource.Task;
            }
        }

        private void ContextStateCallback(IntPtr c, IntPtr userdata)
        {
            switch (pa_context_get_state(c))
            {
                case pa_context_state_t.PA_CONTEXT_READY:
                    pa_context_set_subscribe_callback(c, _contextSubscribeCallback, IntPtr.Zero);
                    var op = pa_context_subscribe(c, pa_subscription_mask_t.PA_SUBSCRIPTION_MASK_SINK, IntPtr.Zero, IntPtr.Zero);
                    pa_operation_unref(op);
                    pa_threaded_mainloop_signal(_mainLoop, 0);
                    break;

                case pa_context_state_t.PA_CONTEXT_TERMINATED:
                case pa_context_state_t.PA_CONTEXT_FAILED:
                    pa_threaded_mainloop_signal(_mainLoop, 0);
                    break;
            }
        }

        private void ContextSubscribeCallback(IntPtr c, pa_subscription_event_type_t t, uint idx, IntPtr userdata)
        {
            if ((t & pa_subscription_event_type_t.PA_SUBSCRIPTION_EVENT_FACILITY_MASK) == pa_subscription_event_type_t.PA_SUBSCRIPTION_EVENT_SINK)
            {
                Task.Run(() =>
                {
                    int? volume = null;
                    bool? mute = null;

                    pa_threaded_mainloop_lock(_mainLoop);

                    var sinkName = GetDefaultSinkName(c);
                    if (sinkName != null)
                    {
                        var info = GetSinkInfo(c, sinkName);
                        volume = GetVolumeValue(info.volume);
                        mute = info.mute;
                    }

                    pa_threaded_mainloop_unlock(_mainLoop);

                    if (volume is int volumeVal && volumeVal != _volume)
                    {
                        _volume = volumeVal;
                        VolumeChanged?.Invoke(this, volumeVal);
                    }
                    if (mute is bool muteVal && muteVal != _mute)
                    {
                        _mute = muteVal;
                        MuteChanged?.Invoke(this, muteVal);
                    }
                });
            }
        }

        private void ServerInfoCallback(IntPtr c, in pa_server_info i, IntPtr userdata)
        {
            unsafe
            {
#pragma warning disable CS8500
                *(string*) userdata = Marshal.PtrToStringUTF8(i.default_sink_name);
#pragma warning restore CS8500
                pa_threaded_mainloop_signal(_mainLoop, 0);
            }
        }

        private void SinkInfoCallback(IntPtr c, in pa_sink_info i, int eol, IntPtr userdata)
        {
            if (eol != 0)
            {
                pa_threaded_mainloop_signal(_mainLoop, 0);
                return;
            }

            unsafe
            {
                *((pa_cvolume, bool)*) userdata = (i.volume, i.mute != 0);
            }
        }

        public async Task<bool> GetMute()
        {
            bool result = false;
            if (_context != IntPtr.Zero)
            {
                await Task.Run(() =>
                {
                    pa_threaded_mainloop_lock(_mainLoop);

                    var sinkName = GetDefaultSinkName(_context);
                    if (sinkName != null)
                    {
                        var (volume, mute) = GetSinkInfo(_context, sinkName);
                        result = mute;
                    }

                    pa_threaded_mainloop_unlock(_mainLoop);
                });
            }
            return result;
        }

        public async Task<int> GetVolume()
        {
            int result = 50;
            if (_context != IntPtr.Zero)
            {
                await Task.Run(() =>
                {
                    pa_threaded_mainloop_lock(_mainLoop);

                    var sinkName = GetDefaultSinkName(_context);
                    if (sinkName != null)
                    {
                        var (volume, mute) = GetSinkInfo(_context, sinkName);
                        result = GetVolumeValue(volume);
                    }

                    pa_threaded_mainloop_unlock(_mainLoop);
                });
            }
            return result;
        }

        public async Task SetMute(bool mute)
        {
            if (_context != IntPtr.Zero)
            {
                await Task.Run(() =>
                {
                    pa_threaded_mainloop_lock(_mainLoop);

                    var sinkName = GetDefaultSinkName(_context);
                    if (sinkName != null)
                    {
                        pa_context_set_sink_mute_by_name(_context, sinkName, mute ? 1 : 0, IntPtr.Zero, IntPtr.Zero);
                    }

                    pa_threaded_mainloop_unlock(_mainLoop);
                });
            }
        }

        public async Task SetVolume(int volume)
        {
            if (_context != IntPtr.Zero)
            {
                await Task.Run(() =>
                {
                    pa_threaded_mainloop_lock(_mainLoop);

                    var sinkName = GetDefaultSinkName(_context);
                    if (sinkName != null)
                    {
                        var info = GetSinkInfo(_context, sinkName);

                        pa_cvolume_set(ref info.volume, info.volume.channels, (uint) (volume * 65536 / 100));

                        pa_context_set_sink_volume_by_name(_context, sinkName, info.volume, IntPtr.Zero, IntPtr.Zero);
                    }

                    pa_threaded_mainloop_unlock(_mainLoop);
                });
            }
        }

        /// <summary>
        /// 获取默认的输出设备名称
        /// 这个方法要在 pa_threaded_mainloop_lock 和 pa_threaded_mainloop_unlock 之间调用
        /// </summary>
        /// <returns></returns>
        private string? GetDefaultSinkName(IntPtr context)
        {
            unsafe
            {
                string? sinkName = null;

                // 取 sinkName 地址，相当于 ref string 用法，在 ServerInfoCallback 给 sinkName 赋值
                var op = pa_context_get_server_info(context, ServerInfoCallback, (IntPtr) (&sinkName));
                while (pa_operation_get_state(op) == pa_operation_state_t.PA_OPERATION_RUNNING)
                {
                    pa_threaded_mainloop_wait(_mainLoop);
                }
                pa_operation_unref(op);
                return sinkName;
            }
        }

        /// <summary>
        /// 获取默认的输出设备信息
        /// 这个方法要在 pa_threaded_mainloop_lock 和 pa_threaded_mainloop_unlock 之间调用
        /// </summary>
        /// <param name="context"></param>
        /// <param name="sinkName"></param>
        /// <returns></returns>
        private (pa_cvolume volume, bool mute) GetSinkInfo(IntPtr context, string sinkName)
        {
            unsafe
            {
                (pa_cvolume, bool) info;
                var op = pa_context_get_sink_info_by_name(context, sinkName, SinkInfoCallback, (IntPtr) (&info));
                while (pa_operation_get_state(op) == pa_operation_state_t.PA_OPERATION_RUNNING)
                {
                    pa_threaded_mainloop_wait(_mainLoop);
                }
                pa_operation_unref(op);
                return info;
            }
        }

        private int GetVolumeValue(in pa_cvolume volume)
        {
            var val = pa_cvolume_avg(volume);
            return (int) Math.Round((val * 100d / 65536));
        }
    }

    public partial class PulseAudioVolumeManager
    {
        public static class PInvoke
        {
            public enum pa_context_state_t
            {
                PA_CONTEXT_UNCONNECTED,
                PA_CONTEXT_CONNECTING,
                PA_CONTEXT_AUTHORIZING,
                PA_CONTEXT_SETTING_NAME,
                PA_CONTEXT_READY,
                PA_CONTEXT_FAILED,
                PA_CONTEXT_TERMINATED,
            }

            [Flags]
            public enum pa_subscription_event_type_t : uint
            {
                PA_SUBSCRIPTION_EVENT_SINK = 0x0000U,
                PA_SUBSCRIPTION_EVENT_SOURCE = 0x0001U,
                PA_SUBSCRIPTION_EVENT_SINK_INPUT = 0x0002U,
                PA_SUBSCRIPTION_EVENT_SOURCE_OUTPUT = 0x0003U,
                PA_SUBSCRIPTION_EVENT_MODULE = 0x0004U,
                PA_SUBSCRIPTION_EVENT_CLIENT = 0x0005U,
                PA_SUBSCRIPTION_EVENT_SAMPLE_CACHE = 0x0006U,
                PA_SUBSCRIPTION_EVENT_SERVER = 0x0007U,
                PA_SUBSCRIPTION_EVENT_AUTOLOAD = 0x0008U,
                PA_SUBSCRIPTION_EVENT_CARD = 0x0009U,
                PA_SUBSCRIPTION_EVENT_FACILITY_MASK = 0x000FU,
                PA_SUBSCRIPTION_EVENT_NEW = 0x0000U,
                PA_SUBSCRIPTION_EVENT_CHANGE = 0x0010U,
                PA_SUBSCRIPTION_EVENT_REMOVE = 0x0020U,
                PA_SUBSCRIPTION_EVENT_TYPE_MASK = 0x0030U,
            }

            public enum pa_subscription_mask_t : uint
            {
                PA_SUBSCRIPTION_MASK_NULL = 0x0000U,
                PA_SUBSCRIPTION_MASK_SINK = 0x0001U,
                PA_SUBSCRIPTION_MASK_SOURCE = 0x0002U,
                PA_SUBSCRIPTION_MASK_SINK_INPUT = 0x0004U,
                PA_SUBSCRIPTION_MASK_SOURCE_OUTPUT = 0x0008U,
                PA_SUBSCRIPTION_MASK_MODULE = 0x0010U,
                PA_SUBSCRIPTION_MASK_CLIENT = 0x0020U,
                PA_SUBSCRIPTION_MASK_SAMPLE_CACHE = 0x0040U,
                PA_SUBSCRIPTION_MASK_SERVER = 0x0080U,
                PA_SUBSCRIPTION_MASK_AUTOLOAD = 0x0100U,
                PA_SUBSCRIPTION_MASK_CARD = 0x0200U,
                PA_SUBSCRIPTION_MASK_ALL = 0x02ffU,
            }

            public enum pa_sample_format_t
            {
                PA_SAMPLE_U8,
                PA_SAMPLE_ALAW,
                PA_SAMPLE_ULAW,
                PA_SAMPLE_S16LE,
                PA_SAMPLE_S16BE,
                PA_SAMPLE_FLOAT32LE,
                PA_SAMPLE_FLOAT32BE,
                PA_SAMPLE_S32LE,
                PA_SAMPLE_S32BE,
                PA_SAMPLE_S24LE,
                PA_SAMPLE_S24BE,
                PA_SAMPLE_S24_32LE,
                PA_SAMPLE_S24_32BE,
                PA_SAMPLE_MAX,
                PA_SAMPLE_INVALID = -1
            }

            public enum pa_operation_state_t
            {
                PA_OPERATION_RUNNING,
                PA_OPERATION_DONE,
                PA_OPERATION_CANCELLED,
            }


            [StructLayout(LayoutKind.Sequential)]
            public struct pa_server_info
            {
                public IntPtr user_name;
                public IntPtr host_name;
                public IntPtr server_version;
                public IntPtr server_name;
                pa_sample_spec sample_spec;
                public IntPtr default_sink_name;
                public IntPtr default_source_name;
                public uint cookie;
                pa_channel_map channel_map;
            }

            [StructLayout(LayoutKind.Sequential)]
            public struct pa_sample_spec
            {
                public pa_sample_format_t format;
                public uint rate;
                public byte channels;
            }

            [StructLayout(LayoutKind.Sequential, Size = 132)]
            public struct pa_channel_map
            {
                public byte channels;
                //public pa_channel_position_t map[PA_CHANNELS_MAX];
            }

            [StructLayout(LayoutKind.Sequential, Size = 132)]
            public struct pa_cvolume
            {
                public byte channels;
                //public pa_volume_t values[PA_CHANNELS_MAX];
            }

            [StructLayout(LayoutKind.Sequential)]
            public struct pa_sink_info
            {
                public IntPtr name;
                public uint index;
                public IntPtr description;
                public pa_sample_spec sample_spec;
                public pa_channel_map channel_map;
                public uint owner_module;
                public pa_cvolume volume;
                public int mute;
                public uint monitor_source;
                public IntPtr monitor_source_name;
                public ulong latency;
                public IntPtr driver;
                public uint flags;
                public IntPtr proplist;
                public ulong configured_latency;
                public uint base_volume;
                public uint state;
                public uint n_volume_steps;
                public uint card;
                public uint n_ports;
                public IntPtr ports;
                public IntPtr active_port;
                public byte n_formats;
                public IntPtr formats;
            }

            [MethodImpl(MethodImplOptions.AggressiveInlining)]
            public static bool PA_CONTEXT_IS_GOOD(pa_context_state_t x)
            {
                return x == pa_context_state_t.PA_CONTEXT_CONNECTING || x == pa_context_state_t.PA_CONTEXT_AUTHORIZING ||
                    x == pa_context_state_t.PA_CONTEXT_SETTING_NAME || x == pa_context_state_t.PA_CONTEXT_READY;
            }

            public delegate void pa_context_notify_cb_t(IntPtr c, IntPtr userdata);

            public delegate void pa_context_subscribe_cb_t(IntPtr c, pa_subscription_event_type_t t, uint idx, IntPtr userdata);

            public delegate void pa_server_info_cb_t(IntPtr c, in pa_server_info i, IntPtr userdata);

            public delegate void pa_sink_info_cb_t(IntPtr c, in pa_sink_info i, int eol, IntPtr userdata);

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_threaded_mainloop_new();

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_threaded_mainloop_get_api(IntPtr m);

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_context_new(IntPtr mainloop, [MarshalAs(UnmanagedType.LPUTF8Str)] string name);

            [DllImport("libpulse.so.0")]
            public static extern void pa_context_set_state_callback(IntPtr c, pa_context_notify_cb_t cb, IntPtr userdata);

            [DllImport("libpulse.so.0")]
            public static extern int pa_context_connect(IntPtr c, IntPtr server, uint flags, IntPtr api);

            [DllImport("libpulse.so.0")]
            public static extern int pa_threaded_mainloop_start(IntPtr m);

            [DllImport("libpulse.so.0")]
            public static extern pa_context_state_t pa_context_get_state(IntPtr c);

            [DllImport("libpulse.so.0")]
            public static extern void pa_threaded_mainloop_wait(IntPtr m);

            [DllImport("libpulse.so.0")]
            public static extern void pa_context_set_subscribe_callback(IntPtr c, pa_context_subscribe_cb_t cb, IntPtr userdata);

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_context_subscribe(IntPtr c, pa_subscription_mask_t m, IntPtr cb, IntPtr userdata);

            [DllImport("libpulse.so.0")]
            public static extern void pa_operation_unref(IntPtr o);

            [DllImport("libpulse.so.0")]
            public static extern void pa_threaded_mainloop_signal(IntPtr m, int wait_for_accept);

            [DllImport("libpulse.so.0")]
            public static extern void pa_threaded_mainloop_lock(IntPtr m);

            [DllImport("libpulse.so.0")]
            public static extern void pa_threaded_mainloop_unlock(IntPtr m);

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_context_get_server_info(IntPtr c, pa_server_info_cb_t cb, IntPtr userdata);

            [DllImport("libpulse.so.0")]
            public static extern pa_operation_state_t pa_operation_get_state(IntPtr o);

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_context_get_sink_info_by_name(IntPtr c, [MarshalAs(UnmanagedType.LPUTF8Str)] string name, pa_sink_info_cb_t cb, IntPtr userdata);

            [DllImport("libpulse.so.0")]
            public static extern uint pa_cvolume_avg(in pa_cvolume a);

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_cvolume_set(ref pa_cvolume a, uint channels, uint v);

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_context_set_sink_mute_by_name(IntPtr c, [MarshalAs(UnmanagedType.LPUTF8Str)] string name, int mute, IntPtr cb, IntPtr userdata);

            [DllImport("libpulse.so.0")]
            public static extern IntPtr pa_context_set_sink_volume_by_name(IntPtr c, [MarshalAs(UnmanagedType.LPUTF8Str)] string name, in pa_cvolume volume, IntPtr cb, IntPtr userdata);
        }
    }
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7dc9f2c0ab4fd8557202b28e752aaff5a730ff9d/LiwhallyawhuleLaqarhifehawhedem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7dc9f2c0ab4fd8557202b28e752aaff5a730ff9d/LiwhallyawhuleLaqarhifehawhedem) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7dc9f2c0ab4fd8557202b28e752aaff5a730ff9d
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7dc9f2c0ab4fd8557202b28e752aaff5a730ff9d
```

获取代码之后，进入 LiwhallyawhuleLaqarhifehawhedem 文件夹，即可获取到源代码
