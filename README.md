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

В запросе обрабатывают излишние таблицы а именно inventory, rental и film, обработка и присоединение этих таблиц не имеет смысла т.к. дальше данные не используются. Все необходимые данные есть в таблицах payment и customer, соответственно, остальные таблицы можно исключить. Предлагается оптимизировать запрос следующим образом:

```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id 
```

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
