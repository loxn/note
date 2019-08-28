# 微服务的组件

1. 服务的注册发现
2. 服务网关
3. 后端通用服务 中间层服务 middle tier service
4. 前端服务，边缘服务 edge service



# spring cloud eureka

**eureka server**

供服务的注册的服务器

记录所有服务的信息和状态，应用的名字，地址，状态



**eureka client**

用来与服务器的交互，轮训负载均衡器，服务切换支持



服务发现的两种方式：

客户端发现：eureka

服务端发现：Nginx	



![1564379725146](D:\笔记\note\微服务\1564379725146.png)