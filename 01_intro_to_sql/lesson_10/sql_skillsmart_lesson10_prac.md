# Практика, ч. 1

##### Задание 1

*Переписать решение к ![заданию 2](../lesson_8/sql_lesson8_prac.md), используя `JOIN`.*

*Текст задания: организовать эквисоединение, которое выводит цену и названия тех товаров, для которых цена за единицу (UnitPrice) в таблице Order Details меньше 20.*

Код решения:
```sql
SELECT Products.ProductName, [Order Details].UnitPrice
  FROM Products
       JOIN [Order Details]
       ON Products.ProductID = [Order Details].ProductID
          AND [Order Details].UnitPrice < 20
 ORDER BY [Order Details].UnitPrice;
```

---

##### Задание 2

*Имеется запрос:*
```SQL
SELECT Orders.Freight, Customers.CompanyName
  FROM Orders
       INNER JOIN Customers
       ON Orders.CustomerID = Customers.CustomerID
 ORDER BY Freight;
```

*Требуется проверить этот запрос с вариантом `FULL JOIN` -- за счёт чего выдача получилась объёмнее?*

**Ответ**: за счет записей с названиями компаний, от которых не поступал заказ. Это записи, которые не подошли под условие отбора, поэтому поля, соответствующие другой таблице, заполняются значением NULL.

Таких компаний две:

<img src="les10_task2.png" width=350 />

---

##### Задание 3

*Как с помощью предложения `WHERE` превратить запрос `CROSS JOIN` в `INNER JOIN`?*

Т.к. операция `INNER JOIN` возвращает пересечение множеств, а `CROSS JOIN` - набор всевозможных комбинаций элементов из этих множеств, с помощью `WHERE` нужно задать условие, по которому будут выбраны только нужные записи. На ум сразу приходят эквисоединения, рассмотренные в ![занятии 8](../lesson_8/sql_lesson8_notes.md).

Т.е. нужно добавить фильтрацию:
```sql
WHERE table1.primary_key = table2.foreign_key
```

---

##### Задание 4

*Переписать следующий запрос в `INNER JOIN`*:

```sql
SELECT Products.ProductName, [Order Details].UnitPrice
  FROM Products
       CROSS JOIN [Order Details]
 WHERE Products.ProductID = [Order Details].ProductID
```

Код решения:
```sql
SELECT Products.ProductName, [Order Details].UnitPrice
  FROM Products
       JOIN [Order Details]
       ON Products.ProductID = [Order Details].ProductID;
```

Результаты двух запросов идентичны:

<img src="les10_task4.png" width=400 />

# Практика, ч. 2

##### Задание 1

*Отобрать с помощью `LEFT JOIN` все записи из таблицы Customers, для которых FK-ключ таблицы Orders пустой.*

Код решения:
```sql
SELECT Customers.*
  FROM Customers
       LEFT JOIN Orders
       ON Customers.CustomerID = Orders.CustomerID
 WHERE Orders.OrderID IS NULL;
```

---

##### Задание 2

*Вывести конкретную информацию по всем пользователям Customers и поставщикам Suppliers -- имя контактной персоны, город и страну, а также идентификацию группы (пользователь или поставщик).*

Необходимые поля:
- имя контактной персоны -- [ContactName]
- город -- [City]
- страна -- [Country]
- дополнительно указать идентификацию группы:
    - `'Customer' AS Type`
    - `'Supplier' AS Type`

Код решения:
```sql
SELECT 'Customer' AS Type, ContactName, City, Country
  FROM Customers

 UNION

SELECT 'Supplier' AS Type, ContactName, City, Country
  FROM Suppliers
 ORDER BY Type, ContactName;
```

---
