## Управление ролями

Список ролей `psql -c "\du"
Создать новую роль с определенными привилегиями

Создание, удаление и изменение ролей требует прав `CREATEROLE`, поэтому удобнее всего выполнять данную операцию из под системной роли _postgres_.

В самом простом варианте создается роль со следующим набором дефолтных атрибутов (опция `-e` выводит на экран сопутствующие созданию роли запросы):
```
$ sudo -u postgres createuser john -e
SELECT pg_catalog.set_config('search_path', '', false);
CREATE ROLE john NOSUPERUSER NOCREATEDB NOCREATEROLE INHERIT LOGIN;
```
Благодаря атрибуту `LOGIN` с помощью роли _john_ можно авторизоваться при подключении к серверу (имя пользователя, которое указываем при создании сервера в pgadmin4, а также роль, для которой происходит запрос при подключении к этому серверу).

Для базового использования достаточно расширить команду атрибутом `-d`, чтобы из под роли можно было создавать БД:
```
sudo -u postgres createuser -d kate
```
Пользователю можно будет также присвоить пароль:
```


```



Создание роли на примерах:


создать роль dts


```
$ createuser -P -s -e joe
Enter password for new role: xyzzy
Enter it again: xyzzy
CREATE ROLE joe PASSWORD 'md5b5f5ba1a423792b526f799ae4eb3d59e' SUPERUSER CREATEDB CREATEROLE INHERIT LOGIN;

createuser -d
CREATE USER davide WITH PASSWORD 'jw8s0F4';
CREATE ROLE jonathan LOGIN;
CREATE ROLE admin WITH CREATEDB CREATEROLE;
```

`CREATE USER` и `createuser` по дефолту создает роль, из под которой 

Команда `CREATE ROLE` по дефолту создает роль с атрибутом NOLOGIN. Такие роли не подходят для работы с БД, их используют, например, для управлением привилегиями других ролей.



Удалить роль

Удаление ролей требует прав `CREATEROLE`, удобнее всего производить действовать из под системной роли _postgres_. Из под другой роли не получится, например из под _dts_:
```
# видим, что у dts нет прав на работу с ролями
[dts@dtsworkstation]$ psql -c "\du"
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 dts       | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test_user | Create DB                                                  | {}

# закономерно возникает ошибка
[dts@dtsworkstation]$ dropuser test_user
dropuser: error: removal of role "test_user" failed: ERROR:  permission denied to drop role
```

Т.к. требуются привелегии, из командной оболчки вызываем команду из под пользователя _postgres_:[^dropuser-cli] 
```
# To remove user joe from the default database server
$ sudo -u postgres dropuser joe
```

Программа `dropuser` реализует тот же функционал, что и команды стандарта PostgreSQL: [^drop-role] [^drop-user] 
```
# drop one role at a time
DROP ROLE [ IF EXISTS ] name [, ...]
DROP ROLE john;
# an exstension of PostgreSQL standard (суть одна)
DROP USER [ IF EXISTS ] name [, ...]
DROP USER kate;
```

---

Список БД
```
psql -l
psql -c "\list"

# аналогичная команда в psql prompt
\list
\l
```
Создать новую БД для определенного пользователя

---

Подключиться к БД под определенным пользователем: суперпользователем, владельцем БД, другим пользователем
Переподключиться из под интерфейса psql

Чтобы подключить к БД, нужно знать:
- название БД (передается с параметром `-d`);
- имя пользователя, из под которого подключаемся к БД (`-U`);
- имя хоста (`-h`) и номер порта (`-p`); эти параметры зачастую не вводятся - для локального сервера определяются по дефолту.


---

Вывести список таблиц конкретной БД
Создать таблицу с определенным названием
Посмотреть список столбцов
Добавить новый столбец с определенным типом данных
Удалить конкретный столбец с определенным типом данных
[^drop-user]: https://www.postgresql.org/docs/current/sql-dropuser.html
[^drop-role]: https://www.postgresql.org/docs/current/sql-droprole.html
[^dropuser-cli]: https://www.postgresql.org/docs/current/app-dropuser.html
