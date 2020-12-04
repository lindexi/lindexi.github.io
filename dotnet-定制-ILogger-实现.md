
# dotnet 定制 ILogger 实现

默认在 dotnet 里面框架提供了 Microsoft.Extensions.Logging 可以和依赖注入做日志框架，而有些业务，如需要自己定制日志行为，此时就需要定制日志

<!--more-->


<!-- CreateTime:6/28/2020 7:49:17 PM -->



当初写一个类继承 ILogger 是做不到定制，需要再写一个类继承 ILoggerProvider 才好做定制

如以下的方法

```csharp
    public class CCloudConsoleLogProvider : ILoggerProvider
    {
        /// <inheritdoc />
        public void Dispose()
        {

        }

        /// <inheritdoc />
        public ILogger CreateLogger(string categoryName)
        {
            return new CCloudConsoleLogger();
        }

        class CCloudConsoleLogger : ILogger
        {
        	// 忽略代码
        }
    }
```

通过 DI 的注入，在注入之前先干掉其他的 ILoggerProvider 实例

```csharp
services.AddLogging(builder =>
{
    builder.ClearProviders();
    builder.AddProvider(new CCloudConsoleLogProvider());
});
```

现在所有拿到的 ILogger 都是从 CCloudConsoleLogProvider 创建的

下面是我定制的符合 honeycomb log 输出格式的日志，输出内容如下

```csharp
[2099-10-19 19:07:45.456][threadName][INFO][类:行数][_traceId:realTraceId][_userId:realUserId][tag:custom] 业务的日志/异常堆栈
```

全部代码

