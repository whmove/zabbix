#zabbix 安装篇
关于zabbix相关介绍就不多说，本文档主要面对zabbix新手，力求简洁明了，容易上手。

##1，环境介绍
本文档以`CentOS 6.7 x86_64`、`zabbix 2.2`、`httpd 2.2`、`php 5.3`为基础

##2，配置zabbix repo
zabbix官方自2.0开始，已提供了rpm包，http://repo.zabbix.com/zabbix/2.2/rhel/6/x86_64/

    rpm -ivh http://repo.zabbix.com/zabbix/2.2/rhel/6/x86_64/zabbix-release-2.2-1.el6.noarch.rpm
    ls /etc/yum.repos.d/zabbix.repo
  
如果不出错误，应该可以看到zabbix的repo配置，如果没有此文件，请检查网络和系统环境。

##3，安装zabbix运行所需要的软件包
安装mysql数据库

    yum install -y mysql-server
    
安装apache及php

    yum install -y httpd php

安装zabbix，因为某些原因，zabbix下载会很慢(遇到无法成功下载的问题，请修改repo文件 ，指向阿里云zabbix的镜像)

    yum install -y zabbix zabbix-agent zabbix-server zabbix-web zabbix-web-mysql
    
**阿里云 zabbix repo**
```
[zabbix]
name=Zabbix Official Repository - $basearch
#baseurl=http://repo.zabbix.com/zabbix/2.2/rhel/6/$basearch/
baseurl=http://mirrors.aliyun.com/zabbix/zabbix/2.2/rhel/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch 
#baseurl=http://repo.zabbix.com/non-supported/rhel/6/$basearch/
baseurl=http://mirrors.aliyun.com/zabbix/non-supported/rhel/6/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1
```








