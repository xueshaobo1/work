![timg](https://raw.githubusercontent.com/xueshaobo1/myapp/master/timg.jpg)



#                          LNMP安装部署

作者：薛少波

版本：v1.0.0

日期：2018年10月

文档编号：

## 注意事项

## 关键字定义

建议：建议的可以修改

必须：必须的必须遵循

禁止：禁止的不允许执行

## 历史编辑记录

## 背景适用范围

## 历史变更记录

| 日期           | 名称         | 版本号 |
| -------------- | ------------ | ------ |
| 2018年10月15号 | LNMP安装部署 | v1.0.0 |

## 目录

nginx1.12.2安装..............................................................................................................................................1

​	环境准备...................................................................................................................................................1

​	nginx下载.................................................................................................................................................1

​	解压安装...................................................................................................................................................1

​	启动nginx.................................................................................................................................................1

​	nginx配置文件参考..................................................................................................................................1

PHP5.6.38安装.................................................................................................................................................2

​	环境准备....................................................................................................................................................2

​	PHP下载....................................................................................................................................................2

​	解压安装....................................................................................................................................................2

​	复制配置文件.............................................................................................................................................2

​	启动PHP PHP-FPM....................................................................................................................................2

​	创建启动项.................................................................................................................................................2

mysql5.6.23安装...............................................................................................................................................3

​	环境准备.....................................................................................................................................................3

​	mysql下载..................................................................................................................................................3

​	解压安装.....................................................................................................................................................3

​	创建启动项修改启动文件...........................................................................................................................3

​	mysql初始化设置密码................................................................................................................................3

### nginx1.12.2安装

#### 环境准备

安装编译nginx所需要的环境

```bash
yum install gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

#### nginx下载

必须下载nginx官网http://nginx.org官方包，禁止使用第三方包

```bash
wget http://101.110.118.70/nginx.org/download/nginx-1.12.2.tar.gz
```

#### 解压安装

```bash
tar -zxvf nginx-1.12.1.tar.gz && cd nginx-1.12.1
./configure 
--prefix=/usr/local/nginx #指定安装位置，可以根据需求设置安装参数
--with-http_stub_status_module 
--with-http_ssl_module 
--with-http_gzip_static_module
make && make install
```

#### 启动nginx

使用nginx自带的脚本启动

```bash
/usr/local/nginx/sbin/nginx
```

脚本参数

```bash
/usr/local/nginx/sbin/nginx -h
nginx version: nginx/1.12.1
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload #启动重启
  -p prefix     : set prefix path (default: /usr/local/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file

```

#### 创建启动项

```bash
vim /etc/init.d/nginx
chmod +x /etc/init.d/nginx
```



```bash
#!/bin/bash

# chkconfig: 2345 99 20

#description: nginx-server

nginx=/usr/local/nginx/sbin/nginx

case $1 in

        start)

                netstat -anptu | grep nginx

                if [ $? -eq 0 ]

                then

                    echo "nginx service is already running"

                else

                     echo "nginx Service started successfully "

                    $nginx

                fi

         ;;

        stop)

                $nginx -s stop

                if [ $? -eq 0 ]

                then

                    echo "nginx service closed successfully"

                else

                    echo "nginx server stop fail,try again"

                fi

        ;;

        status)

                netstat -anlpt | grep nginx

                if [ $? -eq 0 ]

                then

                    echo "nginx server is running"

                else

                    echo "nginx service not started "

                fi

        ;;

        restart)

                 $nginx -s reload

                 if [ $? -eq 0 ]

                 then

                     echo "nginx service restart successfully "

                 else

                     echo "nginx server restart failed"

                 fi

        ;;

        *)

                 echo "please enter {start restart status stop}"

        ;;

