# nginx 入门

##  处理跨域的两种方式

首先有两个域名分别是 http://source.website.com:9002 和 http://target.website.com:9001 。现在从source去访问target则会产生跨域。有以下两种方式处理

### 方式一：在source端的网站配置反向代理，目前前端开发阶段常用这个，nginx配置如下：
```nginx
server {
    listen       9002;
    server_name  source.website.com;

    location / {
        root   html/source;
        index  index.html index.htm;
    }

    location ^~/apis/ {
        # 这里重写了请求，将正则匹配中的第一个分组的path拼接到真正的请求后面，并用break停止后续匹配
        rewrite ^/apis/(.*)$ /$1 break;
        proxy_pass http://target.website.com:9001/;
        # 两个域名之间cookie的传递与回写
        proxy_cookie_domain target.website.com source.website.com;
    }
} 
```
```javacript
  $.get("/apis/data.json")
```
通过把当前网站source的根目录路径转发到对应target网站路径

### 方式一：目标服务器配置headers允许访问
```nginx
server {
    listen       9001;
    server_name  target.website.com;

	add_header 'Access-Control-Allow-Origin' $http_origin;   # 全局变量获得当前请求origin，带cookie的请求不支持*
	add_header 'Access-Control-Allow-Credentials' 'true';    # 为 true 可带上 cookie
	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';  # 允许请求方法
	add_header 'Access-Control-Allow-Headers' $http_access_control_request_headers;  # 允许请求的 header，可以为 *
	add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
    if ($request_method = 'OPTIONS'){
       return 204;                  # 200 也可以
	}

    location / {
        root   html/target;
        index  index.html index.htm;
    }
} 
```

## 配置GZip

```nginx
# /etc/nginx/conf.d/gzip.conf
gzip on; # 默认off，是否开启gzip
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

# 详细配置，进阶使用
gzip_static on;
gzip_proxied any;
gzip_vary on;
gzip_comp_level 6;
gzip_buffers 16 8k;
# gzip_min_length 1k;
gzip_http_version 1.1;

```

参数说明：

**gzip_types**：要采用 gzip 压缩的 MIME 文件类型，其中 text/html 被系统强制启用；

**gzip_static**：默认 off，该模块启用后，Nginx 首先检查是否存在请求静态文件的 gz 结尾的文件，如果有则直接返回该 `.gz` 文件内容；

**gzip_proxied**：默认 off，nginx做为反向代理时启用，用于设置启用或禁用从代理服务器上收到相应内容 gzip 压缩；

**gzip_vary**：用于在响应消息头中添加 `Vary：Accept-Encoding`，使代理服务器根据请求头中的 `Accept-Encoding` 识别是否启用 gzip 压缩；

**gzip_comp_level**：gzip 压缩比，压缩级别是 1-9，1 压缩级别最低，9 最高，级别越高压缩率越大，压缩时间越长，建议 4-6；

**gzip_buffers**：获取多少内存用于缓存压缩结果，16 8k 表示以 8k*16 为单位获得；

**gzip_min_length**：允许压缩的页面最小字节数，页面字节数从header头中的 `Content-Length` 中进行获取。默认值是 0，不管页面多大都压缩。建议设置成大于 1k 的字节数，小于 1k 可能会越压越大；

**gzip_http_version**：默认 1.1，启用 gzip 所需的 HTTP 最低版本；



这个配置可以插入到 http 模块整个服务器的配置里，也可以插入到需要使用的虚拟主机的 `server` 或者下面的 `location` 模块中，当然像上面我们这样写的话就是被 include 到 http 模块中了。



## 常用技巧

### 静态服务

```nginx
# static.conf
server {
  listen       80;
  server_name  static.zhuowenzhou.com;
  charset utf-8;    # 防止中文文件名乱码

  location /download {
    alias	          /usr/local/var/www/static;  # 静态资源目录
    autoindex               on;    # 开启静态资源列目录
    autoindex_exact_size    off;   # on(默认)显示文件的确切大小，单位是byte；off显示文件大概大小，单位KB、MB、GB
    autoindex_localtime     off;   # off(默认)时显示的文件时间为GMT时间；on显示的文件时间为服务器时间
  }
}

```

### 图片防盗链

```nginx
server {
  listen       80;        
  server_name  *.sherlocked93.club;
  
  # 图片防盗链
  location ~* \.(gif|jpg|jpeg|png|bmp|swf)$ {
    valid_referers none blocked server_names ~\.google\. ~\.baidu\. *.qq.com;  # 只允许本机 IP 外链引用，允许指定域名加入白名单
    if ($invalid_referer){
      return 403;
    }
  }
}

```

