# Docker Redis Connection refused解决方法

Docker Redis Connection refused解决方法
<!--more-->


使用docker-compose部署了dnmp+redis环境，但是php在使用127.0.0.1连接redis时却报Connection refused 的错误，后来在stack overflow上找到了解决方法。

原贴地址： [https://stackoverflow.com/questions/42360356/docker-redis-connection-refused/42361204](https://stackoverflow.com/questions/42360356/docker-redis-connection-refused/42361204)


![img](https://img-blog.csdn.net/20180904215132686?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA4Mzc2MTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


![img](https://img-blog.csdn.net/20180904215143429?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA4Mzc2MTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

简单来说，容器之间相互隔绝，在进行了端口映射之后，宿主机可以通过127.0.0.1:6379访问redis，但php容器不行。

不过在php中可以直接使用hostname：redis 来连接redis容器



---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/docker-redis-connection-refused%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95/  

