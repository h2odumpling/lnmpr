# 正向代理
一般用于梯子
```nginx
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
```nginx
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

# location
* rewrite regx replace;
使用正则匹配重写路由，将匹配到的部分重写为替换部分，可以进行部分替换
```nginx
location /admin {
    if (!-e $request_filename) {
        #rewrite  ^/admin$ /admin/;
        rewrite  ^/admin/(.*)$  /admin.php?s=$1  last;
        break;
    }
}
```

## php文件的配置
使用此配置，一般php通过PATH_INFO获取路由所需参数，而使用rewrite一般则是通过get参数的形式进行传递
```nginx
location ~ [^/]\.php(/|$) {
    fastcgi_pass   php73:9000;
    include        fastcgi_params;
    include        fastcgi-php.conf;
}
```
```nginx
# regex to split $uri to $fastcgi_script_name and $fastcgi_path
# 相当于把uri分成两个变量，一个是访问的php名称，一个是后面剩余的链接（一般被用作参数，给PATH_INFO赋值）
fastcgi_split_path_info ^((?U).+\.php)(/.+)$;

# 设置这条会将PATH_INFO变量传递到$_SERVER的PATH_INFO中，如果不设置会导致很多框架的无法获取到路由
fastcgi_param PATH_INFO $fastcgi_path_info;

# 如果未设置PATH_INFO则使用以下重写规则代替，这会将路由当作get参数传递到php中
# rewrite ^((?U).+\.php)(/.+)$ $1?s=$2;

# Check that the PHP script exists before passing it
# try_files $fastcgi_script_name =404;

fastcgi_read_timeout 3600;

fastcgi_index index.php;
```


