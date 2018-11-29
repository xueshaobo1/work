# mysql主从同步读写分离

## 环境背景

随着网站用户增加访问量增大，数据库读写会成为整个架构的瓶颈。所以我们一般使用数据库的主从同步将“读”和“写”分开，从而提高数据库的 读写性能。

## 主从原理

## 环境搭建

- centos 7

- mysql 5.7（mysql安装见[mysql安装](http://www.xueshaobo.top/2018/10/24/LNMP%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2/)）

```bash
sed -i 's#=enforcing#=disabled#' /etc/selinux/config
setenforce 0 
systemctl stop firewalld #关闭防火墙
```

## 主从同步

在MySQL_MASTER配置文件中添加内容

```bash
vim/etc/mysql/my.cnf

server-id = 1 #数据库标识
log_bin = master-bin #主服务器日志文件
log_bin_index = master-bin.index
#binlog_do_db = my_data #指定同步的库
#binlog_ignore_db = mysql #指定不同步的库

max_binlog_size = 500M #每个bin-log最大大小，当此大小等于500M时会自动生成一个新的日志文件。一条记录不会写在2个日志文件中，所以有时日志文件会超过此大小。
binlog_cache_size = 128K #日志缓存大小
expire_logs_day=2 #设置bin-log日志文件保存的天数，此参数mysql5.0以下版本不支持
```

登陆mysql_master服务器

```bash
grant replication slave on *.* to 'admin' @'192.168.1.%' identified by 'admin';
#创建远程登陆账户admin并授权远程登陆
flush privileges;
```

重启mysql服务

```bash
service mysql restart
```

登陆mysql_master服务器

```bash
show master status;
```



![status](https://raw.githubusercontent.com/xueshaobo1/myapp/master/mysql.PNG)

在MYSQL_SLAVE配置文件中添加内容

```bash
vim/etc/mysql/my.cnf

server-id = 2 #数据库标识
relay-log = slave-relay-bin #从服务器日志文件
read-only = on  #从库设置只读权限
relay-log-index = slave-relay-bin.index
#replicate-do-db = test #制定要同步多数据库
```

登陆mysql_slave服务器

```bash
change master to master_host='192.168.1.103',master_port=3306,master_user='admin',master_password='admin',master_log_file='master-bin.000001',master_log_pos=316;
#建立与msyql_maseter的连接
#master_host 主mysql地址
#master_port 主mysql端口
#master_user，master_password 远程用户和密码
#master_log_file File名称
#master_log_pos=2403; Position列对应数字
```

```bash
start|stop slave; #启动|关闭同步
change master to master_connect_retry=50; #设置超时时间
show slave status\G #查看同步状态
```

```bash
Slave_IO_Running:YES
Slave_IO_Running:YES
```

显示以上参数为YES表明同步没有问题

