1.
Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя psql.

Воспользуйтесь командой \? для вывода подсказки по имеющимся в psql управляющим командам.

Найдите и приведите управляющие команды для:

  - вывода списка БД
  - подключения к БД
  - вывода списка таблиц
  - вывода описания содержимого таблиц
  - выхода из psql


  - Запуск:
    
        C:\post>docker run --name pdb3 -v C:\post\vol1\:/var/lib/postgresql/data -e POSTGRES_PASSWORD=5566496 -p 5432:5432 -d postgres
  
  - Подключение:
  
        C:\post>docker exec -it pdb3 bash
        root@0e3fa551b52d:/# psql -h localhost -U postgres

  - Вывод списка БД:

        postgres-# \l
                                 List of databases
          Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
        -----------+----------+----------+------------+------------+-----------------------
        postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
        template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                  |          |          |            |            | postgres=CTc/postgres
        template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                    |          |          |            |            | postgres=CTc/postgres
        (3 rows)

  - Подключение к БД:

        postgres-# \c postgres
        You are now connected to database "postgres" as user "postgres".


  - Вывод списка таблиц:

        postgres-# \dt
        Did not find any relations.
        Можно использовать команду \dtS

  - Вывод описания таблиц:

        postgres-# \dS+

  - Выход из psql:

        postgres-# \q
        root@0e3fa551b52d:/#

2. 
Используя psql создайте БД test_database.

Изучите бэкап БД.

Восстановите бэкап БД в test_database.

Перейдите в управляющую консоль psql внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу pg_stats, найдите столбец таблицы orders с наибольшим средним значением размера элементов в байтах.

Приведите в ответе команду, которую вы использовали для вычисления и полученный результат.


  - Создание БД:


       postgres=# create database test_database ;
        CREATE DATABASE 

  - Восстановите бэкап :

        /var/lib/postgresql/data# psql -U postgres -f ./test_dump.sql test_database


  - Перейдите в управляющую консоль psql внутри контейнера.
Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

        postgres=# \c test_database ;
        You are now connected to database "test_database" as user "postgres".
    

        test_database=# analyze verbose public.orders ;
        INFO:  analyzing "public.orders"
        INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
        ANALYZE
        
  - Используя таблицу pg_stats, найдите столбец таблицы orders с наибольшим средним значением размера элементов в байтах.

        test_database=# select avg_width from pg_stats where tablename='orders';
         avg_width
        -----------
         4
        16
         4
        (3 rows)

3. 
Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

        test_database=# alter table orders rename to orders_old;
        ALTER TABLE
        test_database=# create table orders (id integer, title varchar(80), price integer) partition by range(price);
        CREATE TABLE
        test_database=# create table orders_less partition of orders for values from (0) to (499);
        CREATE TABLE
        test_database=# create table orders_more partition of orders for values from (499) to (99999);
        CREATE TABLE
        test_database=# insert into orders (id, title, price) select * from orders_old;
        INSERT 0 8
        
        При проектировании таблицу можно было сделать многосекционной.

4.
Используя утилиту pg_dump создайте бекап БД test_database.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца title для таблиц test_database?


        /var/lib/postgresql/data# pg_dump -U postgres -d test_database >test_database_dump.sql
         Для уникальности можно добавить индекс или первичный ключ.
           CREATE INDEX ON orders ((lower(title)));