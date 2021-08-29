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