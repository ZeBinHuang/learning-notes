## nginx安装

依赖插件

```
安装nginx需要先将官网下载的源码进行编译，编译依赖gcc环境，如果没有gcc环境，需要安装gcc
yum install gcc-c++
1.PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。
yum install -y pcre pcre-devel
2.zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。
yum install -y zlib zlib-devel
3.OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。
yum install -y openssl openssl-devel
```

安装nginx

```
nginx-1.16.1.tar.gz包拷贝到“/usr/local/src”下。然后解压。
创建文件夹：mkdir /usr/nginx
进入解压好的目录，执行：./configure --prefix=/usr/nginx
编译安装：make && make install
```

防火墙设置

```
查看开放的端口号 firewall-cmd --list-all 
 
设置开放的端口号 firewall-cmd --add-port=80/tcp --permanent 
 
重启防火墙 firewall-cmd --reload 
```



## 常用命令

```
进入 nginx/sbin 目录中
1、查看 nginx 版本号  ./nginx -v 
2、启动 nginx  ./nginx 
3、停止 nginx  ./nginx  -s  stop 
4、当修改了conf文件夹里面配置文件之后要执行的命令  ./nginx -s reload 
```



## 配置文件





## 反向代理

准备工作：

1.在 liunx 系统安装 tomcat，使用默认端口 8080 

2.对外开放访问的端口

 firewall-cmd --add-port=8080/tcp --permanent

 firewall-cmd –reload 

查看已经开放的端口号 firewall-cmd --list-all 

3.在 windows 系统中通过浏览器访问 tomcat 服务器 ：192.168.35.200:8080

反向代理实例一具体配置

```
1. 在 windows 系统的 host 文件进行域名和 ip 对应关系的配置 
打开C:\Windows\System32\drivers\etc\hosts文件，添加以下内容:
192.168.35.200 www.123.com
2. 在 nginx 进行请求转发的配置（反向代理配置）
host文件的名字要和nginx.conf文件的server_name相同,location中设置转发地址
 server {
        listen       80;
        server_name  www.123.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            proxy_pass http://127.0.0.1:8080;
            index  index.html index.htm;
        }
```

反向代理实例二具体配置

```
1.实现效果
使用 nginx 反向代理，根据访问的路径跳转到不同端口的服务中 nginx 监听端口为 9001，
访问 http://192.168.17.129:9001/edu/ 直接跳转到 127.0.0.1:8080 
访问 http:// 192.168.17.129:9001/vod/ 直接跳转到 127.0.0.1:8081 
2、准备工作 （1）准备两个 tomcat 服务器，一个 8080 端口，一个 8081 端口 
（2）在每个tomcat的webapp中创建对应的文件夹和测试页面
3、具体配置 
（1）找到 nginx 配置文件，进行反向代理配置 
 server {
        listen       9001;
        server_name  192.168.35.200;

        location ~/edu/ {
            proxy_pass http://localhsot:8080;
        }

        location ~/vod/ {
            proxy_pass http://localhost:8081;
        }
    }
（2）开放对外访问的端口号 9001 8080 8081 

location 指令说明 
  该指令用于匹配 URL。 
  语法如下： 
  1、= ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配 成功，就停止继续向下搜索并立即处理该请求。 
  2、~：用于表示 uri 包含正则表达式，并且区分大小写。 
  3、~*：用于表示 uri 包含正则表达式，并且不区分大小写。 
  4、^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字 符串匹配度最高的 location 后，立即使用此location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。 
  注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。 
```



## 负载均衡

