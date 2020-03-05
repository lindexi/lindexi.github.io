# win10 UWP 发邮件

UWP 下如何发邮件？可以使用`mailto:xx?subject=*`方式发送？

本文：如何在 UWP 使用默认邮件发邮件。

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->


<div id="toc"></div>

打开设置，应用，默认应用，选择应用 OutLook。这样就和我的一样，如果出错了，那么是邮件不支持。

首先需要找联系人，联系人可以在用户联系找。


```csharp
            //找到一个联系人
            //如果是需要用户选发送到哪个联系人，使用下面方法
            var contactPicker = new ContactPicker();
            contactPicker.SelectionMode = ContactSelectionMode.Fields;//选择联系人一个项
            contactPicker.DesiredFieldsWithContactFieldType.Add(ContactFieldType.Email);//选择email
            Contact contact = await contactPicker.PickContactAsync();
```

让用户选择有email的联系，不选择一个联系全部。这句话说的是，在用户选择联系人之后，让他选择联系人的一个项。

![](http://image.acmx.xyz/f182d3db-d997-4f86-801b-fde591612fa7201721995012.jpg)

选择联系人，选择一个邮箱

![](http://image.acmx.xyz/f182d3db-d997-4f86-801b-fde591612fa7201721995041.jpg)

如果指定一个联系人让用户发送，如开发者，可以直接写自己的邮箱

```csharp
            contact = new Contact()
            {
                Emails =
                {
                    new ContactEmail()
                    {
                        Address = "lindexi_gd@163.com",
                        Description = "UWP 开发者",
                    }
                }
            };
```


然后需要填写主题，内容。可以添加附件，注意附件添加是 StorageFile 。

可以看到，需要写的代码很多，我需要
写一个类来发送，首先使用`Windows.ApplicationModel.Email`


```csharp
    using Windows.ApplicationModel.Contacts;
    using Windows.ApplicationModel.Email;
```

需要主题和内容


```csharp
            var emailMessage = new EmailMessage();

            emailMessage.Subject = subject;
            emailMessage.Body = messageBody;
```

如果需要使用附件，
如何读取 StorageFile ？


```csharp
            if (attachmentFile != null)
            {
                var stream = RandomAccessStreamReference.CreateFromFile(attachmentFile);

                var attachment = new EmailAttachment(
                    attachmentFile.Name,
                    stream);

                emailMessage.Attachments.Add(attachment);
            }
```

然后添加收件人


```csharp
            var email = recipient.Emails.FirstOrDefault<ContactEmail>();
            if (email != null)
            {
                var emailRecipient = new EmailRecipient(email.Address);
                emailMessage.To.Add(emailRecipient);
            }
```

假如发给多个人，使用 `emailMessage.To.Add` list

发邮件很简单，`await EmailManager.ShowComposeNewEmailAsync(emailMessage);`就可以让用户发邮件

![](https://ooo.0o0.ooo/2017/02/19/58a8fe3a8e17d.gif)



如果默认不是wr的，那么发送邮件可以出错，不是所有的软件都支持，于是可以使用另一个方式：

`mailto:{email}?subject={subject}&body={messageBody}`

如果遇到messageBody有换行可以看到这个方法没有换行。

UWP 发送邮件内容如何换行，messageBody 用的是 html ，所以使用 `Uri.EscapeDataString`

我写了一个函数，多谢 [李继龙](mailto:kljzndx@outlook.com) 大神，可以传入 email 主题 内容就可以发送


```csharp
        private async Task UniversallyEmail(string email, string subject, string messageBody)
        {
            messageBody = Uri.EscapeDataString(messageBody); 用于换行
            string url = $"mailto:{email}?subject={subject}&body={messageBody}";
            await Launcher.LaunchUriAsync(new Uri(url));
        }
```


代码：http://download.csdn.net/detail/lindexi_gd/9757862

参见：https://docs.microsoft.com/en-us/windows/uwp/contacts-and-calendar/sending-email

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  