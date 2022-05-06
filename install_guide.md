## Инициализация кластера в пользовательской директории

Порядок иницилизации:
1. Установить `sudo pacman -Syu postgresql`
2. Получаем информацию о пользовательской директории, внутри которой планируем создать рездел под кластер. В моем случае, в системе единственный юзер _username_. Обращаем внимание, что доступ есть только у owner'a директории:
```
$ ls -l /home
drwx------ 25 username username 4096 May  5 10:30 username
```
3. Расширяем права пользовательской группы _username_ на чтение и выполнение файлов. Убеждаемся, что все сработало:
```
$ sudo chmod 750 /home/username
$ ls -l /home
drwxr-x--- 25 username username 4096 May  5 10:30 username
```
4. Добавляем пользователя _postgres_ в группу _username_. Убеждаемся, что все сработало:
```
$ sudo usermod -aG username postgres && groups postgres
username postgres
```
[Почему без шагов 3-4 невозможно выполнить дальнейшие шаги](install_guide_nuances.md#важность-шагов-3-4). 
5. Создаем корневую директорию для кластера в нужном месте и обязательно назначаем _postgres_ ее владельцем:
```
mkdir -p /home/username/path/to/pgsql && sudo chown postgres:postgres /home/username/path/to/pgsql
```
6. Инициализируем кластер, не забыв добавить поддиректорию `data`:
```
sudo -u postgres initdb -D /home/username/path/to/pgsql/data
```
7. Изменяем дефолтное расположение кластера с помощью drop-in файла:
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

8. Чтобы drop-in начал действовать, вызываем команду:
```
sudo systemctl reenable postgresql
```
9. Запускаем сервер (с `--now` не требуется перезагрузка):
```
sudo systemctl enable --now postgresql
systemctl status postgresql
```

На этом все.

Для остановки сервера вводим команду:
```
sudo systemctl stop postgresql
```

## После инициализации

Системный пользователь _postgres_, из под учетной записи которого производится первичная настройка СУБД, создается автоматически при установке пакета postgresql. По умолчанию, home-директорией назначается `/var/lib/postgres`. При инициализации кластера в другом месте (как сейчас) этот момент тоже нужно поправить[^psql-write-history-error-resolve].

Если сервер работает, его нужно остановить. Далее переназначаем home-директорию и проверяем результат:
```
$ sudo usermod --home /home/username/path/to/pgsql postgres
$ sudo -iu postgres
[postgres]$ pwd
/home/username/path/to/pgsql
[postgres]$ exit
```
После этого, например, сообщений об ошибках с записью истории команд в `psql`, выполненных из под учетной записи _postgres_ не будет - home-директория настроена, к ней есть доступ, и файл `.psql_history` успешно обновляется. Здесь же, традиционно, хранится `.bash_history` пользователя _postgres_.


[^drop-in-examples]: https://wiki.archlinux.org/title/Systemd#Drop-in_files
[^psql-write-history-error-resolve]: https://dba.stackexchange.com/a/83822