```
1、实现效果 （1）浏览器地址栏输入地址 http://192.168.17.129/edu/a.html，负载均衡效果，平均 8080 和 8081 端口中 
2、准备工作 
（1）准备两台 tomcat 服务器，一台 8080，一台 8081 
（2）在两台 tomcat 里面 webapps 目录中，创建名称是 edu 文件夹，在 edu 文件夹中创建 页面 a.html，用于测试 
3、在 nginx 的配置文件中进行负载均衡的配置 
	upstream myserver{
        server 192.168.35.200:8080;
        server 192.168.35.200:8081;
    }

    server {
        listen       80;
        server_name  192.168.35.200;

        location / {
            root   html;
            proxy_pass http://myserver;
            index  index.html index.htm;
        }
4、nginx 分配服务器策略 
第一种 轮询（默认） 
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。 
 
第二种 weight 
weight 代表权重默认为 1,权重越高被分配的客户端越多 
	upstream myserver{
        server 192.168.35.200:8080  weight=10;
        server 192.168.35.200:8081  weight=10;
    }
    
第三种 ip_hash 
每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器 
	upstream myserver{
		ip_hash;
        server 192.168.35.200:8080;
        server 192.168.35.200:8081;
    } 
    
第四种 fair（第三方） 
按后端服务器的响应时间来分配请求，响应时间短的优先分配。 
	upstream myserver{
        server 192.168.35.200:8080;
        server 192.168.35.200:8081;
        fair;
    }
```



## 动静分离

```
1.准备资源
2.配置
 	location /www/{
        root /static/;
    }

	location /img/{
		root /static/;
	}
```



## 原理

1.一个 master 和多个 woker 有好处 

（1）可以使用 nginx –s reload 热部署，利用 nginx 进行热部署操作 

（2）每个 woker 是独立的进程，如果有其中的一个 woker 出现问题，其他 woker 独立的， 继续进行争抢，实现请求过程，不会造成服务中断 

2.设置多少个 woker 合适 

worker 数和服务器的 cpu 数相等是最为适宜的 

3.连接数 worker_connection 

第一个：发送请求，占用了 woker 的几个连接数？ 答案：2 或者 4 个 

第二个：nginx 有一个 master，有四个 woker，每个 woker 支持最大的连接数 1024，支持的 最大并发数是多少？ 

 普通的静态访问最大并发数是： worker_connections * worker_processes /2

 而如果是 HTTP 作 为反向代理来说，最大并发数量应该是 worker_connections *  worker_processes/4。



## 搭建高可用集群

```
1.准备工作
（1）需要两台服务器
（2）在两台服务器安装 nginx 
（3）在两台服务器安装 keepalived 
2.在两台服务器安装 keepalived 
（1）使用 yum 命令进行安装  yum install keepalived –y 
（2）安装之后，在 etc 里面生成目录 keepalived，有文件 keepalived.conf 
3.完成高可用配置（主从配置） 
（1）修改/etc/keepalived/keepalivec.conf 配置文件 
global_defs { 
   notification_email { 
     acassen@firewall.loc 
     failover@firewall.loc 
     sysadmin@firewall.loc 
   } 
   notification_email_from Alexandre.Cassen@firewall.loc 
   smtp_server 192.168.17.129 
   smtp_connect_timeout 30 
   router_id LVS_DEVEL 
} 
  
vrrp_script chk_http_port {  
   script "/usr/local/src/nginx_check.sh" 
   
   interval 2      #（检测脚本执行的间隔） 
  
   weight 2 
} 
  
vrrp_instance VI_1 {    
	state BACKUP   # 备份服务器上将 MASTER 改为 BACKUP      
    interface ens33  //网卡     
    virtual_router_id 51   # 主、备机的virtual_router_id 必须相同     
    priority 90     # 主、备机取不同的优先级，主机值较大，备份机值较小 
    advert_int 1 
    authentication { 
        auth_type PASS 
        auth_pass 1111 
    } 
    virtual_ipaddress {         
    	192.168.17.50 // VRRP H 虚拟地址 
    } 
} 
 
（2）在/usr/local/src 添加检测脚本 
#!/bin/bash 
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then     
	/usr/local/nginx/sbin/nginx     
	sleep 2     
	if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then        
   	 	killall keepalived    
    fi fi 
 
（3）把两台服务器上 nginx 和 keepalived 启动 
启动 nginx：./nginx 
启动 keepalived：systemctl start keepalived.service 

4.测试
（1）在浏览器地址栏输入 虚拟 ip 地址 192.168.17.50 
（2）把主服务器 nginx 和 keepalived 停止，再输入 192.168.17.50 
```















