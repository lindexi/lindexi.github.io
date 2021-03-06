# WPF 源代码 资源字典 ResourceDictionary 设置 Source 属性的逻辑

本文来和大家聊聊在 WPF 里面在给 ResourceDictionary 设置 Source 属性时，在 WPF 框架里面做了哪些逻辑

<!--more-->
<!-- CreateTime:2020/11/25 14:22:23 -->



默认添加 Source 时都是指定 WPF 自身的 XAML 资源字典，用途就是指定 XAML 字典作为此控件的资源字典

而默认的 XAML 资源字典使用 Page 形式进行构建，构建之后作为二进制的 Baml 文件被打入程序集中作为程序集资源，在 dotnet 里面有专门的程序集 System.IO.Packaging 来解析程序集资源

而给 Source 属性赋值的时候，给的是一个 Uri 类型的变量，那么 资源字典 ResourceDictionary 是如何通过 Uri 拿到对应的内容的？其实在 WPF 的 ResourceDictionary 的 Source 属性赋值里面有很长的一段逻辑，如下面代码，请大家快速跳过，这些代码只是用来告诉大家，在 WPF 里面使用了很多代码来处理这部分逻辑

```csharp
        public Uri Source
        {
            get
            {
                return _source;
            }
            set
            {
                if (value == null || String.IsNullOrEmpty(value.OriginalString))
                {
                    throw new ArgumentException(SR.Get(SRID.ResourceDictionaryLoadFromFailure, value == null ? "''" : value.ToString()));
                }

                ResourceDictionaryDiagnostics.RemoveResourceDictionaryForUri(_source, this);

                ResourceDictionarySourceUriWrapper uriWrapper = value as ResourceDictionarySourceUriWrapper;

                Uri sourceUri;

                // If the Uri we received is a ResourceDictionarySourceUriWrapper it means
                // that it is being passed down by the Baml parsing code, and it is trying to give us more
                // information to avoid possible ambiguities in assembly resolving. Use the VersionedUri
                // to resolve, and the set _source to the OriginalUri so we don't change the return of Source property.
                // The versioned Uri is not stored, if the version info is needed while debugging, once this method 
                // returns _reader should be set, from there BamlSchemaContext.LocalAssembly contains the version info.
                if (uriWrapper == null)
                {
                    _source = value;
                    sourceUri = _source;
                }
                else
                {
                    _source = uriWrapper.OriginalUri;
                    sourceUri = uriWrapper.VersionedUri;
                }
                
                Clear();
                
                
                Uri uri = BindUriHelper.GetResolvedUri(_baseUri, sourceUri);

                WebRequest request = WpfWebRequestHelper.CreateRequest(uri);
                WpfWebRequestHelper.ConfigCachePolicy(request, false);
                ContentType contentType = null;
                Stream s = null;

                try
                {
                     s = WpfWebRequestHelper.GetResponseStream(request, out contentType);
                }
                catch (System.IO.IOException)
                {
                     // 忽略细节处理的代码
                }

                // MimeObjectFactory.GetObjectAndCloseStream will try to find the object converter basing on the mime type.
                // It can be a sync/async converter. It's the converter's responsiblity to close the stream.
                // If it fails to find a convert, this call will return null.
                System.Windows.Markup.XamlReader asyncObjectConverter;
                ResourceDictionary loadedRD = MimeObjectFactory.GetObjectAndCloseStream(s, contentType, uri, false, false, false /*allowAsync*/, false /*isJournalNavigation*/, out asyncObjectConverter)
                                            as ResourceDictionary;

                if (loadedRD == null)
                {
                    throw new InvalidOperationException(SR.Get(SRID.ResourceDictionaryLoadFromFailure, _source.ToString()));
                }

                // ReferenceCopy all the key-value pairs in the _baseDictionary
                _baseDictionary = loadedRD._baseDictionary;

                // ReferenceCopy all the entries in the MergedDictionaries collection
                _mergedDictionaries = loadedRD._mergedDictionaries;

                // ReferenceCopy all of the deferred content state
                CopyDeferredContentFrom(loadedRD);

                // Take over the deferred resource references
                MoveDeferredResourceReferencesFrom(loadedRD);

                // Copy over the HasImplicitStyles flag
                HasImplicitStyles = loadedRD.HasImplicitStyles;

                // Copy over the HasImplicitDataTemplates flag
                HasImplicitDataTemplates = loadedRD.HasImplicitDataTemplates;

                // Copy over the InvalidatesImplicitDataTemplateResources flag
                InvalidatesImplicitDataTemplateResources = loadedRD.InvalidatesImplicitDataTemplateResources;

                // Set inheritance context on the copied values
                if (InheritanceContext != null)
                {
                    AddInheritanceContextToValues();
                }

                // Propagate parent owners to each of the acquired merged dictionaries
                if (_mergedDictionaries != null)
                {
                    for (int i = 0; i < _mergedDictionaries.Count; i++)
                    {
                        PropagateParentOwners(_mergedDictionaries[i]);
                    }
                }

                ResourceDictionaryDiagnostics.AddResourceDictionaryForUri(uri, this);

                if (!IsInitializePending)
                {
                    // Fire Invalidations for the changes made by asigning a new Source
                    NotifyOwners(new ResourcesChangeInfo(null, this));
                }
            }
        }
```

