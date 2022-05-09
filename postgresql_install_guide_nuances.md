## До установки

Директория `/var/lib/postgres` отсутствует:
```
$ ls -l / | grep var && ls -l /var | grep lib && ls -l /var/lib | grep postgres
drwxr-xr-x  12  root root  4096 May  5 15:00 var
drwxr-xr-x  40  root root  4096 May  5 15:00 lib
```
Группа _postgres_:
```
$ groups postgres
groups: unknown user postgres

```
Команда `cat /etc/group | grep postgres` ничего не возвращает

Среди установленных unit-файлов ничего связанного с postgresql не найдено - команда `systemctl list-unit-files | grep postgresql` ничего не возвращает.

Файл `/usr/lib/systemd/system/postgresql.service` также не найден.
```
ls /usr/lib/systemd/system/ | grep postgresql
```

## После установки

Директория `/var/lib/postgres` создана в процессе установки пакетов:
```
$ ls -l / | grep var && ls -l /var | grep lib && ls -l /var/lib | grep postgres
drwxr-xr-x  12  root root  4096 May  5 15:00 var
drwxr-xr-x  40  root root  4096 May  5 15:00 lib
drwxr-xr-x   3  root root  4096 May  5 16:00 postgres
```
Замечание: все таки root:root, т.е. если бы инициализировал кластер по дефолту, то пришлось бы предварительно воспользоваться `sudo chown postgres:postgres /var/lib/postgres`

Группа _postgres_ создана в процессе установки пакетов:
```
$ groups postgres
postgres

```
Появляется вывод команд:
```
$ cat /etc/group | grep postgres
postgres:x:963:

$ systemctl list-unit-files | grep postgresql
postgresql.service        disabled  disabled

$ ls /usr/lib/systemd/system/ | grep postgresql
postgresql.service
```

Вид созданного при установке unit-файла `postgresql.service`:
```
[Unit]
Description=PostgreSQL database server
After=network.target

[Service]
Type=notify
TimeoutSec=120
User=postgres
Group=postgres

Environment=PGROOT=/var/lib/postgres

SyslogIdentifier=postgres
PIDFile=/var/lib/postgres/data/postmaster.pid
RuntimeDirectory=postgresql
RuntimeDirectoryMode=755

ExecStartPre=/usr/bin/postgresql-check-db-dir ${PGROOT}/data
ExecStart=/usr/bin/postgres -D ${PGROOT}/data
ExecReload=/bin/kill -HUP ${MAINPID}
KillMode=mixed
KillSignal=SIGINT

# Due to PostgreSQL's use of shared memory, OOM killer is often overzealous in
# killing Postgres, so adjust it downward
OOMScoreAdjust=-200

# Additional security-related features
PrivateTmp=true
ProtectHome=true
ProtectSystem=full
NoNewPrivileges=true
ProtectControlGroups=true
ProtectKernelModules=true
ProtectKernelTunables=true
PrivateDevices=true
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
RestrictNamespaces=true
RestrictRealtime=true
SystemCallArchitectures=native

[Install]
WantedBy=multi-user.target
```

## Важность шагов 2-3

Дело в том, что для запуска сервера необходимо соблюдение условий:
1) системный пользователь _postgres_ должен иметь доступ к директории кластера;
2) сервис postgresql должен быть запущен пользователем _postgres_.

В соответствии с документацией[^arch-manual] для Arch-пакетов postgresql инициализация кластера должна производиться командой `sudo -u postgres /var/lib/postgres/data`. Все действительно работает, но почему? Причина в правах доступа:
```
$ ls -l / | grep var && ls -l /var | grep lib && ls -l /var/lib | grep postgres
drwxr-xr-x  12  root      root      4096  May  4  22:02  var
drwxr-xr-x  41  root      root      4096  May  5  06:51  lib
drwxr-xr-x   3  postgres  postgres  4096  May  4  21:49  postgres
       ^ ^
```
Любой пользователь из категории _Others_ (не владелец, root, и не участник группы, root) имеет доступ к содержимому директории `/var/lib`. Пользователь _postgres_ является владельцем директории `/var/lib/postgres`, что и обеспечивает в дальнейшем инициализацию кластера этим юзером.

