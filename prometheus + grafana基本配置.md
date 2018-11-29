# prometheus + grafana基本配置

## 环境介绍

## 环境准备

```bash
yum install ntpdate
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ntpdate -u cn.pool.ntp.org
```

## prometheus安装

- prometheus官网 https://prometheus.io/download/

下载prometheus

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.4.3/prometheus-2.4.3.linux-amd64.tar.gz
```

解压运行prometheus

```bash
tar zxvf prometheus-2.4.3.linux-amd64.tar.gz
cd prometheus-2.4.3.linux-amd64
./prometheus
```

直接访问http://127.0.0.1:9090

创建启动方式

```bash
vim /usr/lib/systemd/system/prometheus.service
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
Restart=on-failure
KillMode=control-group
WorkingDirectory=/usr/local/prometheus/    #指定工作目录
ExecStart=/usr/local/prometheus/prometheus   #指定启动方式
config.file=/usr/local/prometheus/prometheus.yml #指定配置文件
```

## node_exporter安装

exporter监控插件有很多比如mysql_exporter、memcached_exporter、statsd_exporter等这里我们选择基本的node_exporter示范

下载node_exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz
```

解压运行node_exporter

```bash
tar zxvf node_exporter-0.16.0.linux-amd64.tar.gz
cd node_exporter-0.16.0.linux-amd64
./node_exporter
```

创建启动方式

```bash
vim /usr/lib/systemd/system/node.service
[Unit]
Description=node Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
Restart=on-failure
KillMode=control-group
WorkingDirectory=/usr/local/node_exporter/    #指定工作目录
ExecStart=/usr/local/prometheus/node_exporter  #指定启动方式
```

直接访问http://127.0.0.1:9100

加入prometheus监控

```go
  - job_name: 'node_exporter'
    static_configs:
    - targets: ['localhost:9100']
```



### 监控示例

```bash
示例一
(1- (sum(rate(node_cpu_seconds_total{mode="idle"}[1m])) / sum(rate(node_cpu_seconds_total[1m]))))*100
采集cpu使用率
(sum(rate(node_cpu_seconds_total{mode="user"}[1m]))  / sum(rate(node_cpu_seconds_total[1m])))*100
cpu用户态使用率
(sum(rate(node_cpu_seconds_total{mode="system"}[1m]))  / sum(rate(node_cpu_seconds_total[1m])))*100
cpu内核太使用率
(node_memory_Buffers_bytes+node_memory_Cached_bytes+node_memory_MemFree_bytes)/node_memory_MemTotal_bytes
内存空闲的百分比
(rate(node_disk_io_time_seconds_total[1m])/60)*100
硬盘IO使用
rate(node_network_receive_bytes_total[1m])/1024
网络每秒传输/Kb
示例二
cpu使用率
100 - (avg by (instance) (irate(node_cpu{instance="xxx", mode="idle"}[5m])) * 100)
cpu各mode使用率
avg by (instance, mode) (irate(node_cpu{instance="xxx"}[5m])) * 100
机器平均负载
node_load1{instance="xxx"} // 1分钟负载
node_load5{instance="xxx"} // 5分钟负载
node_load15{instance="xxx"} // 15分钟负载
内存使用率
100 - ((node_memory_MemFree{instance="xxx"}+node_memory_Cached{instance="xxx"}+node_memory_Buffers{instance="xxx"})/node_memory_MemTotal) * 100
磁盘使用率
100 - node_filesystem_free{instance="xxx",fstype!~"rootfs|selinuxfs|autofs|rpc_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse.*"} / node_filesystem_size{instance="xxx",fstype!~"rootfs|selinuxfs|autofs|rpc_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse.*"} * 100
网络IO
// 上行带宽
sum by (instance) (irate(node_network_receive_bytes{instance="xxx",device!~"bond.*?|lo"}[5m])/128)
// 下行带宽
sum by (instance) (irate(node_network_transmit_bytes{instance="xxx",device!~"bond.*?|lo"}[5m])/128)
网卡出/入包
// 入包量
sum by (instance) (rate(node_network_receive_bytes{instance="xxx",device!="lo"}[5m]))
// 出包量
sum by (instance) (rate(node_network_transmit_bytes{instance="xxx",device!="lo"}[5m]))
```



