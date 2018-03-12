---
title: vagrant
categories: vagrant
tags: 
- vagrant  
- virtualbox
---



# VirtualBox + Vagrant


## 安装

安装VirtualBox：
网址：https://www.virtualbox.org/wiki/Downloads

安装Vagrant：
网址：http://downloads.vagrantup.com/

下载系统镜像：
网址：http://www.vagrantbox.es/

## 添加镜像
具体操作如下：
添加镜像到Vagrant：
  //镜像的存放位置为/home/box/centos65.box

  cd/home/box/

  vagrant box add centosbox centos65.box
初始化开发环境：
```
  vagrant init centosbox    #初始化

  vagrant up                #启动环境
```
SSH登陆：

  利用Xshell、Putty、SecureCRT等登录。
```
  Ip : 127.0.0.1

  Port : 2222

  Username : root

  Password : vagrant
```
常用配置
Vagrant初始化成功后，会在初始化的目录里生成一个Vagrantfile文件，可以修改该文件进行个性化的定制。

配置IP:
  config.vm.network :private_network, ip: “192.168.33.10”[去掉#]

  你可以把IP改成其他地址，只要不产生冲突就行。
配置同步目录：
  config.vm.synced_folder “../data”, “/vagrant_data” [去掉#，修改为下面]

  config.vm.synced_folder “/home/web/www”, “/data/www“

  /home/web/www：本地目录
  /data/www：    Linux服务器目录
配置虚拟内存：
  在文件结尾end字符前添加下面一段:

  config.vm.provider :virtualbox do |vb|

          vb.customize ["modifyvm", :id, "--memory", "2048"]

  end

  //温馨提示：修改配置后 记得 重启虚拟机。
打包分发:
当你配置好开发环境后，退出并关闭虚拟机。

  在终端里对开发环境进行打包：

  vagrant package

  //打包完成后会在当前目录生成一个package.box的文件，

  //将这个文件传给其他用户，

  //其他用户只要添加这个box并用其初始化自己的开发目录,

  //就能得到一个一模一样的开发环境了。
常用命令
  vagrant init #初始化

  vagrant up #启动虚拟机

  vagrant halt #关闭虚拟机

  vagrant reload #重启虚拟机

  vagrant status #查看虚拟机运行状态
