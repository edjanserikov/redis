# Домашнее задание к занятию "Индексы" - `Джансериков Эдуард`

---

### Задание 1

```
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.
```

```
Код запроса:
SELECT sum(data_length)/(sum(data_length)+sum(index_length))*100 as table_percent, 
       sum(index_length)/(sum(data_length)+sum(index_length))*100 as index_percent
  FROM INFORMATION_SCHEMA.TABLES
```

![Задание 1](https://github.com/edjanserikov/redis/blob/main/img/index_percent.PNG)

---

### Задание 2

```
Выполните explain analyze следующего запроса:
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id

- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
```

```
Запрос:
explain analyze
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
  from payment p,
       rental r,
       customer c,
       inventory i,
       film f
 where date(p.payment_date) = '2005-07-30' 
       and p.payment_date = r.rental_date
       and r.customer_id = c.customer_id
       and i.inventory_id = r.inventory_id
Это первоначальный запрос.


-> Limit: 200 row(s)  (cost=0..0 rows=0) (actual time=4957..4957 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=4957..4957 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4957..4957 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1900..4773 rows=642000 loops=1)
                -> Sort: c.customer_id, f.title  (actual time=1900..1953 rows=642000 loops=1)
                    -> Stream results  (cost=21.7e+6 rows=16e+6) (actual time=0.271..1452 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=21.7e+6 rows=16e+6) (actual time=0.268..1216 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=20.1e+6 rows=16e+6) (actual time=0.265..1089 rows=642000 loops=1)
                                -> Nested loop inner join  (cost=18.5e+6 rows=16e+6) (actual time=0.261..948 rows=642000 loops=1)
                                    -> Inner hash join (no condition)  (cost=1.58e+6 rows=15.8e+6) (actual time=0.252..50.8 rows=634000 loops=1)
                                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.65 rows=15813) (actual time=0.0233..5.02 rows=634 loops=1)
                                            -> Table scan on p  (cost=1.65 rows=15813) (actual time=0.0158..3.53 rows=16044 loops=1)
                                        -> Hash
                                            -> Covering index scan on f using idx_title  (cost=112 rows=1000) (actual time=0.0255..0.168 rows=1000 loops=1)
                                    -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1.01) (actual time=913e-6..0.00131 rows=1.01 loops=634000)
                                -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=99.6e-6..116e-6 rows=1 loops=642000)
                            -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=250e-6 rows=1) (actual time=86.8e-6..103e-6 rows=1 loops=642000)


Запрос:
explain analyze
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.tittle)
  from payment p,
       rental r,
       customer c,
       film f
 where date(p.payment_date) = '2005-07-30' 
       and p.payment_date = r.rental_date
       and r.customer_id = c.customer_id
Убрал таблицу inventory, т.к. она не участвует в результатах выборки. Чуть быстрее запрос выполнился.

-> Limit: 200 row(s)  (cost=0..0 rows=0) (actual time=4939..4939 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=4939..4939 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4939..4939 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1906..4747 rows=642000 loops=1)
                -> Sort: c.customer_id, f.title  (actual time=1906..1960 rows=642000 loops=1)
                    -> Stream results  (cost=20.1e+6 rows=16e+6) (actual time=0.294..1314 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=20.1e+6 rows=16e+6) (actual time=0.289..1088 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=18.5e+6 rows=16e+6) (actual time=0.283..949 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1.58e+6 rows=15.8e+6) (actual time=0.272..47.9 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.65 rows=15813) (actual time=0.0264..4.97 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.65 rows=15813) (actual time=0.0171..3.53 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=112 rows=1000) (actual time=0.0314..0.176 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1.01) (actual time=901e-6..0.00132 rows=1.01 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=105e-6..122e-6 rows=1 loops=642000)


Запрос:
explain analyze
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
  from payment p, 
       customer c
 where date(p.payment_date) = '2005-07-30' 
       and p.customer_id = c.customer_id
Попытался понять логику скрипта. Так понимаю нужно было вытащить по customer сумму платежей за 30.07.2005
Убрал:
- таблицу film, потому что она не нужна для выборки и даже мешает при поиске правильных результатов
- таблицу rentalб потому что в результатах ее нет и фильтров по ней тоже нет
- нашел, что по той логике, которую я понял, неправильные результаты по:
    GOMEZ JOSEPHINE
    GONZALEZ MARTHA
    HENRY PAULINE
    LARUE DARYL
    MADRIGAL RALPH
    MURRELL MANUEL
    RUNYON NATHAN
    THOMPSON DONNA


-> Limit: 200 row(s)  (cost=0..0 rows=0) (actual time=5.92..5.94 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=5.92..5.94 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=5.92..5.92 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=5.08..5.76 rows=634 loops=1)
                -> Sort: c.customer_id  (actual time=5.07..5.1 rows=634 loops=1)
                    -> Stream results  (cost=7140 rows=15813) (actual time=0.0478..4.98 rows=634 loops=1)
                        -> Nested loop inner join  (cost=7140 rows=15813) (actual time=0.0443..4.81 rows=634 loops=1)
                            -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1606 rows=15813) (actual time=0.0358..4.37 rows=634 loops=1)
                                -> Table scan on p  (cost=1606 rows=15813) (actual time=0.0283..3.38 rows=16044 loops=1)
                            -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=576e-6..592e-6 rows=1 loops=634)


Создал индекс для поля payment_date - время запроса стало дольше

-> Limit: 200 row(s)  (cost=0..0 rows=0) (actual time=6..6.02 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=6..6.01 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=6..6 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=5.19..5.86 rows=634 loops=1)
                -> Sort: c.customer_id  (actual time=5.17..5.2 rows=634 loops=1)
                    -> Stream results  (cost=7140 rows=15813) (actual time=0.0742..5.08 rows=634 loops=1)
                        -> Nested loop inner join  (cost=7140 rows=15813) (actual time=0.0701..4.91 rows=634 loops=1)
                            -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1606 rows=15813) (actual time=0.046..4.45 rows=634 loops=1)
                                -> Table scan on p  (cost=1606 rows=15813) (actual time=0.0381..3.31 rows=16044 loops=1)
                            -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=614e-6..631e-6 rows=1 loops=634)

Убрал индекс для поля ayment_date - время запроса стало меньше
-> Limit: 200 row(s)  (cost=0..0 rows=0) (actual time=5.87..5.89 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=5.86..5.88 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=5.86..5.86 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=5.02..5.73 rows=634 loops=1)
                -> Sort: c.customer_id  (actual time=5..5.03 rows=634 loops=1)
                    -> Stream results  (cost=7140 rows=15813) (actual time=0.0518..4.91 rows=634 loops=1)
                        -> Nested loop inner join  (cost=7140 rows=15813) (actual time=0.0486..4.74 rows=634 loops=1)
                            -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1606 rows=15813) (actual time=0.0403..4.28 rows=634 loops=1)
                                -> Table scan on p  (cost=1606 rows=15813) (actual time=0.0323..3.31 rows=16044 loops=1)
                            -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=605e-6..621e-6 rows=1 loops=634)

Не вижу смысла создавать индексы для других полей, потому что все необходимые индексы уже есть.

```

---

### Задание 3*

```
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

Приведите ответ в свободной форме.
```

```
Типы индексов, которые есть в PostgreSQL, но нет в MySQL:
- GiST (Generalized Search Tree): GiST-индексы позволяют создавать пользовательские типы данных и определять на них собственные операции индексирования.
- GIN (Generalized Inverted Index): GIN-индексы предназначены для индексирования векторных данных или данных, содержащих массивы или JSON-документы.
- BRIN (Block Range Index): BRIN-индексы предоставляют эффективное индексирование на основе диапазонов блоков данных, что особенно полезно для больших таблиц с упорядоченными данными.
- SP-GiST (Space-Partitioned Generalized Search Tree): SP-GiST-индексы предоставляют индексирование для специфических типов данных, которые могут быть разбиты на разделы (например, географические данные).
- RUM (Reverse Unique Index): RUM-индексы предназначены для индексирования полнотекстовых данных.
- BLOOM: Bloom-индексы используют вероятностные структуры данных для быстрого определения наличия или отсутствия значений в индексе. Они особенно полезны для больших объемов данных.
```
