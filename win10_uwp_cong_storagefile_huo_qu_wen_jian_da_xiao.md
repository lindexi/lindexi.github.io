# win10 uwp 从StorageFile获取文件大小

本文主要：获取文件大小

        private async Task<ulong> FileSize(Windows.Storage.StorageFile file)
        {
            var size = await file.GetBasicPropertiesAsync();
            return size.Size;
        }
        
        