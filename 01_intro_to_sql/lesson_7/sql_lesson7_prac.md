# Практика

---

##### Задание 1

*Выведите вычислимое поле таблицы Order Details, в котором укажите значение поля Discount (скидка), выраженное в процентах.*

```sql
SELECT OrderID, Discount * 100 AS Discount
  FROM [Order Details];
```

---

##### Задание 2

*Выведите все поля таблицы Order Details, для которых количество единиц товара на складе больше 40. Поле UnitsInStock (складские запасы) извлеките вложенным запросом через FK-поле ProductId, ссылающееся на таблицу Products.*

```sql
SELECT *
  FROM [Order Details]
 WHERE ProductID IN (SELECT ProductID
                       FROM Products
                      WHERE UnitsInStock > 40);
```

---

##### Задание 3

*Расширьте предыдущий запрос проверкой, чтобы стоимость товара (поле Freight таблицы Orders) было не менее 50. Связь с таблицей Orders происходит через FK-поле OrderID.*

```sql
SELECT *
  FROM [Order Details]
 WHERE ProductID IN (SELECT ProductID
                       FROM Products
                      WHERE UnitsInStock > 40)
   AND OrderID IN (SELECT OrderID
                     FROM Orders
                    WHERE Freight >= 50);
```

---

##### Задание 4

Решая последнюю задачу в предыдущем уроке, я задумался, можно ли каким-то образом работать с  данными из результа запроса. Такую возможность как раз и открывают вложенные запросы.

Задача: *Вывести категории (CategoryName, табл. Categories), средняя цена товаров из которых (UnitPrice, табл. Products) больше 30.  Данные отсортировать по возрастанию цены.*

Для начала, перепишем готовое решение, используя вложенные запросы. Позже это упростит доп. обработку данных в соответствии с заданием.

**Важно**: обязательно добавляем *псевдоним* (alias) ко вложенному запросу, т.к. иначе не получится ссылаться на поля полученной выборки. 

```sql
SELECT *
  FROM (SELECT CategoryName, AVG(UnitPrice) AS avg_price
          FROM Products "prod", Categories "cat"
         WHERE cat.CategoryID = prod.CategoryID
         GROUP BY CategoryName) all_categories -- псевдоним выборки
 ORDER BY avg_price;
```

Остается исключить категории, у которых  средняя цена меньше 30.

```sql
SELECT *
  FROM (SELECT CategoryName, AVG(UnitPrice) AS avg_price
          FROM Products "prod", Categories "cat"
         WHERE cat.CategoryID = prod.CategoryID
         GROUP BY CategoryName) all_categories -- псевдоним выборки
 WHERE avg_price >= 30 -- требование в текущем задании
 ORDER BY avg_price;
```

---