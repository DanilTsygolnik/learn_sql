[^createuser]: https://www.postgresql.org/docs/14/app-createuser.html
[^posgtgres-db-roles]: https://www.postgresql.org/docs/14/database-roles.html
[^sql-createrole]: https://www.postgresql.org/docs/14/sql-createrole.html
[^psql-app]: https://www.postgresql.org/docs/current/app-psql.html
[^drop-user]: https://www.postgresql.org/docs/current/sql-dropuser.html
[^drop-role]: https://www.postgresql.org/docs/current/sql-droprole.html
[^dropuser-cli]: https://www.postgresql.org/docs/current/app-dropuser.html

Базы данных MS SMS и PostgreSQL полность не совместимы.[^ms-sql-import-postgresql] 


[^ms-sql-import-postgresql]: https://ru.stackoverflow.com/questions/1020378/mdf-ldf-ms-sql-%D0%B8%D0%BC%D0%BF%D0%BE%D1%80%D1%82-%D0%B2-postgresql


Создание суперпользователя (роли):[^create_superuser]
```
# из под системной роли postgres
sudo -u postgres createuser -sidrlw <username>
# из под системной роли, одноимённой с владельцем сессии
createuser -sidrlw <username>
```

[^create_superuser]: https://stackoverflow.com/a/57997449
