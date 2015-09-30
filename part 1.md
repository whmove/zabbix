#zabbix 安装篇
本文档主要面对zabbix新手，力求简洁明了，不涉及性能及安全部分，并假定用户已禁用selinux和iptables防火墙。

##1，环境介绍
文档以`CentOS 6.7 x86_64`、`zabbix 2.2`、`httpd 2.2`、`php 5.3`、`mysql 5.1`为基础，所有软件都采用rpm包安装方式，以降低门槛。

##2，配置zabbix repo
zabbix官方已提供了rpm包，我们只需要安装官方提供的repo仓库即可。
http://repo.zabbix.com/zabbix/2.2/rhel/6/x86_64/
```
rpm -ivh http://repo.zabbix.com/zabbix/2.2/rhel/6/x86_64/zabbix-release-2.2-1.el6.noarch.rpm
ls /etc/yum.repos.d/zabbix.repo
```
如果不出错误，应该可以看到zabbix的repo配置，如果没有此文件，请检查网络和系统环境。

##3，安装zabbix运行所需要的软件包
安装mysql数据库
```
yum install -y mysql-server
```
安装apache及php
```
yum install -y httpd php
```
安装zabbix，因为某些原因，zabbix下载会很慢(遇到无法成功下载的问题，请修改zabbix的repo文件，指向阿里云的zabbix镜像)
```
yum install -y zabbix zabbix-agent zabbix-server zabbix-web zabbix-web-mysql
```
**阿里云zabbix repo文件内容**
`/etc/yum.repos.d/zabbix.repo`
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

##4，配置zabbix数据库
启动mysql服务，系统会自动帮我们完成数据库的初始化
```
/etc/init.d/mysql start
```
为root用户设置密码
```
mysqladmin -u root password 'pwd123'    # 设置root用户密码为pwd123
```
创建zabbix数据库(正式生产环境中使用如下命令时，请去掉命令行中的密码，以交互方式完成登录)
```
# 创建zabix数据库，设置字符集为utf8
mysql -uroot -ppwd123 -e "create database zabbix character set utf8;"
# 为zabbix用户授权，并设置密码为`zabbixpwd`
mysql -uroot -ppwd123 -e "grant all on zabbix.* to zabbix@localhost identified by 'zabbixpwd';"
# 检查前面工作
mysql -uroot -ppwd123 -e "show databases;"
mysql -uroot -ppwd123 -e "select User,Host from mysql.user where User='zabbix'"
```
导入zabbix的表结构及基础数据(正式生产环境中使用如下命令时，请去掉命令行中的密码，以交互方式完成登录)
```
mysql -uzabbix -pzabbixpwd zabbix < /usr/share/doc/zabbix-server-mysql-2.2.10/create/schema.sql
mysql -uzabbix -pzabbixpwd zabbix < /usr/share/doc/zabbix-server-mysql-2.2.10/create/images.sql
mysql -uzabbix -pzabbixpwd zabbix < /usr/share/doc/zabbix-server-mysql-2.2.10/create/data.sql
```

##5，启动zabbix server服务
修改`vi /etc/zabbix/zabbix_server.conf`配置文件，设置zabbix服务连接数据库所使用的密码
```
DBPassword=zabbixpwd
```
启动zabbix服务
```
/etc/init.d/zabbix-sever start
```
查看zabbix服务端口及启动日志
```
netstat -tunlp | grep :10051
cat /var/log/zabbix/zabbix_server.log
```
##6，配置前端WEB
zabbix前端使用PHP编写，运行时对php环境配置有要求。
我们安装的php 5.3，大部分要求均可满足，如果未能满足在安装时会有提示，按要求修改即可。

开启web服务(如果开启了防火墙的，记得开放相应的端口)
```
/etc/init.d/httpd start
netstat -tunlp | grep :80
```

> Welcome

在游览器中打开`http://hostip/zabbix/`，会看到zabbix安装界面

> Check of pre-requisites

在zabbix web环境检查界面，会看到有一个`php time zone`的条件未满足，
修改`/etc/php.ini`文件中相应设置后，重启web服务，然后再点击`Retry`，请确保检测通过
```
date.timezone = Asia/Shanghai
/etc/init.d/http restart
```

> Configure DB connection

设置数据库连接所使用的账号密码：
User: zabbix
Password: zabbixpwd
然后执行`Test connection`，请确保检测通过

> Zabbix server details

指定zabbix server所在的主机和端口，此文档中都是在同一台主机上，所以直接`Next`

> Pre-Installation summary

再次检查所有配置，如果有问题`Previous`修改，确认无误后`Next`

> Install

可看到web相关配置会保存至文件`etc/zabbix/web/zabbix.conf.php`，以后需要调整，直接修改这个文件即可。

安装完成后，可WEB登录zabbix，初始管理员：
```
Username:Admin
Password:zabbix
```

##7，添加zabbix server自身的监控
```
/etc/init.d/zabbix-agent start
netstat -tunlp | grep :10050
```
前端WEB中，`Configuration`--`Hosts`--`Not monitored`，开启对zabbix server自身的监控。


##8，清场
```
chkconfig mysqld on
chkconfig httpd on
chkconfig zabbix-server on
chkconfig zabbix-agent on
```



##Q&A
1，前端WEB界面看到，`Zabbix server is running`显示为`No`，但zabbix server服务确实是启动的，并且日志中没有任何错误？
>请检查selinux状态，确保是禁用状态，或者为其配置权限
```
[root@zabbix zabbix]# sestatus 
SELinux status:                 disabled
```

