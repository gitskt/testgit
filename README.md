1. ### Working environment exeplification ###
2. ### Example commands for Apache and PHP install ###
3. ### Example commands to verify Apache and PHP versions ###
4. ### Example command to verify Apache process ###
5. ### Example commands for Mysql install ###
6. ### Example command to verify Mysql process ###
7. ### Example commands for Mysql configuration ###
8. ### Example commands for Mysql connect and queries ###
9. ### Example script for Apache and Mysqld process monitor and auto-restart ###


1. ### Working environment exeplification ###
- ESXi VM with 2x vCPU, 4GB vRAM, 20GB Disk, Paravirtual SCSI, 2x NIC
- Guest OS: RHEL 7.5 (Maipo)

2. ### Example commands for Apache and PHP install ###
yum install httpd httpd-devel httpd-manual httpd-tools php php-common
yum install php-pear php-pdo php-mysql php-gd php-xml

3. ### Example commands to verify Apache and PHP versions ###
[root@test ~]# php -v
PHP 5.4.16 (cli) (built: Oct 29 2019 10:01:20) 
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.4.0, Copyright (c) 1998-2013 Zend Technologies

[root@test ~]# httpd -v
Server version: Apache/2.4.6 (Red Hat Enterprise Linux)
Server built:   Jun  9 2019 13:01:04

4. ### Example command to verify Apache process ###
[root@test ~]# systemctl status httpd
â— httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2019-11-11 15:10:58 EET; 20h ago
     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 8520 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=0/SUCCESS)
  Process: 29512 ExecReload=/usr/sbin/httpd $OPTIONS -k graceful (code=exited, status=0/SUCCESS)
 Main PID: 8806 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
    Tasks: 6
   CGroup: /system.slice/httpd.service
           â”œâ”€8806 /usr/sbin/httpd -DFOREGROUND
           â”œâ”€8807 /usr/sbin/httpd -DFOREGROUND
           â”œâ”€8808 /usr/sbin/httpd -DFOREGROUND
           â”œâ”€8810 /usr/sbin/httpd -DFOREGROUND
           â”œâ”€8811 /usr/sbin/httpd -DFOREGROUND
           â””â”€8812 /usr/sbin/httpd -DFOREGROUND

Nov 11 15:10:58 undercloud systemd[1]: Starting The Apache HTTP Server...
Nov 11 15:10:58 undercloud systemd[1]: Started The Apache HTTP Server.

5. ### Example commands for Mysql install ###
rpm -Uvh mysql-community-release-el7-5.noarch.rpm
yum install mysql-community-server

6. ### Example command to verify Mysql process ###
[root@test ~]# systemctl status mysqld
â— mysqld.service - MySQL Community Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2019-11-11 15:10:48 EET; 20h ago
  Process: 8587 ExecStartPost=/usr/bin/mysql-systemd-start post (code=exited, status=0/SUCCESS)
  Process: 8569 ExecStartPre=/usr/bin/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 8586 (mysqld_safe)
    Tasks: 23
   CGroup: /system.slice/mysqld.service
           â”œâ”€8586 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           â””â”€8752 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mysqld.log --pid-file=/var/run/mysqld/my...

Nov 11 15:10:46 undercloud systemd[1]: Starting MySQL Community Server...
Nov 11 15:10:47 undercloud mysqld_safe[8586]: 191111 15:10:47 mysqld_safe Logging to '/var/log/mysqld.log'.
Nov 11 15:10:47 undercloud mysqld_safe[8586]: 191111 15:10:47 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
Nov 11 15:10:48 undercloud systemd[1]: Started MySQL Community Server.

7. ### Example commands for Mysql configuration ###
mysql_secure_installation
systemctl restart mysqld

8. ### Example commands for Mysql connect and queries ###
[root@test ~]# mysql -h localhost -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.46 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE mydb;
Query OK, 1 row affected (0.00 sec)

mysql> use mydb;
Database changed
mysql> create table test_tbl(id INT NOT NULL AUTO_INCREMENT,title VARCHAR(100) NOT NULL,name VARCHAR(40) NOT NULL,PRIMARY KEY ( id ));
Query OK, 0 rows affected (0.05 sec)

mysql> show tables;
+----------------+
| Tables_in_mydb |
+----------------+
| test_tbl       |
+----------------+
1 row in set (0.00 sec)

mysql> desc test_tbl;
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int(11)      | NO   | PRI | NULL    | auto_increment |
| title | varchar(100) | NO   |     | NULL    |                |
| name  | varchar(40)  | NO   |     | NULL    |                |
+-------+--------------+------+-----+---------+----------------+
3 rows in set (0.01 sec)

9. ### Example script for Apache and Mysqld process monitor and auto-restart ###
[root@test ~]# crontab -l
* * * * * /usr/bin/service_monitor.sh mysqld,httpd
[root@test ~]# ll /usr/bin/service_monitor.sh
-rwxr-xr-x. 1 root root 756 Nov 12 12:22 /usr/bin/service_monitor.sh
[root@test ~]# cat /usr/bin/service_monitor.sh
#!/bin/sh
services=$1
lenght=`echo $services | awk -F ',' '{ print NF }'`
for(( i=1; i<=$lenght; i++ ))
do
        service=`echo $services | awk -F ',' '{ print $'$i' }'`
        logfile=/var/log/watchdog.`date '+%Y-%m-%d'`.log
        /bin/systemctl status $service.service | grep running
        if [ "$?" = "0" ];then
                log="${service^} status `date '+%Y-%m-%d %H:%M:%S'`: service running"
                echo $log >> $logfile
        else
                log="${service^} status `date '+%Y-%m-%d %H:%M:%S'`: service not running, starting ..."
                echo $log >> $logfile
                /bin/systemctl start $service.service
                sleep 10
                /bin/systemctl status $service.service | grep running
                if [ "$?" = "0" ];then
                        log="${service^} status `date '+%Y-%m-%d %H:%M:%S'`: service started"
                        echo $log >> $logfile
                fi
        fi
done
[root@test ~]# tail -10f /var/log/watchdog.2019-11-12.log
Mysqld status 2019-11-12 12:32:01: service running
Httpd status 2019-11-12 12:32:01: service running
Mysqld status 2019-11-12 12:33:01: service running
Httpd status 2019-11-12 12:33:01: service running
Mysqld status 2019-11-12 12:34:01: service running
Httpd status 2019-11-12 12:34:01: service running
Mysqld status 2019-11-12 12:35:01: service running
Httpd status 2019-11-12 12:35:01: service running
Mysqld status 2019-11-12 12:36:01: service running
Httpd status 2019-11-12 12:36:01: service running
Mysqld status 2019-11-12 12:37:02: service not running, starting ...
Mysqld status 2019-11-12 12:37:14: service started
Httpd status 2019-11-12 12:37:14: service not running, starting ...
Httpd status 2019-11-12 12:37:24: service started
Mysqld status 2019-11-12 12:38:01: service running
Httpd status 2019-11-12 12:38:01: service running
Mysqld status 2019-11-12 12:39:01: service running
Httpd status 2019-11-12 12:39:01: service running
