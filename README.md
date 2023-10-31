### 12-5 Домашнее задание к занятию «Индексы» - Сергей Шульга

### Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

```
select sum(data_length) as SUM_Data_Length, sum(index_length) as SUM_Index_Length, sum(index_length)*100.0/sum(data_length) as Persentage_ratio
from information_schema.tables
where table_schema='sakila' and data_length is not null;
```


### Задание 2
Выполните explain analyze следующего запроса:

```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```

- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.



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
