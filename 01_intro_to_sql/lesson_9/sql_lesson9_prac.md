# Практика

##### Задание 1

*Найти все пары из разных заказчиков (Customers), для которых не задан регион (поле Region).*

```sql
SELECT t1.CompanyName, t2.CompanyName,
       t1.Region AS comp1_region,
       t2.Region AS comp2_region
  FROM Customers t1, Customers t2
 WHERE t1.Region IS NULL AND t2.Region IS NULL
   AND t1.CompanyName <> t2.CompanyName;
```

---

##### Задание 2

*Найти вложенным запросом список заказов (Orders), в котором у заказчиков (Customers) регион не пуст (поле Region).*

Здесь нужно сперва отобрать все CustomerID из Customers, у которых Region не NULL. Затем с полученной выборкой сверять записи из Orders: если Orders.CustomerID есть в выборке, добавляем в результаты итогового запроса.

```sql
SELECT * FROM Orders
 WHERE Orders.CustomerID = 
       ANY (SELECT CustomerID FROM Customers
             WHERE Region IS NOT NULL);
```

---

##### Задание 3

*Немного условный, но показательный пример. Найти все заказы (таблица Orders), цена за доставку товара которых (Freight) превышает цену любого товара (поле UnitPrice, таблица Products).*

```sql
SELECT * FROM Orders
 WHERE Orders.Freight > ANY (SELECT UnitPrice FROM Products);
```

---