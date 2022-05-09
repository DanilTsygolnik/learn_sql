## Установка postgresql в Manjaro

```
sudo pacman -Syu postgresql
```

В процессе установки создается системный пользователь _postgres_.

## Инициализация кластера в пользовательской директории

В документации PostgreSQL говорится, что кластер инициализировать можно в любой нужно директории. Для этого лишь требуется обеспечить доступ пользователя _postgres_ (который далее вызывает `initdb`) к соответствующему каталогу. Однако, есть оговорка:

> If you are using a pre-packaged version of PostgreSQL, it may well have a specific convention for where to place the data directory, and it may also provide a script for creating the data directory. In that case you should use that script in preference to running initdb directly. Consult the package-level documentation for details.[^postgres-init-cluster-off]

Это как раз наш случай - установили из пакетов на предыдущем шаге. Однако в офф. гайда Manjaro явно не написано "устанавливать строго в `/var/lib/postgres`, иначе не заработает" - это я уже проверил метом проб, и дальнейший текст пошагово описывает способ данное ограничение обойти. В ходе разбирательств наткнулся также на возможные альтернативы: вариант 1[^custom-dir-alternative1], вариант 2[^custom-dir-alternative2] (не проверял).


1. Получаем информацию о пользовательской директории, внутри которой планируем создать рездел под кластер. В моем случае, в системе единственный юзер _username_. Обращаем внимание, что доступ есть только у owner'a директории:
```
$ ls -l /home
drwx------ 25 username username 4096 May  5 10:30 username
```
2. Расширяем права пользовательской группы _username_ на чтение и выполнение файлов. Убеждаемся, что все сработало:
```
$ sudo chmod 750 /home/username
$ ls -l /home
drwxr-x--- 25 username username 4096 May  5 10:30 username
```
3. Добавляем пользователя _postgres_ в группу _username_. Убеждаемся, что все сработало:
```
$ sudo usermod -aG username postgres && groups postgres
username postgres
```
[Почему без шагов 2-3 невозможно выполнить дальнейшие шаги](postgresql_install_guide_nuances.md#важность-шагов-2-3).

4. Создаем корневую директорию для кластера в нужном месте и обязательно назначаем _postgres_ ее владельцем:
```
mkdir -p /home/username/path/to/pgsql && sudo chown postgres:postgres /home/username/path/to/pgsql
```
5. Инициализируем кластер, не забыв добавить поддиректорию `data`:
```
sudo -u postgres initdb -D /home/username/path/to/pgsql/data
```
6. Изменяем дефолтное расположение кластера с помощью drop-in файла:
```
sudo mkdir /etc/systemd/system/postgresql.service.d/
sudo touch /etc/systemd/system/postgresql.service.d/PGROOT.conf
```
Созданный drop-in файл изменит дефолтные настройки из юнита `/usr/lib/systemd/system/postgresql.service`, автоматически созданного во время установки пакетов `postgresql`[^drop-in-examples]. В файл необходимо добавить следующие строки (username заменить на нужное):
```
[Service]
Environment=PGROOT=/home/username/path/to/pgsql
PIDFile=/home/username/path/to/pgsql/data/postmaster.pid
ProtectHome=false
```
Сохраняем файл и переходим к следующему шагу.

7. Чтобы drop-in начал действовать, вызываем команду:
```
sudo systemctl reenable postgresql
```
8. Запускаем сервер (с `--now` не требуется перезагрузка):
```
sudo systemctl enable --now postgresql
systemctl status postgresql
```

Этап инициализации кластера можно считать завершенным. Для остановки сервера вводим команду:
```
sudo systemctl stop postgresql
```

## После инициализации

Системный пользователь _postgres_, из под учетной записи которого производится первичная настройка СУБД, создается автоматически при установке пакета postgresql. По умолчанию, home-директорией назначается `/var/lib/postgres`. При инициализации кластера в другом месте (как сейчас) этот момент тоже нужно поправить[^psql-write-history-error-resolve].

Если сервер работает, его нужно остановить. Ключевое действие здесь - переназначить home-директорию:
```
$ sudo usermod --home /home/username/path/to/pgsql postgres
```
Проверяем результат:
```
$ sudo -iu postgres
[postgres]$ pwd
/home/username/path/to/pgsql
[postgres]$ exit
```
После этого, например, сообщений об ошибках с записью истории команд в `psql`, выполненных из под учетной записи _postgres_ не будет - home-директория настроена, к ней есть доступ, и файл `.psql_history` успешно обновляется. Здесь же, традиционно, хранится `.bash_history` пользователя _postgres_.


[^drop-in-examples]: https://wiki.archlinux.org/title/Systemd#Drop-in_files
[^psql-write-history-error-resolve]: https://dba.stackexchange.com/a/83822
[^postgres-init-cluster-off]:[ссылка на мануал](https://www.postgresql.org/docs/14/creating-cluster.html) 
[^custom-dir-alternative1]: без доступа к `/var/lib/postgres` (т.е. из под юзера), без танцев с бубнами вокруг прав доступа --[ссылка](https://cims.nyu.edu/webapps/content/systems/userservices/databases/PostgreSQL-cluster) 
[^custom-dir-alternative2]: перенести кластер postgresql из `/var/lib/postgres` в пользовательскую директорию --[ссылка](https://www.digitalocean.com/community/tutorials/how-to-move-a-postgresql-data-directory-to-a-new-location-on-ubuntu-18-04) 
