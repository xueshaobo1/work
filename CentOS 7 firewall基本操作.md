# CentOS 7 firewall基本操作

## 环境背景

- centos 7

centos 7防火墙由以前的iptable改为现在的firewall与以前配置有所不同。firewall可以直接代替iptables管理服务器的网络活动，能直接作用于内核的netfilter代码。

## 启动和关闭

```bash
systemctl start|stop|restart|enable firewalld #启动|关闭|重启|开机启动
```

## 查看运行状态

```bash
firewall-cmd --state #运行状态
firewall-cmd --reload #重新加载防火墙
firewall-cmd --list-all #查看防火墙规则
```

## 规则设置

### 添加删除服务

```bash
firewall-cmd --permanent --zone=public --add-service=http #添加http服务
firewall-cmd --permanent --zone=public --remove-service=http #删除http服务
```

### 添加删除端口

```bash
firewall-cmd --zone=public --list-ports #查看开放的端口
firewall-cmd --permanent -zone=public --add-port=7777/tcp #允许开放7777 tcp端口
firewall-cmd --permanent -zone=public --add-port=7777-8000/tcp #允许开放7777-8000 tcp端口
firewall-cmd --permanent --zone=public --remove-port=7777/tcp #删除7777 tcp开放端口
```

### 添加删除规则

```bash
firewall-cmd --zone=public --list-rich-rules #查看所有规则
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.1.1' reject" #封锁192.168.1.1
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.1.0/24' reject" #封锁192.168.1.0段所有ip
firewall-cmd --permanent --remove-rich-rule="rule family='ipv4' source address='192.168.1.1' reject" #删除此规则
```

