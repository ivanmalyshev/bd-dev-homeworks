# Домашнее задание к занятию 4. «PostgreSQL»

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

* вывода списка БД,
* подключения к БД,
* вывода списка таблиц,
* вывода описания содержимого таблиц,
* выхода из psql.

---

## Ответ

Docker-compose.yaml
```yaml
version: '3.8'

services:
  db:
    container_name: pg13
    image: "postgres:13"
    ports:
      - "5432:5432"
    volumes:
      - ./pgdata:/var/lib/postgresql/data/pgdata
      - ./backup/:/var/lib/postgresql/backup
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=midx11011
      - POSTGRES_DB=postgres
      - PGDATA=/var/lib/postgresql/data/pgdata
```

```bash
bd-dev-homeworks/06-db-04-postgresql$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
19852c283278   postgres:13   "docker-entrypoint.s…"   4 minutes ago   Up 3 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg13

bd-dev-homeworks/06-db-04-postgresql$ docker exec -it pg13 bash
root@19852c283278:/# psql -U postgres
psql (13.12 (Debian 13.12-1.pgdg120+1))
Type "help" for help.

postgres=#
```

+ Список БД
```bash
postgres=# \l+
                                                                   List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   |  Size   | Tablespace |                Description                 
-----------+----------+----------+------------+------------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |                       | 7909 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +| 7761 kB | pg_default | unmodifiable empty database
           |          |          |            |            | postgres=CTc/postgres |         |            | 
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +| 7761 kB | pg_default | default template for new databases
           |          |          |            |            | postgres=CTc/postgres |         |            | 
(3 rows)
```

+ подключения к БД

```bash
postgres=# \conninfo
You are connected to database "postgres" as user "postgres" via socket in "/var/run/postgresql" at port "5432".
```
Вывод следующих комманды прикладывать не стал, т.к. вывод довольно большой, описаны списком:

+ Вывод списка таблиц - ` \dtS`
+ Вывод описания содержимого таблиц - `\dS+`
+ Выход из psql - `\q`


## Задача 2

Используя `psql`, создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders`
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления, и полученный результат.

---

## Ответ

Копирование дампа в докер-контейнер и восстановление
```bash
bd-dev-homeworks/06-db-04-postgresql$ docker cp test_data/test_dump.sql pg13:/tmp
Successfully copied 4.1kB to pg13:/tmp

docker exec -it pg13 bash
root@19852c283278:/# psql -U postgres -f /tmp/test_dump.sql test_database
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
COPY 8
 setval 
--------
      8
(1 row)

ALTER TABLE
```

запускаю ANALYZE для сбора статистики в таблице
```bash
postgres=# \c test_database
You are now connected to database "test_database" as user "postgres".
test_database=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | orders | table | postgres
(1 row)

test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```

Столбец таблицы orders  с наибольшим средним значением в байтах
```bash
test_database=# SELECT avg_width FROM pg_stats WHERE tablename='orders';
 avg_width 
-----------
         4
        16
         4
(3 rows)
```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

---

## Ответ

Да, можно было избежать разбиения таблицы вручную, необходимо было определить тип на моменте проектирования и создания - partitioned table

```bash
postgres=# 
begin;
    create table orders_new (
        id integer NOT NULL,
        title varchar(80) NOT NULL,
        price integer) partition by range(price);
    create table orders_less partition of orders_new for values from (0) to (499);
    create table orders_more partition of orders_new for values from (499) to (99999);
    insert into orders_new (id, title, price) select * from orders;
commit;
BEGIN
CREATE TABLE
CREATE TABLE
CREATE TABLE
INSERT 0 8
COMMIT
```

---

## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

---

## Ответ

+ Создание дампа  

```
root@6217ac53fe27:/var/lib/postgresql/backup# pg_dump -U postgres test_database > /var/lib/postgresql/backup/backup.sql
```

После этого можем забрать дамп с хостовой машины, т.к. директория ./backup пробрасывается с хоста в контейнер.

Для определения занчения столбца title можно было бы использовать индекс, для обеспечения уникальности.