大概整理一下的在 Source 的 set 方法里面的逻辑大概如下

```csharp
// 前置属性判断

// 清空当前资源字典的所有元素
Clear();

// 解析 Uri 获取资源
ResourceDictionary loadedRD = xx; // 这里的 loadedRD 就是 loadedR(esource)D(ictionary) 的意思

// 取出资源字典的值
_baseDictionary = loadedRD._baseDictionary;

// 取出资源字典加载的其他资源字典
_mergedDictionaries = loadedRD._mergedDictionaries;

// 其他杂项状态处理
```

可以看到上面代码的逻辑步骤其实很少，核心的逻辑就是 解析 Uri 获取资源 这部分

在开始获取资源之前，需要先将 Uri 转换为绝对路径，也就是说在 XAML 中写的 Uri 将会被补全

```csharp
     _source = value;
     sourceUri = _source;
     Uri uri = BindUriHelper.GetResolvedUri(_baseUri, sourceUri);
```

这里拿到 uri 之后，通过调用 WpfWebRequestHelper 的方法拿到资源的 Stream 对象

```csharp
WebRequest request = WpfWebRequestHelper.CreateRequest(uri);
WpfWebRequestHelper.ConfigCachePolicy(request, false);
ContentType contentType = null;
Stream s = null;

try
{
    s = WpfWebRequestHelper.GetResponseStream(request, out contentType);
}
catch
{
	// 忽略细节代码
}
```

看到了 WebRequest 请不要激动，这不代表一定会从网络上读取哦

因为这个 WebRequest 是使用 WpfWebRequestHelper 的 CreateRequest 拿到的 WebRequest 不一定是一个走网络的 WebRequest 哦，在 WpfWebRequestHelper 的 CreateRequest 方法里面，会根据 Uri 进行判断，假定是获取到一个在应用本地资源的路径，那么将使用 PackWebRequestFactory.CreateWebRequest 返回一个基于 System.IO.Packaging 的 PackWebRequest 对象。否则就是真的走网络了，因此给资源字典设置一个网络上的 Url 也是可以的

在 PackWebRequest 里面，其实就是一个继承了 WebRequest 的类，这个类的命名空间是 System.IO.Packaging 但是放在 PresentationCore 里面，是逻辑上属于 System.IO.Packaging 程序集，但实际上在 PresentationCore 程序集

在 PackWebRequest 通过重写 WebRequest 的方法，实现了实际上没有走网络，而是返回了 PackWebResponse 对象，在 PackWebResponse 里面就是读取程序集的资源作为 Stream 返回

因此调用 WpfWebRequestHelper 的 CreateRequest 方法创建的 WebRequest 在传入的是 uri 是一个本地的资源字典的时候，就是读取本地程序集资源返回 Stream 对象

读取到 Stream 之后需要进行解析，如下面代码

