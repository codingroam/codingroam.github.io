# GitHub提交设置代理


GitHub提交443错误可以设置代理（科学上网）解决

<!--more-->

## GitHub提交443错误设置代理

> https方式提交可以通过设置代理解决，但是更好的办法是设置SSH方式来提交代码，详情请查看笔记《Git设置SSH方式》

### 错误描述

OpenSSL SSL_connect: Connection was reset in connection to github.com:443
 看错误描述就标识ssl连接不到443端口。
 可以设置代理解决。先检查git的全局配置

### 查看全局配置

```csharp
git config --global -l
```

检查是否有https.proxy及http.proxy项

### 设置全局代理设置

示例10808端口是代理软件端口，按个人情况修改。

```css
git config --global http.proxy 127.0.0.1:10808
git config --global https.proxy 127.0.0.1:10808

git config --global http.proxy 127.0.0.1:10818
git config --global https.proxy 127.0.0.1:10818
```

### 已有设置情况修改代理项

```php
git config --global --unset http.proxy
git config --global --unset https.proxy
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/github%E8%AE%BE%E7%BD%AE%E4%BB%A3%E7%90%86/  

