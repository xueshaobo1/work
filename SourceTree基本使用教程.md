#              SourceTree基本使用教程

## 安装 SourceTree

点开压缩包点“next”一路确认完成后打开



![1](https://raw.githubusercontent.com/xueshaobo1/myapp/master/1.PNG)

## 配置SourceTree

点击命令行执行ssh-keygen

进入C盘，“用户”-->“主机名称”目录下找到.ssh文件，打开之后找到id_rsa文件（客户端密钥）

打开SourceTree，“工具”-->“选项”



![2](https://raw.githubusercontent.com/xueshaobo1/myapp/master/TIM%E6%88%AA%E5%9B%BE20181011115835.png)

客户端配置完成

## gitlab端配置

打开内网[gitlab服务器](https://192.168.2.237)注册登陆

添加ssh密钥，复制id_rsa.pub文件内容到文本框，然后“add key”

![TIM截图20181011122431](C:\Users\Admin\Desktop\git\TIM截图20181011122431.png)



![TIM截图20181011122458](C:\Users\Admin\Desktop\git\TIM截图20181011122458.png)



![TIM截图20181011122527](C:\Users\Admin\Desktop\git\TIM截图20181011122527.png)

回到gitlab首页创建新项目![TIM截图20181011123203](C:\Users\Admin\Desktop\git\TIM截图20181011123203.png)

将ssh地址复制下来

![TIM截图20181011123334](C:\Users\Admin\Desktop\git\TIM截图20181011123334.png)

## 配置本地仓库

![TIM截图20181011123609](C:\Users\Admin\Desktop\git\TIM截图20181011123609.png)

打开对应本地仓库的目录发现已经克隆下来了

![TIM截图20181011123851](C:\Users\Admin\Desktop\git\TIM截图20181011123851.png)

在里面新建文件![TIM截图20181011124032](C:\Users\Admin\Desktop\git\TIM截图20181011124032.png)

发现SourceTree里面出现了自己创建的文件点击即可提交到暂存区

![TIM截图20181011124059](C:\Users\Admin\Desktop\git\TIM截图20181011124059.png)

点击提交按钮即可提交到本地仓库

![TIM截图20181011124311](C:\Users\Admin\Desktop\git\TIM截图20181011124311.png)

再点击推送到远程仓库

![TIM截图20181011124516](C:\Users\Admin\Desktop\git\TIM截图20181011124516.png)

再次打开gitlab远程仓库发现有了我们的推送文件

![TIM截图20181011124632](C:\Users\Admin\Desktop\git\TIM截图20181011124632.png)

