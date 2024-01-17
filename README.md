# MariaDB-Master-Mater-DataBase-Replication---CentOS-7--MariaDB-10.2.41
MariaDB Master-Master DataBase Replication Configuration using CentOS 7.9 &amp; MariaDB 10.2.41

## Description

Master-Master DataBase Replication Configuration, I'm using two CentOS 7.9 nodes for database replication, below are nodes details:

##Node 1:
IP: 192.168.0.216
OS: CentOS Linux release 7.9.2009 (Core)
MariaDb: mysql  Ver 15.1 Distrib 10.2.41-MariaDB, for Linux (x86_64) using readline 5.1

##Node 2:
IP: 192.168.0.217
OS: CentOS Linux release 7.9.2009 (Core)
MariaDb: mysql  Ver 15.1 Distrib 10.2.41-MariaDB, for Linux (x86_64) using readline 5.1

##1. Installation of MariaDB (Manual Method)

cd /opt/
wget https://archive.mariadb.org/mariadb-10.2.41/bintar-linux-systemd-x86_64/mariadb-10.2.41-linux-systemd-x86_64.tar.gz --no-check-certificate
tar -xvf mariadb-10.2.41-linux-systemd-x86_64.tar.gz
mkdir -p /opt/mariadb/mysql/
mv mariadb-10.2.41-linux-systemd-x86_64/* mariadb/mysql/


##2. Create mysql Group

groupadd mysql

##3. Create mysql User with primary group mysql

useradd -g mysql mysql

##4. Change MariaDB binary file Ownership

chown -R mysql:mysql /opt/mariadb/mysql/

##Copy my.cnf file 

cp /opt/mariadb/mysql/support-files/my-small.cnf /etc/my.cnf
cp: overwrite ‘/etc/my.cnf’? y

##5. Create Data Directory for Mysql

mkdir -p /var/lib/mysql

##6. Edit MariaDB config file /etc/my.cnf

[server] #create if not exists#
basedir=/opt/mariadb/mysql
datadir=/var/lib/mysql
user=mysql


##7. Initialize System tables

/opt/mariadb/mysql/scripts/mysql_install_db --user=mysql --basedir=/opt/mariadb/mysql

##8. Enable mysql on System Boot

ln -s /opt/mariadb/mysql/support-files/mysql.server /etc/init.d/mysql
cd /etc/init.d/
chkconfig --add mysql


##9. Start & Verify Mysql Service

systemctl start mysql
systemctl status mysql

##10. Set Mysql root password

/opt/mariadb/mysql/bin/mysqladmin -uroot password
New password: <Enter_New_passwd>
Confirm new password:<Confirm_New_passwd>

##11. Initialize secure

/opt/mariadb/mysql/bin/mysql_secure_installation --basedir=/opt/mariadb/mysql

##Sample Output-Start
print: /opt/mariadb/mysql/bin/my_print_defaults

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

##Sample Output-End

##12. Add Binary File location into bashrc file

vi /root/.bashrc

PATH=$PATH:/opt/mariadb/mysql/bin
MANPATH=$MANPATHL/opt/mariadb/mysql/man

##13. Reboot Node 1 & after boot verify mysql status

systemctl status mysql


##14. Allow Mysql port on firewalld

firewall-cmd --add-port=3306/tcp --permanen
firewall-cmd --reload

##15. Repeate Step 1 to 14 on Node 2


##16. Node 1 : Connect to mysql Database & create a sample Database , we will use the database for replication

mysql -uroot -p<your_passwd>
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.2.41-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database country;
Query OK, 1 row affected (0.01 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| country            |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> use country;
Database changed
MariaDB [country]> create table country_master (name varchar(15));
Query OK, 0 rows affected (0.01 sec)

MariaDB [country]> insert into country_master values ('India');
Query OK, 1 row affected (0.00 sec)

MariaDB [country]> insert into country_master values ('Nepal');
Query OK, 1 row affected (0.01 sec)

MariaDB [country]> insert into country_master values ('China');
Query OK, 1 row affected (0.00 sec)

MariaDB [country]> insert into country_master values ('US');
Query OK, 1 row affected (0.00 sec)

MariaDB [country]> select * from country_master;
+-------+
| name  |
+-------+
| India |
| Nepal |
| China |
| US    |
+-------+
4 rows in set (0.00 sec)

MariaDB [country]>

MariaDB [(none)]> quit;
Bye

##17. Node 1: Create backup of the country Database, copy to Node 2 , we will restore it later

mysqldump country > /tmp/country.sql -uroot -p 

rsync -avzr /tmp/country.sql root@192.168.0.217:/tmp

##18. Node 2 Create country Database and Restore the copied dump

MariaDB [(none)]> create database country;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| country            |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> quit;
Bye

mysql -uroot -p country < /tmp/country.sql

Reconnect DB & verify tables

MariaDB [(none)]> use country;
Database changed
MariaDB [country]> select * from country_master;
+-------+
| name  |
+-------+
| India |
| Nepal |
| China |
| US    |
+-------+
4 rows in set (0.01 sec)


##18. Node 1: & Node 2: Edit /etc/my.cnf

[mysqld]#Under mysqld

server-id = 101 #102 for Node 2
binlog-do-db=country #country is my database name , change it accordingly#
relay-log = mysql-relay-bin
relay-log-index = mysql-relay-bin.index
log-bin = mysql-bin

##19. Node 1: & Node 2: Connect to mysql - create Replication user & grant permission

create user 'replusr'@'%' identified by 'replusrpswd'; 
grant replication slave on *.* to 'replusr'@'%';
flush privileges;
flush tables with read lock;
stop slave;
show master satus;

##Node 1

MariaDB [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 |      654 | country      |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

##Node 2

MariaDB [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |     2087 | country      |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)


##20. Node 1: Connect to DB & run belwo querry

MariaDB [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 |      654 | country      |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST='192.168.0.217', MASTER_USER='replusr', MASTER_PASSWORD='replusrpswd', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=2087;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.217
                  Master_User: replusr
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 2087
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 555
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2087
              Relay_Log_Space: 864
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 102
               Master_SSL_Crl:
           Master_SSL_Crlpath:
                   Using_Gtid: No
                  Gtid_IO_Pos:
      Replicate_Do_Domain_Ids:
  Replicate_Ignore_Domain_Ids:
                Parallel_Mode: conservative
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
1 row in set (0.00 sec)

ERROR: No query specified

##verify for Seconds_Behind_Master: 0 & Slave_IO_State: Waiting for master to send event

##21. Node 2: Connect to DB & run belwo querry

MariaDB [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |     2087 | country      |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> stop slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)

MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST='192.168.0.216', MASTER_USER='replusr', MASTER_PASSWORD='replusrpswd', MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=654;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.216
                  Master_User: replusr
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 654
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 555
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 654
              Relay_Log_Space: 864
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 101
               Master_SSL_Crl:
           Master_SSL_Crlpath:
                   Using_Gtid: No
                  Gtid_IO_Pos:
      Replicate_Do_Domain_Ids:
  Replicate_Ignore_Domain_Ids:
                Parallel_Mode: conservative
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
1 row in set (0.01 sec)

ERROR: No query specified

##verify for Seconds_Behind_Master: 0 & Slave_IO_State: Waiting for master to send event


##22. Verify Replication by inserting/deleting values on both server

##Insert new values in Node 1

MariaDB [country]> insert into country_master values ('UK');
Query OK, 1 row affected (0.01 sec)

MariaDB [country]> select * from country_master;
+-------+
| name  |
+-------+
| India |
| Nepal |
| China |
| US    |
| UK    |
+-------+
5 rows in set (0.01 sec)

MariaDB [country]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 |      837 | country      |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.01 sec)


##Reflecting on Node 2:

MariaDB [country]> select * from country_master;
+-------+
| name  |
+-------+
| India |
| Nepal |
| China |
| US    |
| UK    |
+-------+
5 rows in set (0.01 sec)

MariaDB [country]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 |      837 | country      |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.01 sec)


##Delete a row from Node 2

MariaDB [country]> delete from country_master where name='China';
Query OK, 1 row affected (0.00 sec)

MariaDB [country]> select * from country_master;
+-------+
| name  |
+-------+
| India |
| Nepal |
| US    |
| UK    |
+-------+
4 rows in set (0.00 sec)

##Reflecting on Node 1 

MariaDB [country]> select * from country_master;
+-------+
| name  |
+-------+
| India |
| Nepal |
| US    |
| UK    |
+-------+
4 rows in set (0.00 sec)

#Thank You Hope this content will help in configuring MariaDB Master-Master Replication.
