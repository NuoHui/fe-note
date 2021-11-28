## 配置文件

<img src="https://note-assets-1257150743.cos.ap-guangzhou.myqcloud.com/img/%E6%88%AA%E5%B1%8F2021-11-28%20%E4%B8%8B%E5%8D%881.00.10.png" alt="截屏2021-11-28 下午1.00.10" style="zoom:50%;" />

```nginx
# 指定启动nginx worker的用户
#user  nobody;

# worker进程数量、与cpu核数有关，建议Max-1
worker_processes  1;

# 错误日志 路径 级别 debug info notice warn error crit
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

# 进程Id号
#pid        logs/nginx.pid;

# 指令块
events {
    # 默认使用epoll
    use epoll;
    # 每个worker进程允许连接的客户端最大连接数
    worker_connections  1024;
}


http {
    # 导入文件 
    include       mime.types;
    # 默认类型
    default_type  application/octet-stream;
  
  
    # 自定义日志格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    # 用户请求日志路径 格式，可以全局配置
    #access_log  logs/access.log  main;

  
    # 文件上传
    sendfile        on;
    # TCP_NOPUSH 是 FreeBSD 的一个 socket 选项，对应 Linux 的 TCP_CORK，Nginx 里统一用 tcp_nopush 来控制它，并且只有在启用了 sendfile 之后才生效。启用它之后，数据包会累计到一定大小之后才会发送，减小了额外开销，提高网络效率。
    #tcp_nopush     on;

  
  	# keepalive_timeout timeout [header_timeout];
    # 通过配置 keepalive_timeout 可以实现服务器与客户端之间的长连接，用户连接访问服务器可能不止 1 次请求，使用keepalive可以减少系统对TCP连接的建立和销毁的开销。
    # keepalive_timeout 65;　服务器端主动发起keepalive连接，并保持 65 秒，如果超过65秒没有发送任何请求，则服务器主动断开连接，TCP从ESTABLISHED 转为 TIME_WAIT 状态。
    # 第一个参数：设置keep-alive客户端连接在服务器端保持开启的超时值（默认65s）；值为0会禁用keep-alive客户端连接；
    # 第二个参数：可选、在响应的header域中设置一个值“Keep-Alive: timeout=time”；通常可以不用设置；
    #keepalive_timeout  0;
    keepalive_timeout  65;

  
    # 开启gzip压缩
    #gzip  on;

    server {
        # 监听端口
        listen       8080;
        # 域名 IP
        server_name  localhost;

        #charset koi8-r;

        # 在server虚拟主机定义访问日志
        #access_log  logs/host.access.log  main;
        
        # 路由匹配
        location / {
            # nginx指定文件路径的方式有两种，root以及alias、root使用的是相对路径
            # 如果location是 ^~ /t/, root /www/root/html/； 一个请求的URI是/t/a.html时，web服务器将会返回服务器上的/www/root/html/t/a.html的文件。
            # 如果location是 ^~ /t/, alias /www/root/html/new_t/； 一个请求的URI是/t/a.html时，web服务器将会返回服务器上的/www/root/html/new_t/a.html的文件。注意这里是new_t，因为alias会把location后面配置的路径丢弃掉，把当前匹配到的目录指向到指定的目录。使用alias时，目录名后面一定要加"/"。
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include servers/*;
}
```



## 压缩

```nginx
# 开启压缩功能：提高传输效率、节约带宽
gzip on;
# 限制最小压缩：小于1字节文件不会压缩
gzip_min_length: 1;
# 定义压缩的级别、压缩比。文件越大压缩越多，相对cpu使用越多
gzip_comp_level: 3;
# 定义文件压缩类型
gzip_types: text/plain;
```

查看`nginx`进程信息。

```shell
› ps -ef|grep nginx
  501 16988     1   0  1:13PM ??         0:00.01 nginx: master process /usr/local/opt/nginx/bin/nginx -g daemon off;
  501 17186 16988   0  1:13PM ??         0:00.00 nginx: worker process
  501 17214 16557   0  1:14PM ttys004    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox nginx
```



## location匹配规则