### 请求过滤

```nginx
# 非指定请求全返回 403
if ( $request_method !~ ^(GET|POST|HEAD)$ ) {
  return 403;
}

location / {
  # IP访问限制（只允许IP是 192.168.0.2 机器访问）
  allow 192.168.0.2;
  deny all;
  
  root   html;
  index  index.html index.htm;
}

```

### 配置图片、字体等静态文件缓存

```nginx
# 图片缓存时间设置
location ~ .*\.(css|js|jpg|png|gif|swf|woff|woff2|eot|svg|ttf|otf|mp3|m4a|aac|txt)$ {
	expires 10d;
}

# 如果不希望缓存
expires -1;

```

### 单页面项目 history 路由配置

```nginx
server {
  listen       80;
  server_name  blog/zhuowenzhou.club;
  
  location / {
    root       /usr/share/nginx/html/dist;  # vue 打包后的文件夹
    index      index.html index.htm;
    try_files  $uri $uri/ /index.html @rewrites;  
    
    expires -1;                          # 首页一般没有强制缓存
    add_header Cache-Control no-cache;
  }
  
  # 接口转发，如果需要的话
  #location ~ ^/api {
  #  proxy_pass http://api.zhuowenzhou.club;
  #}
  
  location @rewrites {
    rewrite ^(.+)$ /index.html break;
  }
}

```

### HTTP 请求转发到 HTTPS

```nginx
server {
    listen      80;
    server_name www.zhuowenzhou.club;

    # 单域名重定向
    if ($host = 'www.zhuowenzhou.club'){
        return 301 https://www.zhuowenzhou.club$request_uri;
    }
    # 全局非 https 协议时重定向
    if ($scheme != 'https') {
        return 301 https://$server_name$request_uri;
    }

    # 或者全部重定向
    return 301 https://$server_name$request_uri;

    # 以上配置选择自己需要的即可，不用全部加
}

```

### 泛域名路径分离

```
server {
    listen       80;
    server_name  ~^([\w-]+)\.doc\.zhuowenzhou\.com$;

    root /usr/share/nginx/html/doc/$1;
}

```

`test1.zhuowenzhou.com` 自动指向 `/usr/share/nginx/html/doc/test1` 服务器地址；

`test2zhuowenzhou.com` 自动指向 `/usr/share/nginx/html/doc/test2` 服务器地址；

### 泛域名转发

```nginx
server {
    listen       80;
    server_name ~^([\w-]+)\.serv\.zhuowenzhou\.club$;

    location / {
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        Host $http_host;
        proxy_set_header        X-NginX-Proxy true;
        proxy_pass              http://127.0.0.1:8080/$1$request_uri;
    }
}
	
```

`t1.serv.zhuowenzhou.com/api?name=acs` 自动转发到 `127.0.0.1:8080/t1/api?name=acs` ；

`t2.serv.zhuowenzhou.com/api?name=acs` 自动转发到 `127.0.0.1:8080/t2/api?name=acs `；

## 最佳实践

1. 为了使 Nginx 配置更易于维护，建议为每个服务创建一个单独的配置文件，存储在 `/etc/nginx/conf.d `目录，根据需求可以创建任意多个独立的配置文件。
2. 独立的配置文件，建议遵循以下命名约定 `<服务>.conf`，比如域名是 zhuowenzhou.com，那么你的配置文件的应该是这样的 `/etc/nginx/conf.d/zhuowenzhou.com.conf`，如果部署多个服务，也可以在文件名中加上 Nginx 转发的端口号，比如` zhuowenzhou.com.8080.conf`，如果是二级域名，建议也都加上 `fe.zhuowenzhou.com.conf`。
3. 常用的、复用频率比较高的配置可以放到 `/etc/nginx/snippets `文件夹，在 Nginx 的配置文件中需要用到的位置 include 进去，以功能来命名，并在每个 snippet 配置文件的开头注释标明主要功能和引入位置，方便管理。比如之前的 `gzip、cors` 等常用配置，我都设置了 `snippet`。
4. Nginx 日志相关目录，内以 `域名.type.log` 命名（比`be.zhuowenzhou.com.access.log` 和 `be.zhuowenzhou.com.error.log` ）位于 /var/log/nginx/ 目录中，为每个独立的服务配置不同的访问权限和错误日志文件，这样查找错误时，会更加方便快捷。

