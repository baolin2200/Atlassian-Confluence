##Confluence迁移  
###安装JDK环境
<pre>
[root@confluence ~]# mkdir /application
[root@confluence ~]# tar xf jdk-8u91-linux-x64.gz -C /application/
[root@confluence ~]# cd /application/
[root@confluence application]# ln -s /application/jdk1.8.0_91/ /application/jdk
</pre>
* 修改环境变量生效：
<pre>
[root@confluence application]# sed -i.ori '$a export JAVA_HOME=/application/jdk\nexport PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH\nexport CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar' /etc/profile
[root@confluence application]# source /etc/profile
</pre>
* 检查java版本
<pre>
[root@confluence application]# java -version
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
</pre>

###下载Confluence
* 官网以及地址：
<pre>
https://www.atlassian.com/software/confluence/download
本次下载版本为：5.9.12
wget https://downloads.atlassian.com/software/confluence/downloads/atlassian-confluence-5.9.12-x64.bin
</pre>

###安装confluence
* 下载的为bin文件，按照指示自动安装：
<pre>
[root@new-confluence tools]# sh ./atlassian-confluence-5.9.12-x64.bin 
Unpacking JRE ...
Starting Installer ...
六月 20, 2016 7:24:44 下午 java.util.prefs.FileSystemPreferences$2 run
信息: Created system preferences directory in java.home.  
This will install Confluence 5.9.12 on your computer.
OK [o, Enter], Cancel [c]
o
Choose the appropriate installation or upgrade option.
Please choose one of the following:
Express Install (uses default settings) [1], 
Custom Install (recommended for advanced users) [2, Enter], 
Upgrade an existing Confluence installation [3]
1
See where Confluence will be installed and the settings that will be used.
Installation Directory: /opt/atlassian/confluence 
Home Directory: /var/atlassian/application-data/confluence 
HTTP Port: 8090 
RMI Port: 8000 
Install as service: Yes 
Install [i, Enter], Exit [e]
i   
Extracting files ...    
Please wait a few moments while Confluence starts up.
Launching Confluence ...
Installation of Confluence 5.9.12 is complete
Your installation of Confluence 5.9.12 is now ready and can be accessed via
your browser.
Confluence 5.9.12 can be accessed at http://localhost:8090
Finishing installation ...
</pre>

* 语言包下载地址：
<pre>
https://translations.atlassian.com
</pre>

* Copy源数据文件到数据的目录：
<pre>
[root@new-confluence confluence]# pwd
/var/atlassian/application-data/confluence
</pre>

* 注意权限```此处请注意```
<pre>
[root@new-confluence application-data]# ll
drwx------  3 confluence  root   42 6月  28 18:46 confluence 
</pre>

###nginx映射将端口直接映射到ip地址(nginx配置文件)：
* 安装及配置文件
<pre>
[root@new-confluence ~]# yum install nginx -y
[root@new-confluence ~]# cat /etc/nginx/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
\##
include conf.d/upstream/*.conf;
include conf.d/*.conf;

    }
</pre>
<pre>
[root@new-confluence ~]# cat /etc/nginx/conf.d/fluence.conf 
server {
    listen       80;
    server_name  192.168.10.42;
    location / {
        proxy_pass http://info/;
        client_max_body_size 100m;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For    $remote_addr;
        proxy_connect_timeout 90;
        proxy_send_timeout 90;
        proxy_read_timeout 90;
        proxy_buffer_size 4k;
        proxy_buffers 4 32k;
        proxy_busy_buffers_size 64k;
        proxy_temp_file_write_size 64k;
        }
}
[root@new-confluence ~]# cat /etc/nginx/conf.d/upstream/info.conf 
upstream info{
   server 127.0.0.1:8090;
}
</pre>

##版本总结：
* 地址：  
<pre>
IP地址：192.168.56.12
hostname：new-confluence
</pre>
* 版本：  
<pre>
java version "1.8.0_91"
Confluence 5.9.12 
nginx/1.6.3
</pre>
* jdk安装目录：
<pre>
jdk1.8.0_91 安装在 /application/ 目录下；
</pre>
* confluence安装目录 和 数据目录：
<pre>
Installation Directory: /opt/atlassian/confluence    
Home Directory: /var/atlassian/application-data/confluence   
</pre>
* 映射访问地址：   
<pre>
http://192.168.56.12/setup/setupstart.action
</pre>