```nginx
# = 表示精确匹配，只有完全匹配才生效
# location = /uri

# http://website.com/abcd匹配
# http://website.com/ABCD可能会匹配 ，也可以不匹配，取决于操作系统的文件系统是否大小写敏感（case-sensitive）
# http://website.com/abcd?param1&param2匹配，忽略 querystring
# http://website.com/abcd/不匹配，带有结尾的/
# http://website.com/abcde不匹配
server {
  server_name website.com;
  location = /abcd {
    […]
  }
}


# ~表示区分大小写的正则匹配
# location ~ pattern
# ^/abcd$这个正则表达式表示字符串必须以/开始，以$结束，中间必须是abcd
# http://website.com/abcd匹配（完全匹配）
# http://website.com/ABCD不匹配，大小写敏感
# http://website.com/abcd?param1&param2匹配
# http://website.com/abcd/不匹配，不能匹配正则表达式
# http://website.com/abcde不匹配，不能匹配正则表达式
server {
  server_name website.com;
  location ~ ^/abcd$ {
    […]
  }
}

# ~* 表示不区分大小写的正则匹配
# http://website.com/ABCD匹配 (大小写不敏感)
location ~* pattern 
server {
    server_name website.com;
    location ~* ^/abcd$ {
    […]
    }
}

# 不带任何修饰符 也表示前缀匹配 但是在正则匹配之后
location /uri

#  通用匹配 任何未匹配到其他的Location的请求都会匹配到
location /
```

匹配顺序：

- 首先精确匹配 `=`
- 其次前缀匹配 `^~`
- 其次是按文件中顺序的正则匹配
- 然后匹配不带任何修饰的前缀匹配。
- 最后是交给 `/` 通用匹配
- 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

## nginx跨域

```nginx
server {
  listen 90;
  server_name: localhost;
  # 允许跨域请求的域
  add_header 'Access-Control-Allow-Origin' '*';
  # 允许请求带上cookie
  add_header 'Access-Control-Allow-Credentials' 'true';
  # 允许请求的方法 比如GET、POST、PUT、DELETE
  add_header 'Access-Control-Allow-Methods' '*';
  # 允许请求的header
  add_header 'Access-Control-Allow-Headers' '*';
  location / {
  	root /home;
  	index index.html;
  }
}
```



## 静态资源防盗链

```nginx
location ~* .*\.(gif|jpg|ico|png|css|svg|js)$ {
  root /usr/local/nginx/static;
  valid_referers none blocked  *.gupao.com ;
  if ($invalid_referer) {
    #rewrite ^/ http://www.youdomain.com/404.jpg;
    return 403;
    break;
  }
  access_log off;
}
```



## 日志切割

现有日志一般存在`access.log`文件中，但是对于大型项目随着日志内容越来越多，不方便查看，一般我们会对日志进行切割，比如按照天、或者小时为单位。

下面为示例脚本：`cut_log.sh`

```shell
#!/bin/bash
LOG_PATH="/var/log/nginx/"
RECORD_TIME=$(date -d "yesterday" + %Y-%m-%d+%H:%M)
PID=/var/run/nginx/nginx.pid
mv ${LOG_PATH}/access.log ${LOG_PATH}/access.${RECORD_TIME}.log 
mv ${LOG_PATH}/error.log ${LOG_PATH}/error.${RECORD_TIME}.log
# 向Nginx主进程发送信号，用于重新打开日志文件
kill -USR1 `cat $PID`
```

为`cut_log.sh`添加可执行的权限：

```bash
chmod + x cut_log.sh
```

改为定制切割日志

- install 定时任务

```bash
yum install crontabs
```

- `crontab -e` 编辑添加新任务

```bash
*/1 **** /usr/local/nginx/sbin/cut_log.sh
```

- 重启定时任务

```
service crond restart # 重启
service crond start # 启动
service crond stop # 关闭
service crond reload # 重载
crontab -e # 编辑任务
crontab -l # 查看任务列表
```



## 常用命令

```shell
./nginx -s stop # 强制关闭Nginx
./nginx -s quit # 优雅退出
./nginx -t # 校验Nginx配置文件语法
./nginx -v
./nginx -V
./nginx -h # help
./nginx -c # 指定配置文件
./nginx -s reload # 热重载
```

