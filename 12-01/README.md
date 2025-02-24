# Домашнее задание к занятию "Базы данных" - `Джансериков Эдуард`

---

### Задание 1

`
Опишите не менее семи таблиц, из которых состоит база данных:

какие данные хранятся в этих таблицах;
какой тип данных у столбцов в этих таблицах, если данные хранятся в PostgreSQL.
Приведите решение к следующему виду:

Сотрудники (

идентификатор, первичный ключ, serial,
фамилия varchar(50),
...
идентификатор структурного подразделения, внешний ключ, integer).
`

```
Сотрудник (
- ID сотрудника - идентификатор, первичный ключ, serial
- Фамилия - varchar (50)
- Имя - varchar (20)
- Отчество - varchar (50)
- Оклад - money
- Дата найма - timestamp
- ID должности - идентификатор, внешний ключ, serial 
- ID структурного подразделения - идентификатор, внешний ключ, serial 
)

Должность (
- ID должности - идентификатор, первичный ключ, serial
- Наименование - varchar (100)
)

Тип подразделения (
- ID типа подразделения - идентификатор, первичный ключ, serial
- Наименование - varchar (100)
)

Структурное подразделение (
- ID структурного подразделения - идентификатор, первичный ключ, serial
- ID тип подразделения - идентификатор, внешний ключ, serial
- Наименование - varchar (100)
)

Адрес филиала (
- ID адреса филиала - идентификатор, первичный ключ, serial
- ID региона - идентификатор, внешний ключ, serial
- ID населенного пункта - идентификатор, внешний ключ, serial
- Адрес (улица, дом) -  varchar(150)
)

Регион (
- ID региона - идентификатор, первичный ключ, serial
- Тип региона (край, область и т.п.)
- Наименование региона - varchar(30)
)

Населенный пункт (
- ID населенного пункта - идентификатор, первичный ключ, serial
- Тип населенного пункта (город, пгт, село и т.п.)
- Наименование - varchar(30)
)

Проект (
- ID проекта - идентификатор, первичный ключ, serial
- Наименование проекта - varchar(50)
- Дата начала проекта - timestamp
- Дата завершения проекта - timestamp
)

Назначение на проект (
- ID назначения - идентификатор, первичный ключ, serial
- ID проекта - идентификатор, внешний ключ, serial
- ID сотрудника - идентификатор, внешний ключ, serial
)
```

