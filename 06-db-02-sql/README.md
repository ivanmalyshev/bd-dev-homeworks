# Домашнее задание к занятию 2. «SQL»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

---

## Ответ

[ссылка на md. файл](https://github.com/ivanmalyshev/bd-dev-homeworks/06-db-02-sql/docker.compose.yml)

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db; (create database test_bd, create role test-admin-user)
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db; (grant * on table auth to test-admin-user)
- создайте пользователя test-simple-user; (create role test-simple-user)
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.

## Ответ

Список БД с описанием
```
postgres=# \l+
                                                                   List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   |  Size   | Tablespace |                Description                 
-----------+----------+----------+------------+------------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |                       | 8137 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +| 7833 kB | pg_default | unmodifiable empty database
           |          |          |            |            | postgres=CTc/postgres |         |            | 
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +| 7833 kB | pg_default | default template for new databases
           |          |          |            |            | postgres=CTc/postgres |         |            | 
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 |                       | 7833 kB | pg_default | 
(4 rows)
```

Список таблиц
```
postgres=# SELECT table_name FROM information_schema.tables
WHERE table_schema NOT IN ('information_schema','pg_catalog');
 table_name 
------------
 orders
 clients
(2 rows)

postgres=# test_db=# \d+
                               List of relations
 Schema |         Name         |   Type   |  Owner   |    Size    | Description 
--------+----------------------+----------+----------+------------+-------------
 public | clients              | table    | postgres | 8192 bytes | 
 public | clients_id_seq       | sequence | postgres | 8192 bytes | 
 public | clients_zakaz_id_seq | sequence | postgres | 8192 bytes | 
 public | orders               | table    | postgres | 8192 bytes | 
 public | orders_id_seq        | sequence | postgres | 8192 bytes | 
(5 rows)
```

Запрос и список пользователей с правами над таблицами test_db

```
postgres=# SELECT table_name,grantee,privilege_type 
FROM information_schema.table_privileges
WHERE table_schema NOT IN ('information_schema','pg_catalog');
 table_name |     grantee      | privilege_type 
------------+------------------+----------------
 orders     | postgres         | INSERT
 orders     | postgres         | SELECT
 orders     | postgres         | UPDATE
 orders     | postgres         | DELETE
 orders     | postgres         | TRUNCATE
 orders     | postgres         | REFERENCES
 orders     | postgres         | TRIGGER
 clients    | postgres         | INSERT
 clients    | postgres         | SELECT
 clients    | postgres         | UPDATE
 clients    | postgres         | DELETE
 clients    | postgres         | TRUNCATE
 clients    | postgres         | REFERENCES
 clients    | postgres         | TRIGGER
 orders     | test-admin-user  | INSERT
 orders     | test-admin-user  | SELECT
 orders     | test-admin-user  | UPDATE
 orders     | test-admin-user  | DELETE
 orders     | test-admin-user  | TRUNCATE
 orders     | test-admin-user  | REFERENCES
 orders     | test-admin-user  | TRIGGER
 clients    | test-admin-user  | INSERT
 clients    | test-admin-user  | SELECT
 clients    | test-admin-user  | UPDATE
 clients    | test-admin-user  | DELETE
 clients    | test-admin-user  | TRUNCATE
 clients    | test-admin-user  | REFERENCES
 clients    | test-admin-user  | TRIGGER
 orders     | test-simple-user | INSERT
 orders     | test-simple-user | SELECT
 orders     | test-simple-user | UPDATE
 orders     | test-simple-user | DELETE
 clients    | test-simple-user | INSERT
 clients    | test-simple-user | SELECT
 clients    | test-simple-user | UPDATE
 clients    | test-simple-user | DELETE
(36 rows)
```

## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:

    - запросы,
    - результаты их выполнения.

---

## Ответ

```
INSERT INTO orders (name,price) VALUES
('Шоколад',10),
('Принтер',3000),
('Книга',500),
('Монитор',7000),
('Гитара',4000);
INSERT 0 5
```

```
INSERT INTO clients (fullname,country,zakaz_id) VALUES
('Иванов Иван Иванович','USA',NULL),
('Петров Петр Петрович','Canada',NULL),
('Иоганн Себастьян Бах','Japan',NULL),
('Ронни Джеймс Дио','Russia',NULL),
('Ritchie Blackmore','Russia',NULL);
INSERT 0 5
```
```
test_db=# select count(*) from clients;
 count
-------
     5
```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
 
Подсказка: используйте директиву `UPDATE`.

---

## Ответ

```
postgres=# UPDATE clients SET order_number=3 WHERE id=1;
UPDATE 1
postgres=# UPDATE clients SET order_number=4 WHERE id=2;
UPDATE 1
postgres=# UPDATE clients SET order_number=5 WHERE id=3;
UPDATE 1


SELECT * FROM clients;
SELECT * FROM clients WHERE order_number IS NOT NULL;
 id |      last_name       | country | order_number 
----+----------------------+---------+--------------
  4 | Ронни Джеймс Дио     | Russia  |             
  5 | Ritchie Blackmore    | Russia  |             
  1 | Иванов Иван Иванович | USA     |            3
  2 | Петров Петр Петрович | Canada  |            4
  3 | Иоганн Себастьян Бах | Japan   |            5
(5 rows)

 id |      last_name       | country | order_number 
----+----------------------+---------+--------------
  1 | Иванов Иван Иванович | USA     |            3
  2 | Петров Петр Петрович | Canada  |            4
  3 | Иоганн Себастьян Бах | Japan   |            5
(3 rows)


```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

---

## Ответ

```
EXPLAIN (FORMAT YAML) SELECT * FROM clients WHERE order_number IS NOT NULL;
                QUERY PLAN                
------------------------------------------
 - Plan:                                 +
     Node Type: "Seq Scan"               +
     Parallel Aware: false               +
     Relation Name: "clients"            +
     Alias: "clients"                    +
     Startup Cost: 0.00                  +
     Total Cost: 13.50                   +
     Plan Rows: 348                      +
     Plan Width: 204                     +
     Filter: "(order_number IS NOT NULL)"
(1 row)

```

EXPLAIN - выводит план выполнения SQL-запроса, генерируемый планировщиком для заданного оператора. Этот план показывает, как будут сканироваться таблицы (последовательно, по индексу и пр.) и какой алгоритм соединения будет объединять считанных из них строки.
В том числе можно узнать время на выполнение запроса, что при оптимизации работы БД является очень полезной информацией.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

## Ответ

1. Делаем бекап БД
```
pg_dumpall -U postgres > /var/lib/postgresql/backup/backup_table

ls -alh /var/lib/postgresql/backup/
total 16K
drwxr-xr-x 2 root     root     4.0K Sep 18 15:27 .
drwxr-xr-x 1 postgres postgres 4.0K Sep 18 15:20 ..
-rw-r--r-- 1 root     root     7.1K Sep 18 15:27 backup_table


```

Удаляем контейнер очищаем данные в директории ./postgres-data

```
docker-compose down
[+] Running 2/2
 ✔ Container 06-db-02-sql-postgres-1  Removed                                                                                                                                                                 0.2s 
 ✔ Network 06-db-02-sql_default       Removed
 
sudo rm -rf postgres-data/*

```
Поднимаем новый контейнер и восстанавливаем данные из директории ./backup

```
docker-compose up -d
[+] Running 2/2
 ✔ Network 06-db-02-sql_default       Created                                                                                                                                                                 0.0s 
 ✔ Container 06-db-02-sql-postgres-1  Started                       
 docker exec -it 06-db-02-sql-postgres-1 bash

docker exec -it 06-db-02-sql-postgres-1

root@421d3094780d:/# psql -U postgres
psql (12.16 (Debian 12.16-1.pgdg120+1))
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)

root@a6bc73052f42:/# psql -U postgres -f /var/lib/postgresql/backup/backup_table 

```

Проверяем

```
root@421d3094780d:/# psql -U postgres
psql (12.16 (Debian 12.16-1.pgdg120+1))
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
(4 rows)


postgres=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner   
--------+---------+-------+----------
 public | clients | table | postgres
 public | orders  | table | postgres
(2 rows)



```

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

