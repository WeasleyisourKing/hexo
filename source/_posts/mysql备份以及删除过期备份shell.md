---
title: mysql数据库定期备份以及删除
date: 2019-04-24 18:34:23
tags: shell
---
•┈┈┈┈┈┈୨୧┈┈┈┈┈┈•
<!--more-->
\#!/bin/bash



mysql_user="备份账号"

mysql_password="数据库密码"

backup_db_arr=("数据库名称") #要备份的数据库名称，多个用空格分开隔开 如("db1" "db2" "db3")

backup_location=/root/backup/backup #备份数据存放位置，末尾请不要带"/",此项可以保持默认，程序会自动创建文件夹

expire_backup_delete="ON" #是否开启过期备份删除 ON为开启 OFF为关闭

expire_days=7 #过期时间天数 默认为三天，此项只有在expire_backup_delete开启时有效

welcome_msg="hello"

DATE=$(date +%Y-%m-%d_》》%X)

time=`date +%Y-%m-%d_%H:%M:%S`



\#"+%Y-%m-%d %H:%M:%S"

backup_time=`date +%Y-%m-%d_%H:%M:%S` #定义备份详细时间

backup_Ymd=`date +%Y-%m-%d` #定义备份目录中的年月日时间

backup_7ago=`date -d `7 days ago` +%Y%m%d`  #7天之前的日期

backup_dir=$backup_location/$backup_Ymd #备份文件夹全路径

welcome_msg="Welcome to use MySQL backup tools!" #欢迎语

 <!--more-->

\# 判断MYSQL是否启动,mysql没有启动则备份退出

mysql_ps=`ps -ef |grep mysql |wc -l`

mysql_listen=`netstat -an |grep LISTEN |grep $mysql_port|wc -l`

if [ [$mysql_ps == 0] -o [$mysql_listen == 0] ];

then

echo " $DATE ERROR:MySQL is not running! backup stop!" >>/root/backup/backup.log

echo "$time mysql is not running" >>/root/backup/mail.txt

else

echo  "$DATE $welcome_msg" >>/root/backup/backup.log

echo "[$time]"

fi

 

 \# 判断有没有定义备份的数据库，如果定义则开始备份，否则退出备份

if [ "$backup_db_arr" != "" ];then

\#dbnames=$(cut -d ',' -f1-5 $backup_database)

\#echo  $DATE "arr is (${backup_db_arr[@]})"

for dbname in ${backup_db_arr[@]}

do

echo  $time "database $dbname backup start..." >>/root/backup/backup.log#写入日志

echo $time "database $dbname backup start ">>/root/backup/mail.txt#写入邮件日志

```
mkdir -p $backup_dir
mysqldump  -u$mysql_user -p$mysql_password mall | gzip > $backup_dir/$dbname-$time.sql.gz
```

flag=`echo $?`

if [ $flag == "0" ];then

echo $DATE "database $dbname success backup to $backup_dir/$dbname-$backup_time.sql.gz" >>/root/backup/backup.log

echo $time "databases success backup ">>/root/backup/mail.txt#写入邮件日志

else

echo $DATE "database $dbname backup fail!" >>/root/backup/backup.log

echo $time "database backup fail">>/root/backup/mail.txt#写入邮件日志

fi

 

done

else

echo $DATE "ERROR:No database to backup! backup stop" >>/root/backup/backup.log

echo $time "no database to backup">>/root/backup/mail.txt

exit

fi

\# 如果开启了删除过期备份，则进行删除操作

if [ "$expire_backup_delete" == "ON" -a "$backup_location" != "" ];then

\#`find $backup_location/ -type d -o -type f -ctime $expire_days -exec rm -rf {} \;`

```
find $backup_location/ -type d -mtime $expire_days | xargs rm -rf
```

echo "$DATE Expired backup data delete complete!">>/root/backup/backup.log

echo $time "expired backup data delete complete">>/root/backup/mail.txt

fi

echo "$DATE  All database backup success! Thank you!❤️" >>/root/backup/backup.log

mail -s "[1/2]132.148.242.150 mysqlbackup $time" 邮箱名</root/backup/mail.txt

rm -rf /root/backup/mail.txt#删除邮件日志

exit

fi