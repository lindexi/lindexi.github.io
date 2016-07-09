# win10 uwp 手动锁Bitlocker


##bitlocker

Windows BitLocker驱动器加密通过加密Windows操作系统卷上存储的所有数据可以更好地保护计算机中的数据。BitLocker使用TPM帮助保护Windows操作系统和用户数据，并帮助确保计算机即使在无人参与、丢失或被盗的情况下也不会被篡改。 BitLocker还可以在没有TPM的情况下使用。

win10有新的方法，我们可以使用新的。

我们可以在计算机管理，压缩磁盘，分区一个几百M的分区，然后使用Bitlocker锁。

但是在我们解锁后，直到重启才可以锁上。

那么在我们离开的时候如何锁上。

##手动锁

自动锁Bitlocker是不能的，但是我们可以有简单方法去锁

建立`*.bat`

```
manage-bde.exe 盘符: -lock
```
管理员运行


