
## Первые подвижки

Момент с которого все заработало
https://forum.manjaro.org/t/failed-to-start-postgresql-database-server/70238/5

пошел снова к мануалу, но после поисков:
- использую по мануалу строго `/var/lib/postgres`, но предварительно `sudo chown -R postgres:postgres /var/lib/postgres` по совету  https://bbs.archlinux.org/viewtopic.php?pid=1182956#p1182956
- дальше как бы по гайду: аналог `sudo -iu postgres` -> `[postgres]$ initdb -D /var/lib/postgres/data` --- командой `sudo -u postgres initdb -D /var/lib/postgres/data`
- запуск сервиса `systemctl enable --now postgresql.service`; `--now` - потому что без него нужно перезагружаться, чтобы сервис поднялся. А так, статус Active, как надо.

После этих успешных манипуляций:

1 - команда `ps aux | grep postgres` выдает

```
postgres  152742  0.0  0.1 209680 25552 ?        Ss   18:31   0:00 /usr/bin/postgres -D /var/lib/postgres/data
postgres  152744  0.0  0.0 209680  3576 ?        Ss   18:31   0:00 postgres: checkpointer 
postgres  152745  0.0  0.0 209680  3576 ?        Ss   18:31   0:00 postgres: background writer 
postgres  152746  0.0  0.0 209680  8616 ?        Ss   18:31   0:00 postgres: walwriter 
postgres  152747  0.0  0.0 210224  6252 ?        Ss   18:31   0:00 postgres: autovacuum launcher 
postgres  152748  0.0  0.0  64036  3392 ?        Ss   18:31   0:00 postgres: stats collector 
postgres  152749  0.0  0.0 210236  4476 ?        Ss   18:31   0:00 postgres: logical replication launcher
```

2 - интересная штука получается с остановкой сервера.

Как не срабатывает:
```
[dts@dtsworkstation ~]$ service postgresql stop
service: command not found

[dts@dtsworkstation ~]$ pg_ctl -D /var/lib/postgres/data stop
pg_ctl: could not open PID file "/var/lib/postgres/data/postmaster.pid": Permission denied

[dts@dtsworkstation ~]$ sudo pg_ctl -D /var/lib/postgres/data stop
pg_ctl: cannot be run as root
Please log in (using, e.g., "su") as the (unprivileged) user that will
own the server process.
```
Т.е. нужно снова из под пользователя _postgres_:
```
[dts@dtsworkstation ~]$ sudo -u postgres pg_ctl -D /var/lib/postgres/data stop
could not change directory to "/home/dts": Permission denied
waiting for server to shut down.... done
server stopped
```
Получается, процессом сервера владеет пользователь _postgres_. Вопрос: как запускаться из под _dts_ ??

### RTFM

> If you are using a pre-packaged version of PostgreSQL, it may well have a specific convention for where to place the data directory, and it may also provide a script for creating the data directory. In that case you should use that script in preference to running initdb directly. Consult the package-level documentation for details.[^install-from-pre-packages]

Установка из pre-packages, поэтому надо **строго** следовать мануалу по ним[^arch-manual]. А это значит, что если написано использовать `initdb -D /var/lib/postgres/data`, то только с ней и сработает.

> Pre-packaged versions of PostgreSQL will typically create a suitable user account automatically during package installation.[^pgsql-base-user]

В мануале Arch'a об этом четко: после установки через `sudo pacman -Syu postgresql` в ОС будет создан системный пользователь _postgres_. Непосредственно для работы с данными в СУБД нужно добавлять нового пользователя, который не будет иметь доступ к executable-файлам (и это правда так - остановить сервер из под _dts_ даже c `sudo` не вышло).

Предварительные выводы:
1) добавлять своего пользователя придется отдельно в любом случае;
2) в силу особенностей пакета Arch'a инициализация кластера привязана к `/var/lib/postgres/data`. Значит, чтобы хранить рабочие файлы в `/home`, придётся выполнить дополнительные действия[^change-default-data-dir], в т.ч. настроить юниты[^editing-provided-units] с последующим `systemclt reenable postgresql.service`[^reenable-service];
3) если директория `/var/lib/postgres` не создается во время устновки пакетов, то нужно создать через `sudo mkdir /var/lib/postgres`. Далее обязательно `sudo chown -R postgres:postgres /var/lib/postgres`[^chown-before-initdb].

