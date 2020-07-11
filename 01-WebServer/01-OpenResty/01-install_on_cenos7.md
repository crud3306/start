


OpenResty只是一个增强版的 Nginx。


两种安装方式
============
- yum安装
- 源码编译安装


yum安装
============

设置安装的yum源
------------
你可以在你的 CentOS 系统中添加 openresty 仓库，这样就可以便于未来安装或更新我们的软件包（通过 yum update 命令）。运行下面的命令就可以添加我们的仓库：
```sh
sudo yum install yum-utils -y
sudo yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
```

安装Openresty
------------
然后就可以像下面这样安装软件包，比如 openresty：
```sh
sudo yum install openresty -y
```


安装命令行工具 resty (如果不用也可不用安装)
-------------
安装 openresty-resty 包：
> sudo yum install openresty-resty -y

安装之后，就可以查看一下版本号，如下：
```sh
[root@centos7 ~]# resty -V
resty 0.21
nginx version: openresty/1.13.6.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
built with OpenSSL 1.1.0h  27 Mar 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/openresty/nginx --with-cc-opt='-O2 -DNGX_LUA_ABORT_AT_PANIC -I/usr/local/openresty/zlib/include -I/usr/local/openresty/pcre/include -I/usr/local/openresty/openssl/include' --add-module=../ngx_devel_kit-0.3.0 --add-module=../echo-nginx-module-0.61 --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2rc3 --add-module=../set-misc-nginx-module-0.32 --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.08 --add-module=../srcache-nginx-module-0.31 --add-module=../ngx_lua-0.10.13 --add-module=../ngx_lua_upstream-0.07 --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.7 --add-module=../ngx_stream_lua-0.0.5 --with-ld-opt='-Wl,-rpath,/usr/local/openresty/luajit/lib -L/usr/local/openresty/zlib/lib -L/usr/local/openresty/pcre/lib -L/usr/local/openresty/openssl/lib -Wl,-rpath,/usr/local/openresty/zlib/lib:/usr/local/openresty/pcre/lib:/usr/local/openresty/openssl/lib' --with-pcre-jit --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_v2_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --with-http_stub_status_module --with-http_realip_module --with-http_addition_module --with-http_auth_request_module --with-http_secure_link_module --with-http_random_index_module --with-http_gzip_static_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-threads --with-dtrace-probes --with-stream --with-stream_ssl_module --with-http_ssl_module
```


配置路径
-----------
/xxx/openresty/bin/openresty其实是/xxx/openresty/nginx/sbin/nginx的软链
```sh
[root@f36b6d58b3f2 openresty_work]# ll /usr/local/openresty/bin/
total 0
lrwxrwxrwx 1 root root 37 Jun  3 22:07 openresty -> /usr/local/openresty/nginx/sbin/nginx
```

> vim /etc/profile
在最后面添加
```sh
export PATH=/usr/local/openresty/nginx/sbin:$PATH
```
使配置生效
> source /etc/profile

查看OpenResty版本
> nginx -V






源码编译安装
===========
安装
--------------
依赖库安装
> yum install pcre-devel openssl-devel

官网下载源码包安装 
http://openresty.org/cn/download.html
```sh
wget https://openresty.org/download/openresty-1.15.8.2.tar.gz
tar -zxvf openresty-1.15.8.2.tar.gz

cd openresty-1.15.8.2
./configure --prefix=/usr/local/openresty\
              --with-luajit\
              --without-http_redis2_module \
              --with-http_iconv_module
gmake
gmake install

#这里也可用make，注意看configure成功后的提示来选择
make
make install
```


安装位置
---------------
openresty 文件如下
```sh
ll /usr/local/openresty
drwxr-xr-x.  2 root root   4096 Jan 20 07:33 bin
-rw-r--r--.  1 root root  22924 Jan 20 07:33 COPYRIGHT
drwxr-xr-x.  6 root root     52 Jan 20 07:33 luajit
drwxr-xr-x.  6 root root   4096 Jan 20 07:33 lualib
drwxr-xr-x. 11 root root   4096 Jan 20 07:35 nginx
drwxr-xr-x. 47 root root   4096 Jan 20 07:33 pod
-rw-r--r--.  1 root root 226755 Jan 20 07:33 resty.index
drwxr-xr-x.  5 root root     44 Jan 20 07:33 site

#/usr/local/openresty/bin/openresty实际上软链至/usr/local/openresty/nginx/sbin/nginx
```

为了工作目录与安装目录互不干扰，我们在当前用户的家目录下创建目录
```sh
$ cd /home/qym/work
$ mkdir openresty-test openresty-test/logs/ openresty-test/conf/ openresty-test/conf/conf.d
$
$ tree ./openresty-test
/home/qym/work/openresty-test
├── conf
└── logs
```



执行
---------------
```sh
#启动：
/usr/local/openresty/bin/openresty -p /home/qym/work/openresty-test/ -c conf/nginx.conf

/usr/local/openresty/bin/openresty -c /home/qym/work/openresty-test/conf/nginx.conf

#重新加载配置
/usr/local/openresty/bin/openresty -p `pwd` -s reload

#-p 指定工作目录 
#-c 指定配置文件，-c配合-p时，-c后面可以用相对目录


#重新加载配置
/usr/local/openresty/bin/openresty -s reload

#停止
/usr/local/openresty/bin/openresty -s stop
```






使用入门
===========
在上面已经安装好了Openresty之后，下面可以部署一个Hello world的示例。
 

创建相关工作目录
-----------
创建work目录以及相应的log日志目录、conf配置目录
```sh 
mkdir ~/work
cd ~/work
mkdir openresty_work
cd openresty_work
mkdir logs/ conf/
```


准备nginx.conf配置文件
-----------
创建一个简单的纯文本文件 conf/nginx.conf：
```sh
cd ~/work/openresty_work
vim conf/nginx.conf
```
其中包含以下内容：
```sh
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, world</p>")
            ';
        }
    }
}
```


以这种方式用我们的配置文件启动nginx服务器：
-----------
```sh
nginx -p ~/work/openresty_work/ -c conf/nginx.conf 

#或者
cd ~/work/openresty_work/
nginx -p `pwd`/ -c conf/nginx.conf 
```

查看启动情况
```sh
#查看进程
[root@f36b6d58b3f2 openresty_work]# ps -ef|grep nginx
root        83     1  0 22:17 ?        00:00:00 nginx: master process nginx -p /root/work/openresty_work/ -c conf/nginx.conf
nobody      84    83  0 22:17 ?        00:00:00 nginx: worker process
root        98    14  0 22:35 pts/1    00:00:00 grep --color=auto nginx

#查看服务
[root@f36b6d58b3f2 openresty_work]# netstat -tunpl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      83/nginx: master pr
```


访问我们的HelloWorld Web服务
-----------
访问 http://localhost:8080/
```sh
[root@f36b6d58b3f2 openresty_work]# curl http://localhost:8080/
<p>hello, world</p>
```







