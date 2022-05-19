## Управление ролями: требования

Создание, удаление и изменение ролей требует прав `CREATEROLE`, поэтому удобнее всего выполнять данную операцию из под системной роли _postgres_.

## Базовое управление ролями

В самом простом варианте создается роль со следующим набором дефолтных атрибутов (опция `-e` выводит на экран сопутствующие созданию роли запросы):
```
$ sudo -u postgres createuser -e john
SELECT pg_catalog.set_config('search_path', '', false);
CREATE ROLE john NOSUPERUSER NOCREATEDB NOCREATEROLE INHERIT LOGIN;
```
Благодаря атрибуту `LOGIN` с помощью роли _john_ можно авторизоваться при подключении к серверу (имя пользователя, которое указываем при создании сервера в pgadmin4, а также роль, для которой происходит запрос при подключении к этому серверу).

Список ролей можно вывести командой `psql -c "\du"
```
$ psql -U postgres -c "\du"
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 john      |                                                            | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

## Роль для вызова psql

Программные обертки для SQL-запросов:
<table>
    <tr>
        <th>SQL-запрос</th>
        <th>Обертка</th>
    </tr>
    <tr>
        <td>`CREATE USER`</td>
        <td>`createuser`</td>
    </tr>
    <tr>
        <td>`DROP USER`</td>
        <td>`dropuser`</td>
    </tr>
    <tr>
        <td>`CREATE DATABASE`</td>
        <td>`createdb`</td>
    </tr>
    <tr>
        <td>`DROP DATABASE`</td>
        <td>`dropdb`</td>
    </tr>
</table>

Чтобы чтобы каждый раз не вызывать команду с `sudo -u postgres`, нужно создать роль с тем же именем, как и у пользователя текущей сессии терминала. Т.е. если пользователь _jack_ хочет запускать из терминала команды `createdb <new_db>`, то сначала нужно создать роль _jack_ и (!) одноименную БД _jack_ - к ней будет по умолчанию происходить подключение.[^digocn-roles]

Для базового использования достаточно роли с привелегиями на создание БД (атрибутом `-d`):
```
$ sudo -u postgres createuser -de jack
SELECT pg_catalog.set_config('search_path', '', false);
CREATE ROLE jack NOSUPERUSER CREATEDB NOCREATEROLE INHERIT LOGIN;
```

Важное отличие `CREATE USER` (`createuser`) от `CREATE ROLE`: команда `CREATE ROLE` по дефолту создает роль с атрибутом NOLOGIN. Такие роли не подходят для работы с БД, их используют, например, для управлением привилегиями других ролей.


[^digocn-roles]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04