Без доступа к содержимому директории _username_ попытка инициализации кластера приводит к ошибке:
```
$ ls -l /home
drwx------ 25 username username 4096 May  5 10:30 username

$ sudo -u postgres initdb -D /home/username/pgsql/data
could not change directory to "/home/username": Permission denied
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locales
  COLLATE:  en_US.utf8
  CTYPE:    en_US.utf8
  MESSAGES: en_US.utf8
  MONETARY: en_US.UTF-8
  NUMERIC:  en_US.UTF-8
  TIME:     en_US.UTF-8
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

initdb: error: could not access directory "/home/username/pgsql/data": Permission denied
```

После включения прав группы дальнейшая установка проходит без проблем:
```
$ ls -l /home
drwxr-x--- 25 dts dts 4096 May  5 21:53 dts

$ sudo -u postgres initdb -D /home/dts/work/software_data/pgsql/data
[sudo] password for dts: 
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locales
  COLLATE:  en_US.utf8
  CTYPE:    en_US.utf8
  MESSAGES: en_US.utf8
  MONETARY: en_US.UTF-8
  NUMERIC:  en_US.UTF-8
  TIME:     en_US.UTF-8
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /home/dts/work/software_data/pgsql/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Europe/Moscow
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    pg_ctl -D /home/dts/work/software_data/pgsql/data -l logfile start
```

Ради интереса:
```
$ systemctl status postgresql
○ postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
     Active: inactive (dead)
```
Как и ожидалось, попытка запустить сервер сейчас приводит к ошибке в логах. В журнале видно, что не удается запустить сервер в дефолтной `/var/lib/postgres`:
```
$ sudo systemctl enable --now postgresql
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.
Job for postgresql.service failed because the control process exited with error code.
See "systemctl status postgresql.service" and "journalctl -xeu postgresql.service" for details.

$ journalctl -xeu postgresql.service
    ...
    ...
May 05 21:43:18 dtsworkstation postgres[2351]: "/var/lib/postgres/data" is missing or empty. Use a >
    ...
    ...
May 05 21:43:18 dtsworkstation systemd[1]: postgresql.service: Failed with result 'exit-code'.
    ...
    ...
May 05 21:43:18 dtsworkstation systemd[1]: Failed to start PostgreSQL database server.
```

После шагов .. видим успешно запущенный сервер:
```
$ sudo systemctl reenable postgresql
$ sudo systemctl enable --now postgresql
$ systemctl status postgresql
● postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disab>
    Drop-In: /etc/systemd/system/postgresql.service.d
             └─PGROOT.conf
     Active: active (running) since Thu 2022-05-05 21:52:54 MSK; 2s ago
    Process: 2518 ExecStartPre=/usr/bin/postgresql-check-db-dir ${PGROOT}/data (code=exited, s>
   Main PID: 2523 (postgres)
      Tasks: 7 (limit: 16623)
     Memory: 15.7M
        CPU: 55ms
     CGroup: /system.slice/postgresql.service
             ├─2523 /usr/bin/postgres -D /home/dts/work/software_data/pgsql/data
             ├─2525 "postgres: checkpointer "
             ├─2526 "postgres: background writer "
             ├─2527 "postgres: walwriter "
             ├─2528 "postgres: autovacuum launcher "
             ├─2529 "postgres: stats collector "
             └─2530 "postgres: logical replication launcher "

May 05 21:52:53 dtsworkstation systemd[1]: Starting PostgreSQL database server...
May 05 21:52:53 dtsworkstation postgres[2523]: 2022-05-05 21:52:53.788 MSK [2523] LOG:  starti>
May 05 21:52:53 dtsworkstation postgres[2523]: 2022-05-05 21:52:53.788 MSK [2523] LOG:  listen>
May 05 21:52:53 dtsworkstation postgres[2523]: 2022-05-05 21:52:53.788 MSK [2523] LOG:  listen>
May 05 21:52:53 dtsworkstation postgres[2523]: 2022-05-05 21:52:53.827 MSK [2523] LOG:  listen>
May 05 21:52:53 dtsworkstation postgres[2524]: 2022-05-05 21:52:53.918 MSK [2524] LOG:  databa>
May 05 21:52:53 dtsworkstation postgres[2523]: 2022-05-05 21:52:53.976 MSK [2523] LOG:  databa>
May 05 21:52:54 dtsworkstation systemd[1]: Started PostgreSQL database server.
```

## Что происходит при reenable

```
$ sudo systemctl reenable postgresql
Removed /etc/systemd/system/multi-user.target.wants/postgresql.service.
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.
```


[^arch-manual]: https://wiki.archlinux.org/title/PostgreSQL#Initial_configuration