```csharp
ResourceDictionary loadedRD = MimeObjectFactory.GetObjectAndCloseStream(s, contentType, uri, false, false, false /*allowAsync*/, false /*isJournalNavigation*/, out asyncObjectConverter) as ResourceDictionary;
```

此时拿到了 loadedResourceDictionary 也就是 loadedRD 变量，下一步就是取出里面的值

```csharp
                // ReferenceCopy all the key-value pairs in the _baseDictionary
                _baseDictionary = loadedRD._baseDictionary;

                // ReferenceCopy all the entries in the MergedDictionaries collection
                _mergedDictionaries = loadedRD._mergedDictionaries;
```

此时就完成了资源字典的从 Uri 加载了

在资源字典里面，包含了两层内容，第一层的内容就是在这个资源字典里面定义的资源，这些资源放在了 `private Hashtable _baseDictionary` 里面。第二层内容就是 `private ObservableCollection<ResourceDictionary> _mergedDictionaries` 被这个资源字典合并的其他资源字典里面

因此在 WPF 中寻找资源是先从自己的 `_baseDictionary` 尝试获取资源，如获取不到在从 `_mergedDictionaries` 里面获取，如下面代码

```csharp
        private object GetValueWithoutLock(object key, out bool canCache)
        {
            object value = _baseDictionary[key];
            if (value != null)
            {
                OnGettingValuePrivate(key, ref value, out canCache);
            }
            else
            {
                canCache = true;

                //Search for the value in the Merged Dictionaries
                if (_mergedDictionaries != null)
                {
                    for (int i = MergedDictionaries.Count - 1; (i > -1); i--)
                    {
                        // Note that MergedDictionaries collection can also contain null values
                        ResourceDictionary mergedDictionary = MergedDictionaries[i];
                        if (mergedDictionary != null)
                        {
                            value = mergedDictionary.GetValue(key, out canCache);
                            if (value != null)
                            {
                                break;
                            }
                        }
                    }
                }
            }

            return value;
        }
```

从上面代码可以看到，获取的时候是优先从 `_baseDictionary` 获取的。获取不到在从 MergedDictionaries 里面获取，最后添加的资源字典最先寻找。也就是说存在 Key 重复的资源的时候，会先从资源字典本身寻找，如果找不到就从合并的其他字典的最后一个资源字典开始寻找

这就是 WPF 资源字典设置的逻辑

当前整个 WPF 源代码都是开源的，请看 [https://github.com/dotnet/wpf/](https://github.com/dotnet/wpf/)

更多资源字典相关请看

<!-- ?WT.mc_id=WD-MVP-5003260

&WT.mc_id=WD-MVP-5003260 -->

- [Define XAML resources - WPF .NET](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/fundamentals/xaml-resources-define?view=netdesktop-5.0&WT.mc_id=WD-MVP-5003260 )
- [How to: Use an Application-Scope Resource Dictionary - WPF .NET Framework](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/app-development/how-to-use-an-application-scope-resource-dictionary?view=netframeworkdesktop-4.8&WT.mc_id=WD-MVP-5003260 )
- [ResourceDictionary.Source 属性_jiangxinyu的专栏-CSDN博客](https://blog.csdn.net/jiangxinyu/article/details/8756985?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1.control )
- [WPF之ResourceDictionary资源字典_LADT的博客-CSDN博客](https://blog.csdn.net/wlarlen/article/details/79047320 )
- [WPF 资源字典ResourceDictionary使用_Stay Hungry-CSDN博客](https://blog.csdn.net/wcc27857285/article/details/90943244 )
- [WPF 在后台代码定义 ResourceDictionary 资源字典](https://blog.lindexi.com/post/WPF-%E5%9C%A8%E5%90%8E%E5%8F%B0%E4%BB%A3%E7%A0%81%E5%AE%9A%E4%B9%89-ResourceDictionary-%E8%B5%84%E6%BA%90%E5%AD%97%E5%85%B8.html )
- [【WPF学习】第三十四章 资源基础 - Peter.Luo - 博客园](https://www.cnblogs.com/Peter-Luo/p/12288443.html )
- [【WPF学习】第三十五章 资源字典 - Peter.Luo - 博客园](https://www.cnblogs.com/Peter-Luo/p/12289408.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