## Откатиться и проверить гипотезы

Момент 1 - посмотреть, существует ли...
- директория `/var/lib/postgres`
  - [x] до установки postgresql -- нет; после - появляется, судя по всему
- группа _postgres_
  - [x] до установки -- нет; после появляется в файле `/etc/group`
- в списке `systemctl list-unit-files` строчка `postgres.service`
  - [x] до установки postgresql -- нет; после появляется, статус disabled
  - [ ] до инициализации кластера через `sudo -u postgres initdb -D /var/lib/postgres/data`
  - [ ] до вызова команды `systemctl enable --now postgresql`
- файл `/usr/lib/systemd/system/postgresql.service`
  - [x] до установки postgresql -- нет; после появляется
  - [ ] до инициализации кластера через `sudo -u postgres initdb -D /var/lib/postgres/data`
  - [ ] до вызова команды `systemctl enable --now postgresql`

### Инициализация кластера и запуск сервера
Команда `sudo -u postgres pg_ctl -D /var/lib/postgres/data stop` останавливает сервер, но `postgres.service` остается в статусе enabled (см. команду `systemctl list-unit-files`).

Последовательность работает, все по мануалу (даже с `chown` заморачиваться не пришлось):
```
sudo -u postgres initdb -D /var/lib/postgres/data
sudo systemctl enable --now postgresql
systemctl status postgresql
```
По мануалам (??) остановка командой:
```
[dts@dtsworkstation ~]$ sudo -u postgres pg_ctl -D /var/lib/postgres/data stop
could not change directory to "/home/dts": Permission denied
waiting for server to shut down.... done
server stopped
```
Но можно проще: `sudo systemctl stop postgresql`

### Работа с ролями

Как добавить роль[^create-user-how-to-adv]:
```
sudo -u postgres createuser --interactive

# wrong way
$ sudo -u postgres psql
postgres=# createuser --interactive
```

В директории `/var/lib/postgres` получается вызвать `sudo -u postgres initdb -D ...`, т.к. поддиректории доступны для чтения "другими":
```
drwxr-xr-x 12 root root  4096 May  4 22:02 var
drwxr-xr-x 41 root root  4096 May  5 06:51 lib
drwxr-xr-x  3 root root  4096 May  4 21:49 postgres
       ^ ^
```
С директорией `/home/dts` ситуация иная:
```
drwxr-xr-x  3 root     root     4096 Dec 18 18:21 home
drwx------ 25 dts      dts      4096 May  5 09:12 dts
       ^ ^
drwxr-xr-x  2 postgres postgres 4096 May  5 08:55 pgsql
```

Гипотеза:
1) расширить права доступа группы _dts_ для директории `/home/dts` до `r-x`: `sudo chmod 750 /home/dts`
2) добавить пользователя _postgres_ в группу _dts_: `sudo usermod -aG dts postgres`
3) вызвать `sudo -u postgres initdb -D /home/dts/pgsql/data`
4) создать юнит

Проверка

1) ок
2) ок
3) сработало
4) по шагам

Шаг 1
```
sudo mkdir /etc/systemd/system/postgresql.service.d
sudo chmod 755 /etc/systemd/system/postgresql.service.d
sudo vim /etc/systemd/system/postgresql.service.d/customenv.conf
```

Последняя команда открывает т.н. _drop-in_ файл, который изменит дефолтные настройки из юнита `/usr/lib/systemd/system/postgresql.service`, автоматически созданного во время установки пакетов `postgresql`. Название файла `.conf` выбрано произвольно, из контекста введённого по мануалу[^drop-in-examples] текста:
```
[Service]
Environment=PGROOT=/home/dts
PIDFile=/home/dts/data/postmaster.pid
ProtectHome=false
```

Шаг 2
```
$ sudo systemctl reenable postgresql
Removed /etc/systemd/system/multi-user.target.wants/postgresql.service.
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.
```
Не то, что я ожидал - он снова привязался к дефолтному юниту? Или это с учётом моего drop-in файла? -- Да! Так и есть.

