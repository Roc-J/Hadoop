# Ubuntu开启SSH登录 #
## 前言 ##
在新安装的ubuntu系统后，默认是不支持ssh登录的，刚刚重新在VM中安装的ubuntu16.04,不能使用xshell进行连接，所以，需要按照下面步骤进行安装。
## 安装ssh ##
	这里使用的是openssh系列工具
1. 更新下系统工具和依赖，执行`sudo apt-get update`
![](http://i.imgur.com/ydGiK9y.png) 

2. 执行安装命令：`sudo apt-get install openssh-server openssh-client`,等待安装完成
3. ![](http://i.imgur.com/xniG6VJ.png)

## 启动ssh ##
1. 先查看ssh是否在运行，执行命令`sudo ps -e|grep ssh`，如果没有任何显示，是没有运行ssh服务的，反之，出现sshd字样，表示ssh服务已运行。
![](http://i.imgur.com/gMTZSDV.png)

2. 如果ssh没有运行，使用命令`sudo service ssh start`启动。