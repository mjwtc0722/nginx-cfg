# nginx-cfg
Nginx反向代理配置脚本
通过脚本参数实现Nginx反向代理的Server模块配置，配置格式如下
```
# 默认域名反向代理配置
server {
       listen 80;
       server_name www.example.com;
       location / {
           proxy_pass http://www.example.com;
           proxy_redirect off;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           access_log logs/www.example.com main;
       }
}
upstream www.example.com {
       server 10.0.0.1:80;
       server 10.0.0.2:80;
}

# 默认域名及路径反向代理配置
server {
       listen 80;
       server_name www.example.com;
       location / {
           proxy_pass http://www.example.com;
           proxy_redirect off;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           access_log logs/www.example.com main;
       }
       location ^~ /directory/ {
           proxy_pass http://www.example.com.directory;
           proxy_redirect off;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           access_log logs/www.example.com.directory main;
       }
}
upstream www.example.com {
       server 10.0.0.1:80;
       server 10.0.0.2:80;
}
upstream www.example.com.directory {
       server 10.0.0.3:80;
       server 10.0.0.4:80;
}
```
# 使用说明
```
# 需修改的脚本内容
bin=/usr/local/nginx/sbin/nginx              #Nginx可执行程序路径
conf=/usr/local/nginx/conf/nginx.conf        #Nginx主配置文件路径
conf_directory=/usr/local/nginx/conf/vhost   #Nginx虚拟站点配置文件目录，在主配置文件中使用"include  vhost/*.conf;"包含
bak_directory=/root/nginx_backup             #Nginx备份文件路径
```

# 使用方法
```
  nginx-cfg -t [type] -s [server_name] -h [ip:port] -d [directory]
```
# 参数说明
```
	-t [type] :脚本执行的模式
		   add|a    添加域名或目录映射（默认模式）
		   modify|m 修改域名的upstream映射配置
		   delete|d 删除域名或目录映射
		   search|s 查看指定域名或目录的映射
	-s [server_name] :配置的域名，例如:www.yihu.com
	-h [ip:port] :映射的主机地址及端口，多条记录以‘,’隔开,例如:’10.0.0.1:80,10.0.0.2:80’
	-d [directory] :location目录
```
# 使用案例
```
  1、添加一个域名www.example.com，映射到主机10.0.0.1，端口80
	nginx-cfg -t add -s www.example.com -h 10.0.0.1:80

  2、添加一个域名www.example.com,映射到主机10.0.0.2和10.0.0.3，端口都为8080
	nginx-cfg -t add -s www.example.com -h “10.0.0.2:8080,10.0.0.3:8080”

  3、添加一个目录映射，域名为www.example.com，目录为directory，映射到主机10.0.0.4:80和10.0.0.5:8080
	nginx-cfg -t add -s www.example.com -d directory -h “10.0.0.4:80,10.0.0.5:8080”

  4、修改一个域名的映射，现有www.example.com映射至10.0.0.1:80，现在要加一台负载10.0.0.2:80
	nginx-cfg -t modify -s www.example.com -h “+10.0.0.2:80”

  5、修改一个域名的映射，现有www.example.com映射至10.0.0.1:80和10.0.0.2:80，现在要删一台负载10.0.0.1:80
	nginx-cfg -t modify -s www.example.com -h “-10.0.0.1:80”

  6、修改一个域名的映射，现有www.example.com,目录为directory,映射至10.0.0.2:80，现在要替换映射为10.0.0.3:80和10.0.0.4:80
	nginx-cfg -t modify -s www.example.com -d directory -h “=10.0.0.3:80,10.0.0.4:80”

  7、删除域名www.example.com下所有映射
	nginx-cfg -t delete -s www.example.com

  8、删除域名www.example.com下目录directory的映射
	nginx-cfg -t delete -s www.example.com -d directory

  9、查看域名www.example.com对应的映射
	nginx-cfg -t search -s www.example.com
  
  10、查看域名www.example.com下目录directory对应的映射
	nginx-cfg -t search -s www.example.com -d directory
```
