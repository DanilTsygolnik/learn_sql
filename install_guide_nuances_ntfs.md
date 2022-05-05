Я пробую установить на внешний hdd, он монтируется в `/home/dts/...`, при этом `chown` с ним не работает. Однако, он монтируется с правами доступа 777, так что некритично.

Ан нет - критично:
```
...
running bootstrap script ... 2022-05-05 16:24:56.161 MSK [2994] FATAL:  data directory "/home/dts/Documents/current_work/programming/pgsql/data" has wrong ownership
2022-05-05 16:24:56.161 MSK [2994] HINT:  The server must be started by the user that owns the data directory.
child process exited with exit code 1
initdb: removing data directory "/home/dts/Documents/current_work/programming/pgsql/data"
```

Команды Linux не работают с Windows filesystems[^why-chmod-doesnt-work]

Пытался что-то решить с fstab, но на ntfs так и не удалось поменять пользователя на _postgres_. Создал рабочую директорию под ext4, но диск также внешний, с mount-точкой в пользовательской директории `/home/dts/work`. Это решило проблему, получил стандартный вывод:


[^why-chmod-doesnt-work]: https://stackoverflow.com/a/15308595