esac
```

#### nginx配置文件参考

```bash
    server {
         listen 8080;  #端口
         server_name  192.168.2.231; #主机IP或域名
    
         root  html;  #网站根目录
         location / {
            index  index.php index.html index.htm; #索引文件
         }
         location ~ \.php$ {
         #root           html;
         fastcgi_pass   127.0.0.1:9000;
         fastcgi_index  index.php;
         fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
         include        fastcgi_params;
         }
    }

```



### PHP5.6.38安装

#### 环境准备

```bash
yum -y install libmcrypt-devel libxml2-devel bzip2 bzip2-devel curl curl-devel libjpeg-devel libpng-devel freetype-devel openldap openldap-devel
```

#### PHP下载

```bash
wget http://cn2.php.net/distributions/php-5.6.38.tar.gz
```

#### 解压安装

```bash
tar zxfv php-5.6.38.tar.gz && cd php-5.6.38
./configure --enable-fpm --with-mysql && make && make install #这里只安装了mysql模块和fpm模块具体参数--help
```

#### 复制配置文件

```bash
cp php.ini-development /usr/local/php/php.ini
cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
cp sapi/fpm/php-fpm /usr/local/bin
sed -i 's/cgi.fix_pathinfo=/cgi.fix_pathinfo=0/g' /usr/local/php/php.ini
sed -i 's/;pid = run/php-fpm.pid/pid = run/php-fpm.pid/g' /usr/local/etc/php-fpm.conf
#如果文件不存在，则阻止 Nginx 将请求发送到后端的 PHP-FPM 模块， 以避免遭受恶意脚本注入的攻击。
#将 php.ini 文件中的配置项 cgi.fix_pathinfo 设置为 0 
```

#### 启动PHP PHP-FPM

```bash
/usr/local/bin/php-fpm
```

#### 创建启动项

进入php安装包目录

```bash
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chomd +x /etc/init.d/php-fpm
service php-fpm start|stop|restart
```

### mysql5.6.23

#### 环境准备

mysql安装可以自己编译也可以下载对应平台架构编译好的源码进行二进制安装，这里我们下载二进制安装包

#### mysql下载

```bash
wget --no-check-certificate https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.23-el7-x86_64.tar.gz
```

#### 解压安装

```bash
tar zxvf mysql-5.7.23.tar.gz && mv mysql-5.7.23 /opt/mysql
```

创建启动mysql用户和组并为安装文件修改权限

```bash
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
mkdir -p /opt/data
chown -R mysql:mysql /opt/mysql
chown -R mysql:mysql /opt/data
```

#### 创建启动项修改启动文件

```bash
cp mysql/support-files/mysql.server /etc/init.d/mysql
```

打开/etc/init.d/mysql将

```bash
basedir=/opt/mysql #mysql主目录文件
datadir=/opt/data #mysql数据文件
```

修改mysql配置文件

打开/etc/my.cnf

```bash
[client]
port = 3306
default-character-set=utf8
#socket=/opt/run/mysql.sock

[mysqld]
#socket=/opt/run/mysql.sock
basedir = /opt/mysql
datadir = /opt/data
port = 3306
character-set-server=utf8
default_storage_engine = InnoDB


sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
#此处配置文件仅供参考，实际环境需要根据自己需求配置
```

#### mysql初始化

建立软连接连接mysql

```bash
ln -s /opt/mysql/bin/mysql /usr/bin #根据自己路径来
```

初始化mysql

```bash
cd /opt/mysql/bin
./mysqld --initialize --user=mysql --basedir=/opt/mysql --datadir=/opt/data 
#--defaults-file=/etc/mysql/my.cnf 指定配置文件 
#--pid-file=/var/run/mysqld/mysqld.pid 指定进程
#--socket=/var/run/mysqld/mysqld.sock 指定mysql文件位置
#user 启动mysql的用户
#basedir mysql主目录
#datadir mysql数据存储路径
```

初始化过记住mysql随机生成的密码，启动mysql

```bash
service mysql start
```

登陆mysql修改初始化密码

```bash
mysql -u root -p
set password = password('新密码')
quit
```



















