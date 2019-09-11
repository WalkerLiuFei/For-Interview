# Nginx应用

## 正向代理和反向代理

### 反向代理

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

**结论就是，反向代理服务器对于客户端而言它就像是原始服务器，并且客户端不需要进行任何特别的设置**。

1. **保护了真实的web服务器，保证了web服务器的资源安全**
2. **节约有限的IP地址资源**
3. **减少WEB服务器压力,提高响应速度** : 通过在繁忙的web服务器和外部网络之间增加一个高速的web缓冲服务器来降低实际的web服务器的负载的一种技术 (代理缓存).
4. **其他优点**
   1. 请求的统一控制，包括设置权限、过滤规则等
   2. IP 白名单 / 黑名单功能.
   3. 区分动态和静态可缓存内容；
   4. 实现负载均衡，内部可以采用多台服务器来组成服务器集群，外部还是可以采用一个地址访问；
   5. 解决Ajax跨域问题；
   6. 限流  : 作为真实服务器的缓冲，解决瞬间负载量大的问题.

![反向代理](https://upload-images.jianshu.io/upload_images/6807865-90603b54f3e3e521.png?imageMogr2/auto-orient/strip|imageView2/2/w/354/format/webp)  

### 正向代理

正向代理（Forward Proxy）通常都被简称为代理，就是在用户无法正常访问外部资源，比方说受到GFW的影响无法访问twitter的时候，我们可以通过代理的方式，让用户绕过防火墙，从而连接到目标网络或者服务。

**结论就是，正向代理是一个位于客户端和原始服务器(origin server)之间的服务器**。

![img](https://upload-images.jianshu.io/upload_images/6807865-2cede76e2384c39f.png?imageMogr2/auto-orient/strip|imageView2/2/w/349/format/webp) 

## Nginx 限流原理