До `reenable`:
```
$ sudo systemctl status postgresql
○ postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disab>
     Active: inactive (dead) since Thu 2022-05-05 08:57:26 MSK; 1s ago
    Process: 4653 ExecStartPre=/usr/bin/postgresql-check-db-dir ${PGROOT}/data (code=exited, s>
    Process: 4657 ExecStart=/usr/bin/postgres -D ${PGROOT}/data (code=exited, status=0/SUCCESS)
   Main PID: 4657 (code=exited, status=0/SUCCESS)
        CPU: 657ms

May 05 07:16:24 dtsworkstation postgres[4657]: 2022-05-05 07:16:24.786 MSK [4657] LOG:  databa>
May 05 07:16:24 dtsworkstation systemd[1]: Started PostgreSQL database server.
May 05 08:57:26 dtsworkstation postgres[4657]: 2022-05-05 08:57:26.411 MSK [4657] LOG:  receiv>
May 05 08:57:26 dtsworkstation systemd[1]: Stopping PostgreSQL database server...
May 05 08:57:26 dtsworkstation postgres[4657]: 2022-05-05 08:57:26.412 MSK [4657] LOG:  aborti>
May 05 08:57:26 dtsworkstation postgres[4657]: 2022-05-05 08:57:26.413 MSK [4657] LOG:  backgr>
May 05 08:57:26 dtsworkstation postgres[4660]: 2022-05-05 08:57:26.413 MSK [4660] LOG:  shutti>
May 05 08:57:26 dtsworkstation postgres[4657]: 2022-05-05 08:57:26.420 MSK [4657] LOG:  databa>
May 05 08:57:26 dtsworkstation systemd[1]: postgresql.service: Deactivated successfully.
May 05 08:57:26 dtsworkstation systemd[1]: Stopped PostgreSQL database server.
```
После `reenable`:
```
$ sudo systemctl status postgresql
○ postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disab>
    Drop-In: /etc/systemd/system/postgresql.service.d
             └─override.conf
     Active: inactive (dead) since Thu 2022-05-05 08:57:26 MSK; 1h 13min ago
   Main PID: 4657 (code=exited, status=0/SUCCESS)
        CPU: 657ms

May 05 07:16:24 dtsworkstation postgres[4657]: 2022-05-05 07:16:24.786 MSK [4657] LOG:  databa>
May 05 07:16:24 dtsworkstation systemd[1]: Started PostgreSQL database server.
May 05 08:57:26 dtsworkstation postgres[4657]: 2022-05-05 08:57:26.411 MSK [4657] LOG:  receiv>
May 05 08:57:26 dtsworkstation systemd[1]: Stopping PostgreSQL database server...
May 05 08:57:26 dtsworkstation postgres[4657]: 2022-05-05 08:57:26.412 MSK [4657] LOG:  aborti>
May 05 08:57:26 dtsworkstation postgres[4657]: 2022-05-05 08:57:26.413 MSK [4657] LOG:  backgr>
May 05 08:57:26 dtsworkstation postgres[4660]: 2022-05-05 08:57:26.413 MSK [4660] LOG:  shutti>
May 05 08:57:26 dtsworkstation postgres[4657]: 2022-05-05 08:57:26.420 MSK [4657] LOG:  databa>
May 05 08:57:26 dtsworkstation systemd[1]: postgresql.service: Deactivated successfully.
May 05 08:57:26 dtsworkstation systemd[1]: Stopped PostgreSQL database server.
```
Интересно, что название drop-in файла автоматически сменилось на `override.conf`