## pushgateway安装

下载pushgateway

```bash
wget https://github.com/prometheus/pushgateway/releases/download/v0.6.0/pushgateway-0.6.0.linux-amd64.tar.gz
```

解压运行pushgateway

```bash
tar zxvf pushgateway-0.6.0.linux-amd64.tar.gz
cd pushgateway-0.6.0.linux-amd64
./pushgateway
```

创建启动方式

```bash
vim /usr/lib/systemd/system/pushgateway.service
[Unit]
Description=pushgateway Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
Type=simple
Restart=on-failure
KillMode=control-group
WorkingDirectory=/usr/local/pushgateway/
ExecStart=/usr/local/pushgateway/pushgateway


[Install]
WantedBy=multi-user.target
```

直接访问http://127.0.0.1:9091

加入prometheus监控

```go
  - job_name: 'pushgateway'
    static_configs:
    - targets: ['localhost:9091']
```



## grafana安装

下载grafana

```bash
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.3.2-1.x86_64.rpm 
```

安装运行grafana

```bash
yum localinstall grafana-5.3.2-1.x86_64.rpm 
systemctl start grafana-server
```

直接访问http://127.0.0.1:3000

## 自定业务监控脚本

### 实现原理

使用prometheus隔一段时间会从pushgateway提取数据我们只要将数据实时推送到pushgateway上就可以将数据纳入监控脚本

### 监控示例

- **提取出nginx一个小时内404错误的数量**

```bash
#!/bin/bash
instance_name=`hostname -f |cut -d'.' -f1`
if [ $instance_name == "localhost"  ];then 
echo "Must FQDN hostname"
exit 1 
fi

timed=`date +%d/%b/%Y:%H`
nginx=/usr/local/nginx/logs/access.log

label="log_404" 

log_404=`cat $nginx | grep $timed | grep 404 | wc -l`

echo "$label : $log_404"

echo "$label $log_404" | curl --data-binary @- http://192.168.2.178:9091/metrics/job/pushgateway/instance_hostname/$instance_name
```

- **网站访问时间**

```bash
#!/bin/bash
i=1
while i=1
do
instance_name=`hostname -f |cut -d'.' -f1`
if [ $instance_name == "localhost"  ];then
echo "Must FQDN hostname"
exit 1
fi

label="time_out"

time_out=`curl -o /dev/null -s -w "%{time_total}\n" "http://www.xueshaobo.top"`

echo "$label : $time_out"

echo "$label $time_out" | curl --data-binary @- http://192.168.2.178:9091/metrics/job/pushgateway/instance_hostname/$instance_name
```

## python编写客户端

prometheus提供了多种语言编写监控客户端[监控示例](https://prometheus.io/docs/instrumenting/writing_clientlibs/)

环境准备

```bash
pip install prometheus_client #安装prometheus客户端模块
yum install gcc python-devel #安装编译模块
pip install psutil #安装监控模块（可选择其他模块）
```

- 简单示例

```python
# !/usr/bin/python
# -*- coding:utf-8 -*- 
from prometheus_client import Gauge 

g = Gauge('my_inprogress_requests', 'Description of gauge') 

g.set(value) #value的值为正数或浮点数
```



- 监控cpu user使用时间比

```python
# !/usr/bin/python
# -*- coding:utf-8 -*- 
from prometheus_client import start_http_server, Summary,Gauge
import psutil

g = Gauge('cpu', 'cpu-time',['hostname'])
#第一个数为metric名称，第二个参数为说明，第三个数为标签（标签为列表可以为多个）
start_http_server(8000)

while True:

    g.labels(hostname='web-server').set(psutil.cpu_times().user) #设置标签的值 在设置metric的值
```

直接访问http://127.0.0.1:8000