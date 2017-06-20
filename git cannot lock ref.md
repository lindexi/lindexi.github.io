# git cannot lock ref

cannot lock ref 'refs/remotes/origin/xx':'refs/remotes/origin/xx/xx' exists cannot create 'ref/remotes/origin/xx'

<!--more-->
<!-- csdn -->

请使用下面代码

```csharp
git update-ref -d refs/remotes 
git fetch
```
https://stackoverflow.com/questions/43533473/error-cannot-lock-ref-refs-tags-exists-cannot-create-refs-tags