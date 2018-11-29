# gitlab + jenkins持续集成

## gitlab安装配置

### 环境要求

gitlab服务器官方要求2核4g（1核1g也可以跑起来，生产环境建议使用官方推荐配置）

环境依赖

```bash
yum -y install policycoreutils 
systemctl enable postfix && systemctl start postfix-server openssh-clients postfix
```

### gitlab安装

```bash
vim /etc/yum.repos.d/gitlab-ce.repo
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
yum makecache
yum install gitlab-ce
```

### gitlab操作

```bash
sudo gitlab-ctl start    # 启动所有 gitlab 组件；
sudo gitlab-ctl stop        # 停止所有 gitlab 组件；
sudo gitlab-ctl restart        # 重启所有 gitlab 组件；
sudo gitlab-ctl status        # 查看服务状态；
sudo gitlab-ctl reconfigure        # 启动服务；
sudo vim /etc/gitlab/gitlab.rb        # 修改默认的配置文件；
gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab；
sudo gitlab-ctl tail        # 查看日志；
```

### 本地git配置

- linux

```bash
yum install git -y
```

- windown

安装git客户端[下载地址](https://git-scm.com/downloads)

### git基本命令

```bash
ssh-keygen #生成ssh-key
git config --global user.name "username" #配置用户名
git config --global user.email "useremail" #配置用户邮箱

git clone url #克隆远程仓库
git add #提交文件
git commit -m "" #提交本地仓库
git push origin master #上传到gitlab服务器

git init #初始化本地仓库
git pull #拉去远程分支 

git branch name #创建那么分支
git checkout -b name #创建name分支并切换
git merge name #合并分支
git branch -d name #删除分支

```

### gitlab权限问题

![2018-11-29_110047](C:\Users\Admin\Desktop\2018-11-29_110047.png)

## jenkins安装配置

### jenkins安装

jenkins环境要求安装java1.8

```bash
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel -y
```

YUM安装

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins -y
```

或者直接清华镜像站下载

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat/jenkins-2.96-1.1.noarch.rpm
rpm -ivh jenkins-2.96-1.1.noarch.rpm
systemctl start jenkins
```

### jenkins插件安装

[jenkins插件下载地址](https://plugins.jenkins.io/)

- 常用插件介绍

1. ssh
2. git plugin

