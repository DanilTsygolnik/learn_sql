В предыдущих занятиях рассматривались варианты соединения таблиц либо по умолчанию, либо через эквисоединения. 

В SQL имеется специальный оператор `JOIN` (в нескольких модификациях), который явно задаёт фактически все ключевые виды соединений двух таблиц на основе теории множеств в рамках школьного курса.

Базовый синтаксис `JOIN`:
```sql
FROM Table1 JOIN Table2 ON условие
```

Оператор `JOIN` может уточняться дополнительными модификаторами-префиксами.     
Например, это может быть `AS`:
```sql
FROM Table1 
JOIN Table2 AS t2
  ON условие
```


## Операция INNER JOIN

В сущности, результатом операции является пересечение двух множеств.

Мы указываем условие связи, например, по FK, и получаем только те записи из обеих таблиц, которые отвечают этому требованию.

Простой пример -- свяжем через `JOIN` таблицу товаров (Products) и категорий товаров (Categories), чтобы показать название товара и его категорию:
```sql
SELECT Products.ProductName, Categories.CategoryName
  FROM Products
 INNER JOIN Categories
    ON Products.CategoryID = Categories.CategoryID;
```

`INNER JOIN` -- это `JOIN` в SQL по умолчанию, его можно записывать просто как `JOIN` (без `INNER`).
```sql
SELECT Products.ProductName, Categories.CategoryName
  FROM Products
  JOIN Categories
    ON Products.CategoryID = Categories.CategoryID;
```

## Операция FULL JOIN

Операция возвращает все записи, входящие в обе таблицы. По сути, это операция объединения множеств.

Пример:
```sql
SELECT Orders.Freight, Customers.CompanyName
  FROM Orders
  FULL JOIN Customers
    ON Orders.CustomerID = Customers.CustomerID
 ORDER BY Freight;
```

Мы соединили две таблицы по FK идентификатора пользователя.

В чём различие такой записи от варианта, если бы мы использовали `INNER JOIN`?
```sql
INNER JOIN Customers
```

Технически, мы получим несколько дополнительных записей, у которых в поле Freight записан NULL. Это записи, которые не подошли под условие отбора, поэтому поля, соответствующие другой таблице, заполняются значением NULL.

## Операция CROSS JOIN

Это декартово произведение двух таблиц (все ко всем), аналогичное обычному соединению:
```sql
SELECT Employees.FirstName, Employees.LastName, Orders.Freight
  FROM Employees, Orders;
```

Вариант с `JOIN`:
```sql
SELECT Employees.FirstName, Employees.LastName, Orders.Freight
  FROM Employees
 CROSS JOIN Orders;
```

Результаты выдачи будут эквивалентны.

Рекомендуется всегда использовать запись с `JOIN`, чтобы не возникало неоднозначностей в трактовке работы запроса.

## Практика

Закрепление теории и ход работы в [отчете](sql_skillsmart_lesson10_prac.md).

---
