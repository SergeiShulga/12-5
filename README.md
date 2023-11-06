### 12-5 Домашнее задание к занятию «Индексы» - Сергей Шульга

### Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

```
SELECT  SUM(DATA_LENGTH) , SUM(INDEX_LENGTH),
CONCAT(  ROUND ((SUM(INDEX_LENGTH) / SUM(DATA_LENGTH)) *100), ' %')  AS Отношение
FROM INFORMATION_SCHEMA.TABLES
WHERE  TABLE_SCHEMA = 'sakila';
```
![alt text](https://github.com/SergeiShulga/12-5/blob/main/img/001.png)

### Задание 2
Выполните explain analyze следующего запроса:

```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
![alt text](https://github.com/SergeiShulga/12-5/blob/main/img/002.png)
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

В запросе используется лишняя таблица film, так как данные из неё не используются. Все необходимые данные есть в таблицах payment и customer, соответственно, остальные таблицы можно исключить. Предлагается оптимизировать запрос следующим образом:

```
select  concat(c.last_name, ' ', c.first_name) as Full_name, sum(p.amount)
from payment p, rental r, customer c, inventory i
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
group by Full_name;
```
![alt text](https://github.com/SergeiShulga/12-5/blob/main/img/003.png)

```
SELECT
    CONCAT(c.last_name, ' ', c.first_name) AS Full_name,
    SUM(p.amount)
FROM
    payment p
    JOIN rental r ON p.payment_date = r.rental_date
    JOIN customer c ON r.customer_id = c.customer_id
    JOIN inventory i ON r.inventory_id = i.inventory_id
WHERE
    p.payment_date >= '2005-07-30'
    AND p.payment_date < DATEADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY Full_name;
```

```
SELECT
    CONCAT(c.last_name, ' ', c.first_name) AS Full_name,
    SUM(p.amount)
FROM
    payment p
CROSS JOIN
    customer c
WHERE
    DATE p.payment_date < DATEADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY Full_name;
```
что то это не работает
### Задание 3*
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

Приведите ответ в свободной форме.

PostgreSQL поддерживает несколько типов индексов:
- B-Tree - в MySQL так же используется;
- Hash - в MySQL используется только в механизме хранения Memory;
- GiST (R-Tree) - в MySQL так же используется;
- SP-GiST
- GIN (Inverted) - в MySQL так же используется;
- BRIN
- bloom

Для разных типов индексов применяются разные алгоритмы, ориентированные на определённые типы запросов. По умолчанию команда CREATE INDEX создаёт индексы-B-
деревья, эффективные в большинстве случаев. Выбрать другой тип можно, написав название типа индекса после ключевого слова USING. Например, создать хеш-индекс можно так:

CREATE INDEX имя ON таблица USING HASH (столбец);
