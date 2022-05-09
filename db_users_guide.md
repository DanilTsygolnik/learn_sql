## Работа с ролями

Пользователям PostgreSQL, работающим с БД, назначаются т.н. роли[^posgtgres-db-roles]. Это набор параметров, включающих id пользователя, а также операции, которые он может производить внутри СУБД, с БД в частности.

На практике удобно создавать роли с теми же именами, что и у пользователя системы (username - username), хотя концептуально данные идентификаторы никак не связаны.


Простейший вариант[^createuser]:
```
# простейший вариант
sudo -u postgres createuser -d username
```
Данная cli-команда упрощает работу с командой СУБД `CREATE ROLE`[^sql-createrole].

## Базовая работа с БД через cli

К базами данных можно работать через cli с помощью утилиты `psql`[^psql-app]:
```
sudo -u postgres psql -d <database>

$ sudo -u postgres psql -d test_user
test_user=# \conninfo
You are connected to database "test_user" as user "postgres" via socket in "/run/postgresql" at port "5432".
```
Поскольку _postgres_ является superuser'ом, подключиться можно к любой БД (??).

Подключиться из под другой роли, если позволят права доступа:
```
sudo -u postgres psql -d <database> -U <username>

$ sudo -u postgres psql -d test_user -U test_user
test_user=> \conninfo
You are connected to database "test_user" as user "test_user" via socket in "/run/postgresql" at port "5432".
```

Команду `psql` можно вызывать в текущей сессии напрямую, если имеется роль, имя которой совпадает с именем пользователяш:
```
$ whoami
dts

$ psql -d test_user
psql: error: FATAL:  role "dts" does not exist

# создаем роли и убеждаемся, что все работает
$ sudo -u postgres createuser -d dts
$ psql -d test_user
test_user=> \conninfo
You are connected to database "test_user" as user "dts" via socket in "/run/postgresql" at port "5432".
```
Это значительно упрощает работу, т.к. теперь можно извлекать некоторые сведения о БД из сессии терминала, а не только в интерактивной оболочке (??) postgresql. Например:
```
$ psql -l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 test_user | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
(4 rows)
```
Команда `psql username` подключает пользователя к БД с тем же именем. Чтобы это сработало, нужно заранее создать такую:[^digocn-roles] 
```
$ sudo -u postgres createdb username
```


[^createuser]: https://www.postgresql.org/docs/14/app-createuser.html
[^digocn-roles]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04
[^posgtgres-db-roles]: https://www.postgresql.org/docs/14/database-roles.html
[^sql-createrole]: https://www.postgresql.org/docs/14/sql-createrole.html
[^psql-app]: https://www.postgresql.org/docs/current/app-psql.html
