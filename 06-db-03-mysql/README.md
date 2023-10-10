# Домашнее задание к занятию 3. «MySQL»

## Введение

Перед выполнением задания вы можете ознакомиться с
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с этим контейнером.

---

### Ответ

docker-compose.yml
```yaml
version: '3.9'

services:
  mysql:
    image: mysql:8
    ports:
      - 3306:3306
    volumes:
      - ./06-mysql/apps/mysql:/var/lib/mysql
      - ./06-mysql/config/conf.d:/etc/mysql/conf.d
    environment:
      - MYSQL_DATABASE=netology_db
      - MYSQL_ROOT_PASSWORD=midx11011
      - MYSQL_PASSWORD=midx11011
      - MYSQL_USER=netology_user
```

```
 docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
8edcd89a7db0   mysql:8   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   06-db-03-mysql-mysql-1
```

Изучите бекап БД и востановитесь:

```bash
mid@mid-desktop:~/bd-dev-homeworks/06-db-03-mysql$ docker cp test_data/test_dump.sql 06-db-03-mysql-mysql-1:/tmp
Successfully copied 4.1kB to 06-db-03-mysql-mysql-1:/tmp
mid@mid-desktop:~/bd-dev-homeworks/06-db-03-mysql$ docker exec -it 06-db-03-mysql-mysql-1 bash
bash-4.4# mysql -u root -p netology_db < /tmp/test_dump.sql 
Enter password: 
bash-4.4# 
```

Версия mysql сервера и список таблиц БД

```
mysql> \s
--------------
mysql  Ver 8.1.0 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          9
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.1.0 MySQL Community Server - GPL
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 7 min 13 sec

Threads: 2  Questions: 49  Slow queries: 0  Opens: 138  Flush tables: 3  Open tables: 56  Queries per second avg: 0.113
```

```

mysql> USE netology_test;
ERROR 1049 (42000): Unknown database 'netology_test'
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| netology_db        |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> USE netology_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------------+
| Tables_in_netology_db |
+-----------------------+
| orders                |
+-----------------------+
1 row in set (0.00 sec)

mysql> 
```

Количество записей с price > 300.

```mysql 
mysql> select count(*) from orders where price > 300;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)

mysql> 
```

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

* плагин авторизации mysql_native_password
* срок истечения пароля — 180 дней
* количество попыток авторизации — 3
* максимальное количество запросов в час — 100
* аттрибуты пользователя: 
  * Фамилия "Pretty"
  * Имя "James".

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.

Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и
**приведите в ответе к задаче**.

---

### Ответ

Создание пользователя с заданными параметрами:
```mysql
mysql> create user 'test'@'localhost'
    -> identified with mysql_native_password by 'test-pass'
    -> with max_connections_per_hour 100
    -> password expire interval 180 day
    -> failed_login_attempts 3 password_lock_time 2
    -> attribute '{"first_name":"James", "last_name":"Pretty"}';
Query OK, 0 rows affected (0.02 sec)

mysql> grant select on netology_db.* to test@localhost;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> select * from information_schema.user_attributes where user = 'test';
+------+-----------+------------------------------------------------+
| USER | HOST      | ATTRIBUTE                                      |
+------+-----------+------------------------------------------------+
| test | localhost | {"last_name": "Pretty", "first_name": "James"} |
+------+-----------+------------------------------------------------+
1 row in set (0.00 sec)
```

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:

* на `MyISAM`,
* на `InnoDB`.


## Ответ

```mysql
mysql> SELECT table_schema,table_name,engine FROM information_schema.tables WHERE table_schema = DATABASE();
+--------------+------------+--------+
| TABLE_SCHEMA | TABLE_NAME | ENGINE |
+--------------+------------+--------+
| netology_db  | orders     | InnoDB |
+--------------+------------+--------+
1 row in set (0.00 sec)

mysql> set profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> alter table orders engine = InnoDB;
Query OK, 0 rows affected (0.07 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> Alter table orders ENGINE = MyIsam;
Query OK, 5 rows affected (0.05 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> show profiles;
+----------+------------+------------------------------------+
| Query_ID | Duration   | Query                              |
+----------+------------+------------------------------------+
|        1 | 0.07211475 | alter table orders engine = InnoDB |
|        2 | 0.05096400 | Alter table orders ENGINE = MyIsam |
+----------+------------+------------------------------------+
2 rows in set, 1 warning (0.00 sec)
```



## Задача 4

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

* скорость IO важнее сохранности данных;
* нужна компрессия таблиц для экономии места на диске;
* размер буффера с незакомиченными транзакциями 1 Мб;
* буффер кеширования 30% от ОЗУ;
* размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

---
### Ответ
```mysql
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.1/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M

# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.1/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid

#IO Speed
# 0 - скорость
# 1 - сохранность
# 2 - универсальный параметр
innodb_flush_log_at_trx_commit = 0

#Compression
# Barracuda - формат файла с сжатием
innodb_file_format=Barracuda

#Set buffer
innodb_log_buffer_size  = 1M

#Set Cache size
key_buffer_size = 307М

#Set log size
max_binlog_size = 100M

[client]
socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/
```

---

### Как оформить ДЗ

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---