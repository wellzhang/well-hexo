---

title: Vagrantfile配置说明
categories: vagrant
tags: vagrant
---


在我们的开发目录下有一个文件Vagrantfile，里面包含有大量的配置信息，主要包括三个方面的配置，虚拟机的配置、SSH配置、Vagrant的一些基础配置。Vagrant是使用Ruby开发的，所以它的配置语法也是Ruby的，但是我们没有学过Ruby的人还是可以跟着它的注释知道怎么配置一些基本项的配置。

## box设置
```
config.vm.box = "CentOs7"
```
该名称是再使用 vagrant init 中后面跟的名字。

## hostname设置
```
config.vm.hostname = "for_work"
```
设置hostname非常重要，因为当我们有很多台虚拟服务器的时候，都是依靠hostname來做识别的。比如，我安装了php7 php56两台虚拟机，再启动时，我可以通过vagrant up php7来指定只启动哪一台。

## 虚拟机网络设置
```
config.vm.network "private_network", ip: "192.168.33.10"
#config.vm.network "public_network"
```
Vagrant有两种方式来进行网络连接，一种是host-only(主机模式)，意思是主机和虚拟机之间的网络互访，而不是虚拟机访问internet的技术，也就是只有你一個人自High，其他人访问不到你的虚拟机。另一种是Bridge(桥接模式)，该模式下的VM就像是局域网中的一台独立的主机，也就是说需要VM到你的路由器要IP，这样的话局域网里面其他机器就可以访问它了。我一般设置为host－only模式。 
当然该模式，再指定ip的时候注意不要跟主机所在网段发生冲突。

## 同步目录设置
```
config.vm.synced_folder  "/Users/helei/www", "/vagrant"
```
我们上面介绍过/vagrant目录默认就是当前的开发目录，这是在虚拟机开启的时候默认挂载同步的。我们还可以通过配置来设置额外的同步目录。

## 端口转发设置
```
config.vm.network :forwarded_port, guest: 80, host: 80
```
上面这句配置可厉害了，这一行的意思是把对host机器上8080端口的访问请求forward到虚拟机的80端口的服务上，例如你在你的虚拟机上使用nginx跑了一个php应用，那么你在host机器上的浏览器中打开http://localhost时，Vagrant就会把这个请求转发到VM里面跑在80端口的nginx服务上，因此我们可以通过这个设置来帮助我们去设定host和VM之间，或是VM和VM之间的信息交互。 
个人不建议使用该方法，经常因为两台机子端口占用的问题，导致不能正常通信。还是使用上面说的两种网络方式进行设置吧。

## 多实例配置
上面说的配置方式，均是单机模式，下面说说如何进行集群机器的部署与配置，这是vagrant让我正真激动与兴奋的地方。

看完下面，你会觉得超级简单

现在我们来建立多台VM跑起來，並且让他们之间能够相通信，假设一台是应用服务器、一台是redis服务器，那么这个结构在Vagrant中非常简单，其实和单台的配置差不多，你只需要通过config.vm.define来定义不同的角色就可以了，现在我们打开配置文件进行如下设置：
```
Vagrant.configure("2") do |config|
  config.vm.define :web do |web|
    web.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "web", "--memory", "512"]
    end
    web.vm.box = "CentOs7"
    web.vm.hostname = "web"
    web.vm.network :private_network, ip: "192.168.33.10"
  end

  config.vm.define :redis do |redis|
    redis.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "redis", "--memory", "512"]
    end
    redis.vm.box = "CentOs7"
    redis.vm.hostname = "redis"
    redis.vm.network :private_network, ip: "192.168.33.11"
  end
end
```
这里的的设置与设置单台机器非常的类似，如果还需要机器，只需要再配置文件中拷贝一下，然后重新加载一下这个配置文件就ok啦。是不是非常容易？后面我打算学hadoop的时候，就用这种方式来试试。 
现在只需要重新启动一下vagrant up机器，你就会在虚拟机中看到两台虚拟机欢快的跑起来了。 
然后这个时候，在使用vagrant ssh登录时，需要指明一下登录的是哪一台机器就ok啦。

比如，我要登录到redis中去。
```
vagrant ssh redis
```
这么简单就完成登录了。登录成功后，可以使用ping命令，检查一下机器之间是否能够互相通信。