во-первых файл поправил
во-вторых сохранил как PGROOT.conf, удалил override.conf
в-третьих reenable
создал в /home/dts/pgsql , задал права доступа postgres:postgres
инициализировал кластер `sudo -u postgres initdb -D /home/dts/pgsql`
запустил сервер
```
$ sudo systemctl enable --now postgresql
$ systemctl status postgresql
● postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disab>
    Drop-In: /etc/systemd/system/postgresql.service.d
             └─PGROOT.conf
     Active: active (running) since Thu 2022-05-05 10:36:51 MSK; 16s ago
    Process: 8643 ExecStartPre=/usr/bin/postgresql-check-db-dir ${PGROOT}/data (code=exited, s>
   Main PID: 8647 (postgres)
      Tasks: 7 (limit: 16623)
     Memory: 15.7M
        CPU: 61ms
     CGroup: /system.slice/postgresql.service
             ├─8647 /usr/bin/postgres -D /home/dts/pgsql/data
             ├─8650 "postgres: checkpointer "
             ├─8651 "postgres: background writer "
             ├─8652 "postgres: walwriter "
             ├─8653 "postgres: autovacuum launcher "
             ├─8654 "postgres: stats collector "
             └─8655 "postgres: logical replication launcher "

May 05 10:36:51 dtsworkstation systemd[1]: Starting PostgreSQL database server...
May 05 10:36:51 dtsworkstation postgres[8647]: 2022-05-05 10:36:51.961 MSK [8647] LOG:  starti>
May 05 10:36:51 dtsworkstation postgres[8647]: 2022-05-05 10:36:51.962 MSK [8647] LOG:  listen>
May 05 10:36:51 dtsworkstation postgres[8647]: 2022-05-05 10:36:51.962 MSK [8647] LOG:  listen>
May 05 10:36:51 dtsworkstation postgres[8647]: 2022-05-05 10:36:51.962 MSK [8647] LOG:  listen>
May 05 10:36:51 dtsworkstation postgres[8649]: 2022-05-05 10:36:51.965 MSK [8649] LOG:  databa>
May 05 10:36:51 dtsworkstation postgres[8647]: 2022-05-05 10:36:51.967 MSK [8647] LOG:  databa>
May 05 10:36:51 dtsworkstation systemd[1]: Started PostgreSQL database server.
```

#### Создание юнита

После вызова `systemctl enable --now postgresql.service` создается файл:
`/usr/lib/systemd/system/postgresql.service` (AKA unit ??). Он судя по всему прописывает, что БД должна быть именно в директории `/var/lib/postgres/data`. Чтобы это пофиксить, нужно воспользоваться юнитами (в частности, когда переназначаем кластер в `/home/user/dir`).

Как это работает

шаг 1 - создать юнит
To replace the unit file `/usr/lib/systemd/system/unit`, create the file      
    --> `/etc/systemd/system/unit`       
and reenable the unit to update the symlinks[^editing-provided-units]

шаг 2 - reenable [^reenable-service]

----

Возможно понадобится:
- как правильно удалять файлы .services https://superuser.com/questions/513159/how-to-remove-systemd-services



[^change-default-data-dir]: https://wiki.archlinux.org/title/PostgreSQL#Change_default_data_directory
[^editing-provided-units]: https://wiki.archlinux.org/title/Systemd#Replacement_unit_files
[^reenable-service]: https://wiki.archlinux.org/title/Systemd#Using_units
[^arch-manual]: https://wiki.archlinux.org/title/PostgreSQL#Initial_configuration
[^install-from-pre-packages]: вообще, популярно разъясняются моменты, на которых споткнулся
    например, что нужна та самая `sudo chown -R postgres:postgres /var/lib/postgres`
    что обязательно нужно чистить `/var/lib/postgres/data`, иначе `initdb` не сработает
    и ещё несколько важных моментов (на будущее)
    https://www.postgresql.org/docs/14/creating-cluster.html
[^pgsql-base-user]: https://www.postgresql.org/docs/14/postgres-user.html
[^chown-before-initdb]: https://bbs.archlinux.org/viewtopic.php?pid=1182956#p1182956
[^create-user-how-to-adv]: https://stackoverflow.com/a/62922162
[^drop-in-examples]: https://wiki.archlinux.org/title/Systemd#Drop-in_files


доп. ссылки

[^arch-install-posgtres]: https://wiki.archlinux.org/title/PostgreSQL#Installation
[^postgres-locale]: https://www.postgresql.org/docs/current/locale.html
[^sys-locales]: https://wiki.archlinux.org/title/Locale#Generating_locales