```csharp
    public class CCloudConsoleLogProvider : ILoggerProvider
    {
        /// <inheritdoc />
        public void Dispose()
        {

        }

        /// <inheritdoc />
        public ILogger CreateLogger(string categoryName)
        {
            return new CCloudConsoleLogger(categoryName);
        }

        class CCloudConsoleLogger : ILogger
        {
            public CCloudConsoleLogger(string categoryName)
            {
                _categoryName = categoryName;
            }

            class Empty : IDisposable
            {
                /// <inheritdoc />
                public void Dispose()
                {
                }
            }

            /// <inheritdoc />
            public IDisposable BeginScope<TState>(TState state)
            {
                return new Empty();
            }

            /// <inheritdoc />
            public bool IsEnabled(LogLevel logLevel)
            {
                return true;
            }

            private readonly string _categoryName;

            /// <inheritdoc />
            public void Log<TState>(LogLevel logLevel, EventId eventId, TState state, Exception exception,
                Func<TState, Exception, string> formatter)
            {
                // [2099-10-19 19:07:45.456][threadName][INFO][类:行数][_traceId:realTraceId][_userId:realUserId][tag:custom] 业务的日志/异常堆栈
                string message;
                if (typeof(TState) == typeof(CCloudLogInfo))
                {
                    var logInfo = state as CCloudLogInfo;
                    Debug.Assert(logInfo != null, nameof(logInfo) + " != null");
                    logInfo.CategoryName = _categoryName;
                    message = formatter(state, exception);
                }
                else
                {
                    // [2099-10-19 19:07:45.456][threadName][INFO][类:行数][_traceId:realTraceId][_userId:realUserId][tag:custom] 业务的日志/异常堆栈
                    message =
                        $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss.fff}][{Thread.CurrentThread.Name}:{Thread.CurrentThread.ManagedThreadId}][{CCloudLogExtension.LogLevelToString(logLevel)}][{_categoryName}][-][-][-][EventId={eventId.Id}:{eventId.Name}] {formatter(state, exception)}";
                }

                Console.WriteLine(message);
            }
        }
    }

    internal class CCloudLogInfo
    {
        public string CategoryName { set; get; }

        public string ThreadName { get; set; }
        public int ThreadId { get; set; }

        public string ClassFile { set; get; }

        public int LineNumber { get; set; }

        public string Message { set; get; }

        public string TraceId { set; get; }

        public string UserId { set; get; }

        public string MemberName { set; get; }

        public string[] Tags { set; get; }

        public LogLevel LogLevel { set; get; }
    }

    public static class CCloudLogExtension
    {
        public static void Error(this ILogger logger,
            string message,
            Exception exception = null,
            string traceId = null,
            string userId = null,
            [System.Runtime.CompilerServices.CallerMemberName]
            string memberName = "",
            [System.Runtime.CompilerServices.CallerFilePath]
            string sourceFilePath = "",
            [System.Runtime.CompilerServices.CallerLineNumber]
            int sourceLineNumber = 0,
            params string[] tags)
        {
            const LogLevel logLevel = LogLevel.Error;
            var logInfo = new CCloudLogInfo()
            {
                ClassFile = Path.GetFileName(sourceFilePath),
                ThreadId = Thread.CurrentThread.ManagedThreadId,
                ThreadName = Thread.CurrentThread.Name,
                LineNumber = sourceLineNumber,
                Message = message,
                TraceId = traceId,
                UserId = userId,
                MemberName = memberName,
                Tags = tags,
                LogLevel = logLevel
            };

            logger.Log(logLevel, eventId: EmptyEventId, logInfo, exception, Formatter);
        }

        public static void Warning(this ILogger logger,
            string message,
            Exception exception = null,
            string traceId = null,
            string userId = null,
            [System.Runtime.CompilerServices.CallerMemberName]
            string memberName = "",
            [System.Runtime.CompilerServices.CallerFilePath]
            string sourceFilePath = "",
            [System.Runtime.CompilerServices.CallerLineNumber]
            int sourceLineNumber = 0,
            params string[] tags)
        {
            const LogLevel logLevel = LogLevel.Warning;
            var logInfo = new CCloudLogInfo()
            {
                ClassFile = Path.GetFileNameWithoutExtension(sourceFilePath),
                ThreadId = Thread.CurrentThread.ManagedThreadId,
                ThreadName = Thread.CurrentThread.Name,
                LineNumber = sourceLineNumber,
                Message = message,
                TraceId = traceId,
                UserId = userId,
                MemberName = memberName,
                Tags = tags,
                LogLevel = logLevel
            };

            logger.Log(logLevel, eventId: EmptyEventId, logInfo, exception, Formatter);
        }

        // 为什么只有 Info 可以添加 Exception 不添加信息，因为如果是 Warning 和 Error 推荐写是哪个模块
        public static void Info(this ILogger logger,
            Exception exception = null,
            string traceId = null,
            string userId = null,
            [System.Runtime.CompilerServices.CallerMemberName]
            string memberName = "",
            [System.Runtime.CompilerServices.CallerFilePath]
            string sourceFilePath = "",
            [System.Runtime.CompilerServices.CallerLineNumber]
            int sourceLineNumber = 0,
            params string[] tags)
        {
            // ReSharper disable ExplicitCallerInfoArgument
            Info(logger, null, exception, traceId, userId, memberName, sourceFilePath, sourceLineNumber, tags);
            // ReSharper restore ExplicitCallerInfoArgument
        }

        public static void Info(this ILogger logger, 
            string message,
            Exception exception = null,
            string traceId = null,
            string userId = null,
            [System.Runtime.CompilerServices.CallerMemberName]
            string memberName = "",
            [System.Runtime.CompilerServices.CallerFilePath]
            string sourceFilePath = "",
            [System.Runtime.CompilerServices.CallerLineNumber]
            int sourceLineNumber = 0,
            params string[] tags)
        {
            // 刚好在 Linux 下构建的在 Linux 下运行，而在 Windows 构建的库在 Windows 下执行。此时使用 GetFileNameWithoutExtension 能保持输入路径和解析相同
            // 假定在 Windows 下构建而在 Linux 下构建，只是让路径变长而已，我相信咱的日志系统炸不了…… 或者说，炸了再说
            // 炸了的解决方法是在 dotnet runtime\src\libraries\System.Private.CoreLib\src\System\IO\Path.cs 的 GetFileName 方法里面将 `PathInternal.IsDirectorySeparator(path[i])` 替换为实际需要的 \ 或 / 符号

            const LogLevel logLevel = LogLevel.Information;
            var logInfo = new CCloudLogInfo()
            {
                ClassFile = Path.GetFileName(sourceFilePath),
                ThreadId = Thread.CurrentThread.ManagedThreadId,
                ThreadName = Thread.CurrentThread.Name,
                LineNumber = sourceLineNumber,
                Message = message,
                TraceId = traceId,
                UserId = userId,
                MemberName = memberName,
                Tags = tags,
                LogLevel = logLevel
            };

            logger.Log(logLevel, eventId: EmptyEventId, logInfo, exception, Formatter);
        }

        public static void Debug(this ILogger logger,
            string message,
            Exception exception = null,
            string traceId = null,
            string userId = null,
            [System.Runtime.CompilerServices.CallerMemberName]
            string memberName = "",
            [System.Runtime.CompilerServices.CallerFilePath]
            string sourceFilePath = "",
            [System.Runtime.CompilerServices.CallerLineNumber]
            int sourceLineNumber = 0,
            params string[] tags)
        {
            var logInfo = new CCloudLogInfo()
            {
                ClassFile = Path.GetFileName(sourceFilePath),
                ThreadId = Thread.CurrentThread.ManagedThreadId,
                ThreadName = Thread.CurrentThread.Name,
                LineNumber = sourceLineNumber,
                Message = message,
                TraceId = traceId,
                UserId = userId,
                MemberName = memberName,
                Tags = tags,
                LogLevel = LogLevel.Debug
            };

            logger.Log(logInfo.LogLevel, eventId: EmptyEventId, logInfo, exception, Formatter);
        }

        private static string Formatter(CCloudLogInfo logInfo, Exception exception)
        {
            // honeycomb-log
            // [2099-10-19 19:07:45.456][threadName][INFO][类:行数][_traceId:realTraceId][_userId:realUserId][tag:custom] 业务的日志/异常堆栈

            const string empty = "-";

            var traceMessage = string.IsNullOrEmpty(logInfo.TraceId) ? empty : $"_traceId:{logInfo.TraceId}";

            var userMessage = string.IsNullOrEmpty(logInfo.UserId) ? empty : $"_userId:{logInfo.UserId}";

            var logLevelMessage = LogLevelToString(logInfo.LogLevel);

            var logInfoMessage = string.IsNullOrEmpty(logInfo.Message) ? exception?.Message : logInfo.Message;

            return $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss.sss}][{logInfo.ThreadName}:{logInfo.ThreadId}][{logLevelMessage}][{logInfo.ClassFile}:{logInfo.CategoryName}.{logInfo.MemberName}:{logInfo.LineNumber}][{traceMessage}][{userMessage}][tags:{string.Join(";", logInfo.Tags)}] {logInfoMessage} {exception?.ToString()}";
        }

        public static string LogLevelToString(LogLevel logLevel)
            => logLevel switch
            {
                LogLevel.Trace => "TRACE",
                LogLevel.Debug => "DEBUG",
                LogLevel.Information => "INFO",
                LogLevel.Warning => "WARNING",
                LogLevel.Error => "ERROR",
                LogLevel.Critical => "CRITICAL",
                LogLevel.None => "NONE",
                _ => "NONE"
            };

        private static readonly EventId EmptyEventId = new EventId();
    }
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。