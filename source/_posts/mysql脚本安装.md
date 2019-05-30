---
title: 脚本安装mysql
date: 2019-04-24 17:34:23
tags: shell
---
•┈┈┈┈┈┈୨୧┈┈┈┈┈┈•
<!--more-->
            #!/bin/bash

echo "正在查找相关mysql程序"
mysql1=$(rpm -qa |grep mysql)
if [ ! -n "$mysql1" ]; then
  echo "服务器没有安装相关mysql程序"
else
  echo "服务器安装了相关mysql程序，正在卸载。。。"
  echo $mysql1
  rpm -e --nodeps $mysql1
  echo "mysql相关程序卸载完成"
fi

echo "------------------------------------------------"

echo "正在查看相关mariadb程序"
mariadb1=$(rpm -qa |grep mariadb)
if [ ! -n "$mariadb1" ]; then
  echo "服务器没有安装相关mariadb程序"
else
  echo "服务器安装了相关mariadb程序，正在卸载。。。"
  echo $mariadb1
  rpm -e --nodeps $mariadb1
  echo "mariadb相关程序卸载完成"
fi

echo "------------------------------------------------"

echo "正在检验是否存在mysql相关用户和组"
mysqlG=$(nl /etc/group | grep mysql)
mysqlS=$(nl /etc/shadow | grep mysql)
mysqlU=$(nl /etc/passwd | grep mysql)
if [ ! -n "$mysqlU" ]; then
  echo "不存在mysql用户"
else
  echo "存在mysql用户，正在删除。。。"
  echo $mysqlU
  userdel mysql #这里写死了，待优化
  echo "mysql用户删除成功"
fi

echo "------------------------------------------------"

echo "正在查找mysql相关目录"
mulu=$(find / -name mysql)
if [ ! -n "$mulu" ]; then
  echo "无相关mysql目录"
else
  echo "服务器存在相关mysql目录，正在删除。。。"
  echo $mulu
  rm -rf $mulu
  echo "删除完成！"
fi

echo "------------------------------------------------"

echo "正在查找mysqld.log"
MYSQL_LOG=$(find / -name mysqld.log)
if [ ! -n "$MYSQL_LOG" ]; then
  echo "没找到mysqld.log"
else
  echo "找到mysqld.log，正在删除。。。"
  echo $MYSQL_LOG
  rm -rf $MYSQL_LOG
  echo "删除完成!"
fi

echo "------------------------------------------------"

read -p "请输入centos的安装范围(只能输入el6或el7):" fanwei
if [ "$fanwei" = "el6" ]; then
  rpm -ivh mysql-community-common-5.7.21-1.el6.x86_64.rpm mysql-community-libs-5.7.21-1.el6.x86_64.rpm mysql-community-client-5.7.21-1.el6.x86_64.rpm mysql-community-server-5.7.21-1.el6.x86_64.rpm
  echo "mysql安装成功"
elif [ "$fanwei" = "el7" ]; then
  rpm -ivh mysql-community-common-5.7.21-1.el7.x86_64.rpm mysql-community-libs-5.7.21-1.el7.x86_64.rpm mysql-community-client-5.7.21-1.el7.x86_64.rpm mysql-community-server-5.7.21-1.el7.x86_64.rpm
  echo "mysql安装成功"
  mysql --version
  
else
  echo "输入错误，再见！"
  exit 0
fi

echo "------------------------------------------------"

echo "正在改写/etc/my.cnf"

aa="[mysqld]"
a="symbolic-links=0"
b="log-error=/var/log/mysqld.log"
c="pid-file=/var/run/mysqld/mysqld.pid"
d="character_set_server=utf8"
e="skip-name-resolve"
f="lower_case_table_names=1"
g="sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
h="[client]"
i="default-character-set=utf8"
j="[mysql]"

moren1=/var/lib/mysql
moren2=/var/lib/mysql/mysql.sock

read -p "是否需要更改数据存储目录？(y or n)：" yorn
if [ "$yorn" = "y" ]; then
  echo -e "\033[31m 提示：socket路径在datadir目录下，datadir默认路径/var/lib/mysql，socket默认路径/var/lib/mysql/mysql.sock \033[0m"
  read -p "输入datadir路径：" datadir
  read -p "输入socket路径：" socketdir
  echo $aa > /etc/my.cnf
  echo datadir=$datadir >> /etc/my.cnf
  echo socket=$socketdir >> /etc/my.cnf
  echo $a >> /etc/my.cnf
  echo $b >> /etc/my.cnf
  echo $c >> /etc/my.cnf
  echo $d >> /etc/my.cnf
  echo $e >> /etc/my.cnf
  echo $f >> /etc/my.cnf
  echo $g >> /etc/my.cnf
  echo $h >> /etc/my.cnf
  echo $i >> /etc/my.cnf
  echo $j >> /etc/my.cnf
  echo socket=$socketdir >> /etc/my.cnf
  echo "写入成功。"
elif [ "$yorn" = "n" ]; then
  echo "将使用默认路径/var/lib/mysql，/var/lib/mysql/mysql.sock"
  echo $aa > /etc/my.cnf
  echo datadir=$moren1 >> /etc/my.cnf
  echo socket=$moren2 >> /etc/my.cnf
  echo $a >> /etc/my.cnf
  echo $b >> /etc/my.cnf
  echo $c >> /etc/my.cnf
  echo $d >> /etc/my.cnf
  echo $e >> /etc/my.cnf
  echo $f >> /etc/my.cnf
  echo $g >> /etc/my.cnf
  echo $h >> /etc/my.cnf
  echo $i >> /etc/my.cnf
  echo $j >> /etc/my.cnf
  echo socket=$moren2 >> /etc/my.cnf
else
  echo "输入错误，拜您个拜!"
  exit 0
fi

echo "------------------------------------------------"
echo "启动mysql"
service mysqld restart
echo "开始配置mysql"

HOSTNAME="localhost"
PORT="3306"
USER_NAME="root"
PASSWORD_LOG=$(grep 'temporary password' /var/log/mysqld.log)
PASSWORD_LOG_NEW=${PASSWORD_LOG##*: }
PASSWORD_NEW="Mysql@123456"
echo $PASSWORD_LOG_NEW

mysql --connect-expired-password -u"$USER_NAME" -p"$PASSWORD_LOG_NEW" << EOF >/dev/null 2>&1
ALTER USER 'root'@'localhost' IDENTIFIED BY '$PASSWORD_NEW';
flush privileges;
use mysql;
update user set host='%' where user ='root' and host='localhost';
select host,user from user;
grant all privileges on *.* to root@'%' identified by '$PASSWORD_NEW' with grant option;
EOF

echo -e "\033[31m密码初始化完成，远程功能初始化完成。正在重启mysql\033[0m"
service mysqld restart

echo "------------------------------------------------"

read -p "请输入centos版本便于设置mysql开机自启动(centos6/centos7)" VERSION1
if [ "$VERSION1" = "centos6" ]; then
chkconfig mysqld on
echo "开机自启动设置成功"
elif [ "$VERSION1" = "centos7" ]; then
systemctl enable mysqld.service
echo "开机自启动设置成功"
else
echo "输入错误，自己手动设置吧!"
exit 0
fi

