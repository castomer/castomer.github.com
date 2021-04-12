---
categories:
  - tools
  - vagrant
tags:
  - vagrant
  - tools
title: vagrant入门
url: /2013/12/29/vagrant-basic/
date: 2013-12-29
author: "Dongfang Qu"
---


vagrant是一个基于virtualbox的虚拟机创建、配置、销毁和管理的工具，可以帮助开发者迅速的搭建开发和测试环境。

## 下载

建议从[官方网站](http://www.vagrantup.com/downloads.html)直接下载最新版本。由于我使用的是debian sid amd64的操作系统，所以我的下载方式如下：

  `wget -c https://dl.bintray.com/mitchellh/vagrant/vagrant_1.4.1_x86_64.deb`

## 安装

    sudo dpkg -i vagrant_1.4.1_x86_64.deb

## 概念

* box

vagrant将虚拟机系统的模板定义为box，vagrant根据box系统来实例化具体的虚拟机实例。

* provision

vagrant根据box实例化一台具体的虚拟机后执行的一些用户自定义软件包安装、配置动作。

## 命令

- vagrant init

在工作目录中初始化vagrant配置文件Vagrantfile.

    vagrant init # 初始化默认的配置文件，需要手动编辑Vagrantfile文件添加虚拟机模板box
    vagrant init precise32 http://files.vagrantup.com/precise32.box # 初始化vagrant工程的同时引入模板box precise32

- vagrant box

设置vagrant可使用的模板box名称和模板文件的地址。

    vagrant box add box_name box_url # box_url可以是本地目录
    vagrant box remove box_name # 删除模板box定义
    vagrant box list # 获得本机可用box列表

- vagrant up

实例化虚拟机并开机，如果已经实例化过，则直接开机。

    vagrant up # 全部开机
    vagrant up vm_01 vm_02 # vm_01 vm_02 开机

- vagrant destroy

销毁已经实例化的虚拟机。

    vagant destroy # 交互式全部销毁
    vagant destroy -f # 非交互式
    vagrant destroy -f vm_01 # 指定销毁机器列表
    vagant destroy -h

- vagrant suspend

保存当前虚拟机的状态，并使其暂停。

    vagrant suspend # 暂定机器状态
    vagrant suspend vm_01 vm_02 # 暂停指定列表的虚拟机
    vagrant suspend -h

- vagrant ssh

ssh登录虚拟机。

    vgrant ssh vm_01

- vagrant status

显示当前虚拟机状态。

    vagrant status # 所有虚拟机的状态
    vagrant status vm_01 vm_02
    vagrant status -h

- vagrant provision

    vagrant provision
    vagrant provision vm_01 vm_02
    vagrant provision -h

- vagrant resume

恢复之前suspend的机器。

    vagrant resume
    vagrant resume vm_01 vm_02
    vagrant resume -h

- vagrant halt

关闭虚拟机。

    vagrant halt
    vagrant halt vm_01 vm_02
    vagrant halt -h

- vagrant package

基于一个virtualbox虚拟机当前的配置打包一个可重用的box。

    vagrant package --base vm_01 --output /some_path/new.box
    vagrant package -h

- vagrant plugin

vagrant支持第三方插件，可通过以下命令进行安装和卸载。

    vagrant plugin install/uninstall some_plugin
    vagrant plugin -h

- vagrant ssh-config

自定义ssh配置项，一般用不到，详细信息请参考帮助。

    vagrant ssh-config -h

- vagrant reload

相当于`vagrant halt && vagrant up`

- vagrant help

打印帮助信息，任何一个命令不懂的时候，后面都可以直接跟-h参数获得帮助。

## 配置

    # vagrant api的版本，一般不需要关注，１和２的配置语法不太一样。
    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

      # 配置虚拟机使用的模板box名称
      # aqueducts需要使用：vagrant box add aqueducts box_url导入到本机
      config.vm.box = "aqueducts"

    # 定义虚拟机的名称
      config.vm.define :vm_01 do |this|
        this.vm.provision "shell", inline: "echo '192.168.2.11 vm_01 localhost' > /etc/hosts" # inline provision,简单的shell语句
        this.vm.provision "shell", path: "provision/provision_ruby.sh" # 安装ruby的shell 脚本
        this.vm.provision "shell", inline: "service iptables stop && chkconfig iptables off" # 默认会使用root权限执行provision动作
        this.vm.network :private_network, ip: "192.168.2.11" # 定义网络类型和ip, 每个vagrant虚拟机会有两个网卡，一个用来和宿主机通信(ssh),另一个用于虚拟机之间通信。
        this.vm.hostname = "vm_01"
      end

      config.vm.define :vm_01 do |this|
        this.vm.box = "other" # 不使用默认的aqueducts box
        this.vm.provision "shell", inline: "echo '192.168.2.12 vm_02 localhost' > /etc/hosts"
        this.vm.provision :shell, path: "provision/provision_redis.sh", privileged: false # 安装redis的shell脚本，使用非root账号安装
        this.vm.provision "shell", inline: "service iptables stop && chkconfig iptables off"
        this.vm.network :private_network, ip: "192.168.2.12"
        this.vm.network "forwarded_port", guest: 6379, host: 6379 # 设置端口映射
        this.vm.hostname = "vm_01"
      end

      # 设置共享目录，虚拟机挂载路径
      config.vm.synced_folder "./", "/home/vagrant/vagrant_data"

       # virtualbox 全局配置
       config.vm.provider :virtualbox do |vb|
         # vb.gui = true # 是否调用virtualbox gui
         vb.customize ["modifyvm", :id, "--memory", "256"] # 内存
         vb.customize ["modifyvm", :id, "--cpus", 1] # cpu个数
       end

    end