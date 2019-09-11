# Nginx负载均衡算法

## 轮询(默认)

每个请求按时间顺序逐一分配到不同的后端服务，如果后端某台服务器死机，自动剔除故障系统，使用户访问不受影响。

```json
upstream bakend {  
    server 192.168.0.1;    
    server 192.168.0.2;  
}
```

## weight (轮询权重)

weight的值越大分配到的访问概率越高，主要用于后端每台服务器性能不均衡的情况下。或者仅仅为在主从的情况下设置不同的权值，达到合理有效的地利用主机资源。

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 

例如 : 

```json
upstream bakend {  
    server 192.168.0.1 weight=10;  
    server 192.168.0.2 weight=10;  
}
```

## ip_hash

每个请求按访问IP的哈希结果分配，使来自同一个IP的访客固定访问一台后端服务器，并且可以有效解决动态网页存在的session共享问题。

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。 
例如：

```json
upstream bakend {  
    ip_hash;  
    server 192.168.0.1:88;  
    server 192.168.0.2:80;  
} 
```

## Fair (第三方)

比 weight、ip_hash更加智能的负载均衡算法，fair算法可以根据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间 来分配请求，响应时间短的优先分配。Nginx本身不支持fair，如果需要这种调度算法，则必须安装upstream_fair模块。

按后端服务器的响应时间来分配请求，响应时间短的优先分配。 
例如：

```python
upstream backend {  
    server 192.168.0.1:88;  
    server 192.168.0.2:80;  
    fair;  
}
```

## url_hash（第三方）

按访问的URL的哈希结果来分配请求，使每个URL定向到一台后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身不支持url_hash，如果需要这种调度算法，则必须安装Nginx的hash软件包。

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

注意：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法。

例如：

```json
upstream backend {  
    server 192.168.0.1:88;  
    server 192.168.0.2:80;  
    # 指定负载均衡算法
    hash $request_uri;  
    # Hash算法
    hash_method crc32;  
}
```

## 自定义负载均衡算法

通过 lua脚本自定义负载均衡算法

