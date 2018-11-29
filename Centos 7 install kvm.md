

# Centos 7安装KVM

## 环境准备

```bash
sed -i 's#=enforcing#=disabled#' /etc/selinux/config
setenforce 0 
systemctl stop firewalld #关闭防火墙
```

```bash
yum -y install qemu-kvm python-virtinst libvirt libvirt-python virt-manager libguestfs-tools bridge-utils virt-install #安装需要的环境
```

- `qemu-kvm` 主要的KVM程序包

- `python-virtinst` 创建虚拟机所需要的命令行工具和程序库

- `virt-manager` GUI虚拟机管理工具

- `virt-top` 虚拟机统计命令

- `virt-viewer` GUI连接程序，连接到已配置好的虚拟机

- `libvirt` C语言工具包，提供libvirt服务

- `libvirt-client` 为虚拟客户机提供的C语言工具包

- `virt-install` 基于libvirt服务的虚拟机创建命令

- `bridge-utils` 创建和管理桥接设备的工具

  ```bash
  systemctl start|stop|restart|enable|status libvirtd  #启动|停止|重启|开机启动|查看状态
  ```

  ## 安装kvm

  ```bash
  mkdir -p /tmp/kvm  #存放镜像位置
  mkdir -p /home/kvm #安装目录位置
  virt-install  \
  --virt-type=kvm  \ 
  --name=centos7 --vcpus=1 --memory=1024 \ #名称、cpu个数、内存大小
  --location=/tmp/kvm/CentOS-7-x86_64-Minimal-1511.iso \  #安装镜像位置
  --disk path=/home/kvm/centos7.qcow2,size=10,format=qcow2 \  #磁盘位置、磁盘大小
  --network bridge=virbr0 \ #桥接网卡
  --graphics none \ #是否开启图像界面
  --extra-args='console=ttyS0' \ #定义额外参数
  --force \
  
  virt-install --virt-type=kvm --name=centos7 --vcpus=1 --memory=1024 --location=/tmp/kvm/CentOS-7-x86_64-Minimal-1511.iso --disk path=/home/kvm/centos7.qcow2,size=10,format=qcow2 --network bridge=virbr0 --graphics none --extra-args='console=ttyS0' --force
  ```

  进入命令行安装界面

  ```bash
  Installation
  
   1) [x] Language settings                 2) [!] Timezone settings
          (English (United States))                (Timezone is not set.)
   3) [!] Installation source               4) [!] Software selection
          (Processing...)                          (Processing...)
   5) [!] Installation Destination          6) [x] Kdump
          (No disks selected)                      (Kdump is enabled)
   7) [ ] Network configuration             8) [!] Root password
          (Not connected)                          (Password is not set.)
   9) [!] User creation
          (No user will be created)
    Please make your choice from above ['q' to quit | 'b' to begin installation |
    'r' to refresh]:
  ```

- Language settings   #语言设置

- Timezone settings #时区设置

- Installation source #安装方式

- Software selection #自定义安装软件

- Installation Destination #指定安装的磁盘

- Network configuration #网路设置

- Root password #设置root密码

  **（记得调整号安装方式要刷新要不然无法显示无法安装）**

设置完成按b就会自定安装安装完成重启进入系统

## virsh 常用命令

```bash
virsh console "name"     #启动相对"name"命令行
virsh start "name"     # 虚拟机开启（启动）：
virsh reboot "name"    # 虚拟机重新启动
virsh shutdown "name"  # 虚拟机关机
virsh destroy "name"   # 强制关机（强制断电）
virsh suspend "name"   # 暂停（挂起）KVM 虚拟机
virsh resume "name"    # 恢复被挂起的 KVM 虚拟机
virsh undefine "name"  # 该方法只删除配置文件，磁盘文件未删除
virsh autostart "name" # 随物理机启动而启动（开机启动）
virsh autostart --disable "name" # 取消标记为自动开始（取消开机启动）
virsh list "name"|all  #查看kvm虚拟机列表
```

## 桥接网卡

默认使用是NAT模式虚拟机可以访问外网，但是外网主机无法访问虚拟机。可以根据环境要求配置供外网访问的虚拟主机。这里需要桥接网卡。

![NAT](https://raw.githubusercontent.com/xueshaobo1/myapp/master/nat.png)

先单独创建虚拟网卡然后进行桥接

```bash
cd /etc/sysconfig/network-scripts/
cp ifcfg-ens33 ifcfg-br0 #新建虚拟网卡，配置复制前面的网卡

vim ifcfg-ens33
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="88436a77-36c0-4a7c-81ad-e10f6d3edda5"
DEVICE="ens33"
ONBOOT="yes"
BRIDGE=br0 #添加桥接的网卡

vim ifcfg-br0
TYPE="Bridge" #改为桥接模式
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
NAME="br0"
#UUID="88436a77-36c0-4a7c-81ad-e10f6d3edda5"
DEVICE="br0"
ONBOOT="yes"
```

重启网络

```bash
systemctl restart network
```

![IFCONFIG](https://raw.githubusercontent.com/xueshaobo1/myapp/master/net.PNG)

新建虚拟主机

```bash
virt-install  \
--virt-type=kvm  \ 
--name=centos7 --vcpus=1 --memory=1024 \ 
--location=/tmp/kvm/CentOS-7-x86_64-Minimal-1511.iso \  
--disk path=/home/kvm/centos7.qcow2,size=10,format=qcow2 \  
--network bridge=br0 \ #网卡桥接指定为br0
--graphics none \ 
--extra-args='console=ttyS0' \ 
--force \
```

安装完成之后启动虚拟机网络

![IP](https://raw.githubusercontent.com/xueshaobo1/myapp/master/IP.PNG)

外网主机可以直接ping通192.168.2.169