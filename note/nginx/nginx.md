1. http://nginx.org/en/download.html

2. https://www.cnblogs.com/liang-wei/p/5849771.html

3. yum install gcc

   yum install gcc-c++

   yum install -y pcre pcre-devel

   yum install -y zlib zlib-devel

   yum install -y openssl openssl-devel

4. 安装

   ./configure \

   --prefix=/usr/local/nginx \

   --pid-path=/var/run/nginx/nginx.pid \

   --lock-path=/var/lock/nginx.lock \

   --error-log-path=/var/log/nginx/error.log \

   --http-log-path=/var/log/nginx/access.log \

   --with-http_gzip_static_module \

   --http-client-body-temp-path=/var/temp/nginx/client \

   --http-proxy-temp-path=/var/temp/nginx/proxy \

   --http-fastcgi-temp-path=/var/temp/nginx/fastcgi \

   --http-uwsgi-temp-path=/var/temp/nginx/uwsgi \

   --http-scgi-temp-path=/var/temp/nginx/scgi

5. mkdir -p /var/temp/nginx

   make

   make install

6. 启动和关闭

   cd /usr/local/nginx/sbin/

   ./nginx

   ./nginx -c /usr/local/nginx/conf/nginx.conf	（指定配置文件启动）

   ./nginx -s stop		（快速停止）

   ./nginx -s quit		 （推荐）

   ./nginx -s reload  	 (重启)

7. 虚拟主机配置

   aa.test.com	访问	/usr/local/nginx/virtual_host_test/a_html  目录

   bb.test.com	访问	/usr/local/nginx/virtual_host_test/b_html  目录

   此时如果直接ip访问，则访问 a_html，即从上往下

       server{  
           listen          80; 
           server_name     aa.test.com;
           location  /     {
                  root    /usr/local/nginx/virtual_host_test/a_html;
                  index   index.html index.htm;
            }
       } 
       server{ 
           listen          80; 
           server_name     bb.test.com;
           location  /     {
                   root    /usr/local/nginx/virtual_host_test/b_html;
                   index   index.html index.htm;
           }
       }

8. 反向代理

   访问 aa.test.com	指向node1:8080

   ```
   # 此处不能用 _ ,1.8.1版本之后
   upstream tomcat-server1 {
        server 192.168.11.20:8080;
   }
   server {
   	listen 80;
       server_name aaa.test.com;
       location / {
           proxy_pass http://tomcat-server1;
           index index.jsp index.html index.htm;
       }
   }
   ```

9. 动静分离

   ```
   
   ```

   

10. 负载均衡

   ```
   upstream tomcat-server {
   	server node1:8080 weight=10;
   	server node2:8080 weight=10;
   }
   server {
   	listen          80;
   	server_name     aa.test.com;
   	location / {
   		proxy_pass  http://tomcat-server;
   		index       index.jsp index.html index.htm;
   	}
   }
   ```

11. 高可用

    a.安装keepalived

    ​	前置yum install openssl

    ​	yum install keepalived

    b.配置keepalived

    ​	主机

    ```
    global_defs {
       notification_email {
         loxn1827@foxmail.com
       }
       notification_email_from loxn1827@foxmail.com
       smtp_server 127.0.0.1
       smtp_connect_timeout 30
       router_id LVS_DEVEL				#运行keepalived机器的一个标识
    }
    # 默认情况，下，keepalived 进程退出才会 IP 漂移，这个脚本实现 nginx 进程退出也实现漂移
    vrrp_script check_nginx {
        script "/etc/keepalived/check_nginx.sh"         ##监控脚本
        interval 2                                      ##时间间隔，2秒
        weight 2                                        ##权重
    }
    vrrp_instance VI_1 {
        state MASTER					#标示状态为MASTER 备份机为BACKUP
        interface eth0					#设置实例绑定的网卡
        virtual_router_id 51	 		#同一实例下virtual_router_id必须相同
        priority 100						#MASTER权重要高于BACKUP 比如BACKUP为99
        advert_int 1					#MASTER与BACKUP负载均衡器之间同步检查的时间间隔，单位是秒
        authentication {
            auth_type PASS				#主从服务器验证方式
            auth_pass 1111
        }
        track_script {
            check_nginx        #监控脚本
        }
        virtual_ipaddress {
            192.168.11.200				#可以多个虚拟IP
        }
    }
    ```

    从机

    ```
    global_defs {
       notification_email {
         loxn1827@foxmail.com
       }
       notification_email_from loxn1827@foxmail.com
       smtp_server 127.0.0.1
       smtp_connect_timeout 30
       router_id LVS_DEVEL
    }
    vrrp_script check_nginx {
        script "/etc/keepalived/check_nginx.sh"         ##监控脚本
        interval 2                                      ##时间间隔，2秒
        weight 2                                        ##权重
    }
    vrrp_instance VI_1 {
        state BACKUP					# 从机为BACKUP
        interface eth0
        virtual_router_id 51			# ID 和主机一致
        priority 99						# 优先级比主机低
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 1111
        }
        track_script {
            check_nginx        #监控脚本
        }
        virtual_ipaddress {
            192.168.11.200				# 和主机一致
        }
    }
    
    ```

    c.配置nginx

    ```
    upstream tomcat-server {
    	server node1:8080 weight=10;
    	server node2:8080 weight=10;
    }
    server {
    	listen          80;
    	server_name     cc.test.com;
    	location / {
    		proxy_pass  http://tomcat-server;
    		index       index.jsp index.html index.htm;
    	}
    }
    ```

    d. hosts

    192.168.11.200	cc.test.com

    e.访问cc.test.com

    f.kill 主机keepalived进程，发现 VIP 漂移到从机上。再次启动主机，VIP又会回到主机上

12. web缓存

    

13. https转http

/usr/local/tjweb/webapps/ROOT/nginx.conf
/etc/nginx/conf.d/nginx.conf
/etc/nginx/nginx.conf













