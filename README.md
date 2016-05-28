# Centos 7.0 安装Zabbix3.0服务端    
  

##1.1 基础环境准备 
>  
<pre>  
[root@zabbix-server ~]# cat /etc/redhat-release   
CentOS Linux release 7.2.1511 (Core) 
[root@zabbix-server ~]# uname -a
Linux zabbix-server 3.10.0-327.18.2.el7.x86_64 #1 SMP Thu May 12 11:03:55 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
[root@zabbix-server ~]# 
[root@zabbix-server ~]# getenforce 
Disabled
[root@zabbix-server ~]# systemctl stop firewalld
</pre>  
##1.2 安装yum源  
<pre>
[root@zabbix-server ~]# rpm -ivh http://mirrors.aliyun.com/zabbix/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
Retrieving http://mirrors.aliyun.com/zabbix/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
warning: /var/tmp/rpm-tmp.mwua8Y: Header V4 DSA/SHA1 Signature, key ID 79ea5ed4: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:zabbix-release-3.0-1.el7         ################################# [100%]
</pre>  
##1.3 安装zabbix相关的软件包  
<pre>
[root@zabbix-server ~]# yum install zabbix-server zabbix-agent zabbix-get zabbix-web zabbix-server-mysql zabbix-web-mysql mariadb-server mariadb -y
</pre>
**备注：Centos7的数据库默认是mariadb,不是MySQL**  
##1.4 修改PHP时区  
<pre>
[root@zabbix-server /]# sed -i 's@# php_value date.timezone Europe/Riga@php_value date.timezone Asia/Shanghai@g' /etc/httpd/conf.d/zabbix.conf
</pre>  
## 1.5 数据库配置
<pre>  
[root@zabbix-server /]# vim /etc/my.cnf 
[mysqld]
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
</pre>  
## 1.6 启动数据库，创建数据库  
<pre>
[root@zabbix-server /]# systemctl start mariadb  
[root@zabbix-server /]# mysql
MariaDB [(none)]> create database zabbix;
MariaDB [(none)]> grant all on zabbix.* to 'zabbix'@'localhost' identified by '123456';
</pre>  
## 1.7 导入Zabbix数据  
<pre>
[root@zabbix-server /]# cd /usr/share/doc/zabbix-server-mysql-3.0.3/
[root@zabbix-server zabbix-server-mysql-3.0.3]# zcat create.sql.gz |mysql -uzabbix -p123456 zabbix
</pre>
## 1.8 Zabbix server配置  
<pre>
[root@zabbix-server /]# vim /etc/zabbix/zabbix_server.conf 
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=123456
</pre> 
## 1.9 启动Zabbix及httpd  
<pre>
[root@zabbix-server /]# systemctl start httpd
[root@zabbix-server /]# systemctl start zabbix-server
</pre> 
## 2.0 Web界面安装Zabbix  
http://192.168.56.11/zabbix/  
#备注：一路安装步骤即可安装，此处省略