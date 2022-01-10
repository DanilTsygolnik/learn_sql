# Практика, ч. 1

##### Задание 1

*Добавьте нового пользователя в таблицу Employees.*

Исходная таблица Employees:

<img src="les11_task1_1.png" />

Код запроса:
```sql
INSERT INTO Employees
       ([EmployeeID],
        [LastName],
        [FirstName],
        [Title],
        [TitleOfCourtesy],
        [BirthDate],
        [HireDate],
        [Address],
        [City],
        [Region],
        [PostalCode],
        [Country],
        [ReportsTo])
VALUES (10,
        'Barton',
        'Issy',
        'Sales Representative',
        'Ms.',
        '1970-12-14 00:00:00.000',
        '1992-09-18 00:00:00.000',
        '140th Ave NE',
        'Redmond',
        'WA',
        '98052',
        'USA',
        2) ;
```

Таблица Employees после выполнения запроса:

<img src="les11_task1_2.png" />

---

##### Задание 2

*Связать добавленного пользователя с какой-либо территорией с помощью таблицы EmployeeTerritories (многие-ко-многим).*

Исходная таблица EmployeeTerritories:

<img src="les11_task2_1.png" />

Код запроса:
```sql
INSERT INTO EmployeeTerritories (EmployeeID, TerritoryID)
VALUES (12, 95054),
       (12, 85014),
	   (12, 94105);
```

Таблица EmployeeTerritories после выполнения запроса:

<img src="les11_task2_2.png" />

---

##### Задание 3

*Попробуйте добавить новую запись в таблицу заказов Orders. Возникнут ли какие-либо конфликты?* 

Исходная таблица Orders (фрагмент):

<img src="les11_task3_1.png" />

В таблице много полей, через некоторые осуществляется связь с другими таблицами. При составлении запроса нужно:
- вручную указать значения для полей, в которых может возникнуть конфликт со значениями "по умолчанию";
- учесть связи таблиц и в полях FK указать корректные (существующие) идентификаторы.

Определить ключевые поля поможет раздел Columns, а названия таблиц, хранящих корректные значениями ключей - раздел Keys:

<img src="les11_task3_2.png" />

Код запроса:
```sql
INSERT INTO Orders (CustomerID, EmployeeID, ShipVia)
VALUES ('WARTH', 12, 3);
```

Поле OrderID не указано, т.к. значение данного идентификатора (PK) создается автоматически.

Конфликтов не возникло, в таблицу Orders добавлена новая запись:

<img src="les11_task3_3.png" />

---
