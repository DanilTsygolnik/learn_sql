## Установка postgresql в Manjaro

```
sudo pacman -Syu postgresql
```

Полный гайд[^postgresql_manjaro].

## Инициализация кластера в пользовательской директории

По минимуму, без пароля:
```
$ initdb -D $HOME/postgres_data -U $USER
```

С паролем:
```
$ initdb -D $HOME/postgres_data -U $USER -A md5 -W
```

## Настройки для работы сервера

В файле `$HOME/postgres_data/postgresql.conf`: [^postgresql_conf_setting_up]
1) назначаем порт, например `port = 11000` (любое число больше 10000);
2) для работы сервера прописываем **существующие** директории в строчке `unix_socket_directories = '$HOME/postgres_data/run, /tmp'`

В файл `$HOME/.bashrc` добавляем строчки: [^short_commands_setting_up]
```bash
export PGPORT=11000
export PGHOST=$HOME/postgres_data/run
```
В дальнейшем это позволит использовать короткие команды `psql`, `createdb`, не указывая порт и хост при каждом вызове.

## Работа с БД

Запуск сервера осуществляется командой:
```bash
$ pg_ctl -D $HOME/postgres_data -l $HOME/postgres_data/logfile start
```
Убеждаемся, что сервер запущен:
```bash
$ pg_ctl status -D $HOME/postgres_data
pg_ctl: server is running (PID: 7380)
```
После первого запуска создаём для пользователя одноимённую БД командой `createdb`. Если бы не добавили переменные в файл `.bashrc`, потребовалась бы полная запись:
```
createdb -p 11000 -h $HOME/postgres_data/run
```
На этом настройка закончена, и можно приступать к работе с БД.

Остановка сервера осуществляется командой:
```
pg_ctl -D $HOME/postgres_data stop
```


[^postgresql_manjaro]: https://gist.github.com/marcorichetta/af0201a74f8185626c0223836cd79cfa
[^postgresql_conf_setting_up]: https://cims.nyu.edu/webapps/content/systems/userservices/databases/PostgreSQL-cluster
[^short_commands_setting_up]: несколько подсказок
    подсказка https://askubuntu.com/a/736037
    список переменных в доках PostgreSQL https://www.postgresql.org/docs/current/libpq-envars.html
    пояснения по переменным и `.bashrc` https://stackoverflow.com/a/13046663
