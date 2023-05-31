# 正向代理
一般用于梯子\
```
location /{
    proxy_pass $scheme://$host$request_uri;
    //解决url带.导致的nginx503问题
    proxy_set_header Host $http_host;

    //关闭磁盘缓存读写，减少io
    proxy_max_temp_file_size 0;
    //设置代理连接超时
    proxy_connect_timeout = 30;

}
```

# 反向代理
一般用于负载均衡，隐藏服务器
```
upstream stream_name{
    //通过hash算法使统一的ip由统一的服务器接受，用于保持session一致
    ip_hash;
    //权重越高，访问量越大
    server 127.0.0.1:8080 weight=1;     //服务器1，权重1
    server 127.0.0.1:8080 weight=2;     //服务器2，权重2
}
server{
    listen 80;
    server_name localhost;
    location /{
        proxy_pass http://stream_name;
    }
}
```

