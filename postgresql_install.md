Сперва на OS
```
sudo pacman -Syu postgresql
```

Затем в любой удобной директории[^creating-db-cluster] инициализировать кластер. Это директория, в которой будут храниться все вновь созданные базы данных.

Популярные варианты расположения кластера:
- `/usr/local/pgsql/data`
- `/var/lib/pgsql/data`


Мысли про себя: по идее, имеет смысл выделить где-то внутри `/home` - чтобы Timeshift не захватывал изменения (для рабочих документов бэкапы делаю отдельно). В любом случае, не переживать насчёт "Что, если установлю не туда?" - директорию кластера можно без проблем изменить[^change-cluster-dir] (похожие действия потребуются и при апгрейде postgresql[^postgresql-upgrade]). 


Инициализация производится с помощью команды `initdb`. Если указанной директории пока нет, `initdb` попытается создать её[^creating-db-cluster].

Инструкции по докам [не сработали](initdb_sudo_issue.md). 

Из сообщений об ошибках было понятно, что проблемы с доступом пользователя _postgres_ к директориям инициализации. Догадка подтвердилась[^use-chmod-advice], но на рабочее решение нашёл в другой ветке:
1) Создал директорию `/home/dts/pgsql/data`, соответственно права доступа user:group - dts:dts (аккаунт, из под которого обычно работаю);
2) вызвал команду `initdb -D /home/dts/pgsql/data` (т.е. из под своего аккаунта).


Последний шаг - запуск сервиса postgresql[^basic-systemctl-usage]:
```
# enable service
sudo systemctl enable postgresql.service
# check if postgresql is running
sudo systemctl status postgresql.service
```

Как было у меня
```
systemctl enable postgresql.service

Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.

systemctl status postgresql.service

○ postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disab>
     Active: inactive (dead)
```


> Tip: If you create a PostgreSQL role/user with the same name as your Linux username, it allows you to access the PostgreSQL database shell without having to specify a user to login (which makes it quite convenient). [^dp-create-user]




---

Как остановить sql-сервер[^stop-sql-server].


[^creating-db-cluster]: https://www.postgresql.org/docs/14/creating-cluster.html
[^change-cluster-dir]:
    arch wiki https://wiki.archlinux.org/title/PostgreSQL#Change_default_data_directory
    совет https://stackoverflow.com/a/6993148
[^postgresql-upgrade]: https://wiki.archlinux.org/title/PostgreSQL#Upgrading_PostgreSQL
[^stop-sql-server]: https://www.postgresql.org/docs/14/server-shutdown.html
[^use-chmod-advice]: https://stackoverflow.com/a/40158939
[^dp-create-user]: https://wiki.archlinux.org/title/PostgreSQL#Create_your_first_database/user
[^basic-systemctl-usage]: https://wiki.archlinux.org/title/Systemd#Using_units

