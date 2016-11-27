----------

Title: win10 uwp 存放网络图片到本地

----------

有时候我们的网络很垃圾，我的的UWP要在第一次打开网络图片，就把图片存放到本地，下次可以从本地打开。

这就是先把图片下载，然后显示出来，存放到本地，接着下次要使用就可以从本地获取。

最好这个和我们用户是透明，我们不知道图片在哪，是本地还是网络，只要给一个Uri就有一个图片。

这里图片我用BitmapImage，Uri是输入网络的

## 下载图片

图片也是和其他一样，我们可以简单用系统给的网络web下载。

我们需要输入Uri，然后把图片下载。

图片要显示，需要SetSourceAsync，他需要的参数IRandomAccessStream，而这个需要Buffer写数据，不能用byte，我开始用的`System.Net.Http`没有获取Buffer方法，于是我查了垃圾wr，最后用`Windows.Web.Http`

先获取图片

```
                Windows.Web.Http.HttpClient http = new Windows.Web.Http.HttpClient();
                IBuffer buffer = await http.GetBufferAsync(uri);

                BitmapImage img = new BitmapImage();
                using (IRandomAccessStream stream = new InMemoryRandomAccessStream())
                {
                    await stream.WriteAsync(buffer);
                    stream.Seek(0);
                    await img.SetSourceAsync(stream);
                    await StorageImageFolder(stream, uri);
                    return img;
                }

```

StorageImageFolder就是保存图片

## 保存图片

我们需要知道一个Uri就可以拿到一个图片，但是Uri不能做文件名，于是我用md5

Uwp使用Md5，可以去看我写的文章

```
        private static string Md5(string str)
        {
            HashAlgorithmProvider hashAlgorithm =
                 HashAlgorithmProvider.OpenAlgorithm(HashAlgorithmNames.Md5);
            CryptographicHash cryptographic = hashAlgorithm.CreateHash();

            IBuffer buffer = CryptographicBuffer.ConvertStringToBinary(str, BinaryStringEncoding.Utf8);

            cryptographic.Append(buffer);

            return CryptographicBuffer.EncodeToHexString(cryptographic.GetValueAndReset());
        }

```

我们的图片存放在本地，最后放在`ApplicationData.Current.LocalCacheFolder`

在存放文件，RandomAccessStream需要转byte[]

```
        private static async Task<byte[]> ConvertIRandomAccessStreamByte(IRandomAccessStream stream)
        {
            DataReader read = new DataReader(stream.GetInputStreamAt(0));
            await read.LoadAsync((uint)stream.Size);
            byte[] temp = new byte[stream.Size];
            read.ReadBytes(temp);
            return temp;
        }

```

存放文件

```
            string image = Md5(uri.AbsolutePath);
            StorageFile file = await folder.CreateFileAsync(image);
            await FileIO.WriteBytesAsync(file, await ConvertIRandomAccessStreamByte(stream));

```

## 从本地打开

把Uri转为图片名，打开本地文件

```
            string name = Md5(uri.AbsolutePath);
            StorageFile file = await folder.GetFileAsync(name);
            using (var stream = await file.OpenAsync(FileAccessMode.Read))
            {
                BitmapImage img = new BitmapImage();
                await img.SetSourceAsync(stream);

                return img;
            }

```

## 所有代码

```
    public static class ImageStorage
    {
        /// <summary>
        /// 获取图片
        /// 如果本地存在，就获取本地
        /// 如果本地不存在，获取网络
        /// </summary>
        /// <param name="uri"></param>
        /// <returns></returns>
        public static async Task<BitmapImage> GetImage(Uri uri)
        {
            return await GetLoacalFolderImage(uri) ??
                   await GetHttpImage(uri);
        }

        /// <summary>
        /// 从本地获取图片
        /// </summary>
        /// <param name="uri"></param>
        private static async Task<BitmapImage> GetLoacalFolderImage(Uri uri)
        {
            StorageFolder folder = await GetImageFolder();

            string name = Md5(uri.AbsolutePath);

            try
            {
                StorageFile file = await folder.GetFileAsync(name);
                using (var stream = await file.OpenAsync(FileAccessMode.Read))
                {
                    BitmapImage img = new BitmapImage();
                    await img.SetSourceAsync(stream);

                    return img;
                }
            }
            catch (Exception)
            {
                return null;
            }
        }

        private static async Task<BitmapImage> GetHttpImage(Uri uri)
        {
            try
            {
                Windows.Web.Http.HttpClient http = new Windows.Web.Http.HttpClient();
                IBuffer buffer = await http.GetBufferAsync(uri);

                BitmapImage img = new BitmapImage();
                using (IRandomAccessStream stream = new InMemoryRandomAccessStream())
                {
                    await stream.WriteAsync(buffer);
                    stream.Seek(0);
                    await img.SetSourceAsync(stream);
                    await StorageImageFolder(stream, uri);
                    return img;
                }
            }
            catch (Exception)
            {
                return null;
            }
        }

        private static async Task StorageImageFolder(IRandomAccessStream stream, Uri uri)
        {
            StorageFolder folder = await GetImageFolder();

            string image = Md5(uri.AbsolutePath);

            try
            {
                StorageFile file = await folder.CreateFileAsync(image);
                await FileIO.WriteBytesAsync(file, await ConvertIRandomAccessStreamByte(stream));
            }
            catch (Exception)
            {

            }

        }

        private static async Task<byte[]> ConvertIRandomAccessStreamByte(IRandomAccessStream stream)
        {
            DataReader read = new DataReader(stream.GetInputStreamAt(0));
            await read.LoadAsync((uint)stream.Size);
            byte[] temp = new byte[stream.Size];
            read.ReadBytes(temp);
            return temp;
        }

        private static async Task<StorageFolder> GetImageFolder()
        {
            //文件夹
            string name = "image";

            StorageFolder folder = null;

            //从本地获取文件夹
            try
            {
                folder = await ApplicationData.Current.LocalCacheFolder.GetFolderAsync(name);
            }
            catch (FileNotFoundException)
            {
                //没找到
                folder = await ApplicationData.Current.LocalCacheFolder.
                    CreateFolderAsync(name);
            }

            return folder;
        }

        private static string Md5(string str)
        {
            HashAlgorithmProvider hashAlgorithm =
                 HashAlgorithmProvider.OpenAlgorithm(HashAlgorithmNames.Md5);
            CryptographicHash cryptographic = hashAlgorithm.CreateHash();

            IBuffer buffer = CryptographicBuffer.ConvertStringToBinary(str, BinaryStringEncoding.Utf8);

            cryptographic.Append(buffer);

            return CryptographicBuffer.EncodeToHexString(cryptographic.GetValueAndReset());
        }

    }

```


