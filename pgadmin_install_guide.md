## Установка через pip

По офф. мануалу[^install-pip-manual]:
```
$ sudo mkdir /home/user/path/pgadmin
$ sudo chown $USER /home/user/path/pgadmin
$ python3 -m venv /home/user/path/pgadmin
$ source /home/user/path/pgadmin/bin/activate
(pgadmin4) $ pip install pgadmin4
```
Т.к. выбрана кастомная директория, нужно добавить пользовательских настроек поверх файла `config.py`[^pgadmin-config-man]. Предварительно создаем дополнительные директории::
```
$ cd /home/username/path/pgadmin && mkdir sessions storage
```
В новом текстовом файле указываем фактические пути, которые нужны для работы pgadmin[^digocn-pgadmin-conf-local]:
```
LOG_FILE = '/home/username/path/pgadmin/pgadmin4.log'
SQLITE_PATH = '/home/username/path/pgadmin/pgadmin4.db'
SESSION_DB_PATH = '/home/username/path/pgadmin/sessions'
STORAGE_DIR = '/home/username/path/pgadmin/storage'
SERVER_MODE = True
```
Сохраняем файл, как `config_local.py` и помещаем в директорию виртуального окружения, где хранятся файлы пакета pgadmin4:       
`/home/user/pgadmin_environment_dir/lib/python3.10/site-packages/pgadmin4`.

В моем случае расположение получилось такое `/home/username/path/pgadmin/lib/python3.10/site-packages/pgadmin4/config_local.py`.

## Первичная настройка

При первом запуске `pgadmin4` вводим любые email и пароль, их же вводим далее в веб-браузере для входа в UI СУБД. Вообще, первичную настройку делал по гайду[^pgadmin-init-setup-guide].

Перед запуском `pgadmin4` я не стал создавать для _postgres_ пароль (раздел Prerequisites, step 4). Так что при создании первого сервера я не стал задавать пароль _postgres_, как в гайде, сразу перешел к созданию сервера. Дальше следовал по шагам, но т.к. не задавал никаких паролей, то и не вводил в это поле ничего, и все работает. Даже если создать новый сервер (ПКМ Servers > Register > Server) и указать пароль, то в дальнейшем подключиться получится и без него - во всплывающей форме оставляем пустое поле и жмем "ОК".

<img srs="/img/create_another_server.png">

<img src="/img/pgadmin_connect_after_logout.png">


[^install-pip-manual]: https://www.pgadmin.org/download/pgadmin-4-python/
[^digocn-pgadmin-conf-local]: https://www.digitalocean.com/community/tutorials/how-to-install-configure-pgadmin4-server-mode
[^pgadmin-init-setup-guide]: https://linuxhint.com/install-pgadmin4-manjaro-linux/
[^pgadmin-config-man]: https://www.pgadmin.org/docs/pgadmin4/6.8/config_py.html
