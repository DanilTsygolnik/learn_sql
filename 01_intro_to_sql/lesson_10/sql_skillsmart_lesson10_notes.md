# Конспект, ч. 1

В предыдущих занятиях рассматривались варианты соединения таблиц либо по умолчанию, либо через эквисоединения. 

В SQL имеется специальный оператор `JOIN` (в нескольких модификациях), который явно задаёт фактически все ключевые виды соединений двух таблиц на основе теории множеств в рамках школьного курса.

Базовый синтаксис `JOIN`:
```sql
FROM Table1 JOIN Table2 ON условие
```

Оператор `JOIN` может уточняться дополнительными модификаторами-префиксами.     

Пример структуры запроса:
```sql
SELECT *
  FROM table AS t0
       JOIN table1 AS t1
       ON условие_1
          AND условие_2;

       JOIN table2 AS t2
       ON условие_3
          AND условие_4;
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

# Конспект, ч. 2

## Операция LEFT JOIN / LEFT OUTER JOIN

Это частный случай формы `FULL JOIN`, когда мы получаем данные из первой ("левой") таблицы вместе с данными из "правой" таблицы, которые пересекаются с первой. Остальные данные правой таблицы исключаются.
```sql
SELECT Customers.CompanyName, Orders.OrderID
  FROM Customers 
       LEFT JOIN Orders
       ON Customers.CustomerID = Orders.CustomerID 
 ORDER BY Customers.CompanyName; 
```

Мы получим все записи из таблицы пользователей Customers, даже если им не найдено сопоставлений из таблицы заказов Orders.

## Операция RIGHT JOIN

Схема аналогична предыдущей, только в качестве базовой мы берём "правую" таблицу:
```sql
SELECT Orders.OrderID, Employees.LastName, Employees.FirstName
  FROM Orders 
       RIGHT JOIN Employees 
       ON Orders.EmployeeID = Employees.EmployeeID 
 ORDER BY Orders.OrderID; 
```

Мы получим все записи из таблицы исполнителей Employees, даже если для них нет соответствия в таблице заказов Orders.

## Операция SELF JOIN

Это вариант комбинирования таблицы с самой собой. Похожую технику мы уже рассматривали ранее. Например, мы хотим получить все пары пользователей из одного города:
```sql
SELECT A.CompanyName AS CustomerName1, 
       B.CompanyName AS CustomerName2, 
       A.City 
  FROM Customers A, Customers B 
 WHERE A.CustomerID <> B.CustomerID
   AND A.City = B.City 
 ORDER BY A.City;
```

## Операция UNION

С помощью оператора UNION мы можем скомбинировать результаты двух SQL-запросов по полям, которые в общем случае не являются ключевыми, но должны иметь одинаковые типы. По умолчанию `UNION` исключает дублирующиеся значения, а если они требуются, надо использовать `UNION ALL`.

Например, мы хотим отобрать все города из двух таблиц -- пользователей Customers и поставщиков Suppliers:
```sql
SELECT City FROM Customers 

 UNION ALL 

SELECT City FROM Suppliers 
 ORDER BY City; 
```

В каждом из запросов `SELECT` можно использовать условие отбора:
```sql
SELECT City, Country 
  FROM Customers 
 WHERE Country='USA' 

 UNION 

SELECT City, Country 
  FROM Suppliers 
 WHERE Country='USA' 
 ORDER BY City; 
```

Чтобы разделить записи по таблицам, можно в каждый `SELECT` включить собственное идентификационное поле:
```sql
SELECT 'Customer' As Type, City, Country FROM Customers 
 WHERE Country='USA' 

 UNION 

SELECT 'Supplier' As Type, City, Country FROM Suppliers 
 WHERE Country='USA' 
 ORDER BY City;
```

# Практика

Упражнения по `INNER JOIN`, `FULL JOIN`, `CROSS JOIN` -- [отчет](sql_skillsmart_lesson10_prac.md#практика-ч-1), раздел "Практика, ч. 1".

Упражнения по `LEFT JOIN`, `UNION` -- [отчет](sql_skillsmart_lesson10_prac.md#практика-ч-2), раздел "Практика, ч. 2".

---
