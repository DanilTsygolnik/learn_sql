# Конспект

Чаще всего СУБД используются в так называемой **клиент-серверной архитектуре**, когда логика обработки базы данных явно отделена от прикладной логики (взаимодействие с пользователем, решение прикладных задач).

Например, мы хотим предоставить пользователю удобный графический интерфейс -- настольную программу для Windows, которая будет обращаться к базе данных, отбирать из неё различные сведения для отчётов, добавлять новые сведения, и т. д. Такая архитектура организуется следующим образом: имеется выделенный сервер обработки баз данных, который работает на собственной машине в сети и независим от местонахождения клиентских компьютеров. Этот сервер следит за сохранностью базы данных, обрабатывает запросы, обеспечивает транзакционную работу, реализует блокировку на уровне записи или даже ее отдельного поля и устойчиво функционирует при наличии миллионов клиентских обращений. Он снимает с пользовательских машин нагрузку, связанную с обработкой баз данных, разграничивает доступ пользователей и программ к данным, скрывая отдельные таблицы и их отдельные поля и ограничивая возможность их модификации и так далее.

Драйвер СУБД выполняет в этом случае функции связи прикладной программы с сервером баз данных. Клиентское приложение реагирует на действия пользователя, выполняет необходимую счетную работу, но наборы данных полностью получает от СУБД, которая обрабатывает запросы SQL на своём удаленном компьютере.

## Язык запросов SQL

Обработка данных непосредственно в самой СУБД выполняется с помощью языка структурированных запросов к базам данных **Structured Query Language (SQL)**. Он позволяет одной командой отобрать подмножества записей из одной или нескольких таблиц по заданному критерию отбора, сортировать содержимое таблиц и выполнять другие действия по фильтрации и модификации содержимого таблиц. SQL также может добавлять, модифицировать и удалять отдельные записи или целые таблицы, и т. д.

Язык SQL был разработан в IBM в середине 1970-х годов. На сегодняшний день, существует ряд стандартов SQL, но 99% всех потребностей прикладной разработки БД удовлетворяет стандартная версия SQL. Кроме того, SQL по сути не зависит от СУБД, и однократно изучив этот язык запросов, можно переносить этот навык на любые другие реляционные системы.

Обработку данных внутри СУБД осуществляется с помощью **команд** SQL.

### Структура команд

**Операторы** рекомендуется писать прописными буквами (например, `SELECT`).

Структура команд:

```sql
SELECT поле-значений FROM таблица ;
```

**Сообщения** содержат информацию о событиях в клиентской программе. Пока непонятно, как это выглядит, и что имеется в виду под "клиентской программой" (программа в СУБД, или в приложении которое связано с БД??)

Код можно комментировать следующими способами:
```sql
-- это однострочный комментарий

/* Это многострочный комментарий в 1 строку */

/*
Это комментарий
в несколько строк
*/
```

---

Примеры запросов:
```sql
SELECT * FROM Contacts; -- выбрать все поля из таблицы Contacts

SELECT CompanyName, Phone FROM Customers;
```

Не забывать ставить точку с запятой в конце команд!

### Фильтрация записей

Дополнение к командам, позволяющие выбирать  данные в соответствии с определенными критериями. Кретирии задаются условиями, например, с помощью булевых выражений.

**Булевые операторы в SQL**

|Обозначение|Оператор|
|:---:|:---:|
|`AND`|Логическое И|
|`OR`|Логическое ИЛИ|
|`NOT`|Логическое НЕ|
|`>`|Больше|
|`>=`|Больше или равно|
|`<`|Меньше|
|`<=`|Меньше или равно|
|`=`|Равно|
|`!=`|Не равно|

Общая структура:
```sql
SELECT поле-значений FROM таблица WHERE условие ;
```

---

Пусть, например, из таблицы Orders требуется отобрать список заказов, для которых значение поля Freight (плата за груз) больше значения 100, а регион доставки (ShipRegion) -- 'RJ'. Для этого надо выполнить следующую команду SQL:
```sql
SELECT * FROM Orders
WHERE (Freight > 100) AND (ShipRegion == 'RJ');
```

# Практика

Использование оператора `SELECT`, фильтрация запросов. Задания и ход работы в ![отчете](sql_task3_prac.md).