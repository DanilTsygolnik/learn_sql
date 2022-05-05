Вид дефолтного файла `postgresql.service` (после установки):
```

```

## Важность шагов 3-5

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

## Что происходит при reenable

```
$ sudo systemctl reenable postgresql
Removed /etc/systemd/system/multi-user.target.wants/postgresql.service.
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.
```
