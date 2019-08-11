# Nginx特点

## 反向代理

- 正向代理

  为客户端做代理（如VPN），将客户端的请求转发给代理，（客户端 - 代理） - 服务器

  客户端知道目标服务器，而服务器不知道客户端是通过代理访问的

- 反向代理

  为服务器做代理，客户端 - （外网 - 内网）

  客户端并不知道中间的过程

  ------

- Nginx是反向代理模式的

  将静态资源配置在Nginx上，用root说明是静态资源，由Nginx直接返回

  动态资源配置到Tomcat，用proxy_pass说明是动态请求，需要进行转发

  客户端 - Nginx - Tomcat，Tomcat的请求IP地址是Nginx的地址，而非客户端请求地址
  
  ![image-20190811004905413](/Users/supeeeer/Documents/myGitHub/Notes/img/nginx1.png)

## 负载均衡

方法：指定多台Tomcat（upstream）

缺点：
	请求可以到A server，也可以到B server，不受控制
	Session会话信息等，不能在保存到服务器上

## 热部署

**热部署**：配置文件nginx.conf修改后，不需要stop Nginx，就能让配置文件生效

1. 修改nginx.conf后，主进程master负责推送给woker进程更新配置信息，woker进程收到信息后，更新进程内部的线程信息（及时更新）
2. 修改nginx.conf后，重新生成新的worker进程，以新的配置进行处理新的请求，老的worker进程，等把那些以前的请求处理完毕后，kill掉即可（新换旧）

Nginx采用2

![image-20190811004733122](/Users/supeeeer/Documents/myGitHub/Notes/img/nginx.png)

## 并发处理

Nginx采用了Linux的**epoll模型**： epoll模型基于事件驱动机制（异步），它可以监控多个事件是否准备完毕，完成的就放入epoll队列中，worker只需要从epoll队列循环处理即可