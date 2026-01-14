# Домашнее задание к занятию "`SQL. Часть 2`" - `Евгений Головенко`

### Задание 1

Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию: 
- фамилия и имя сотрудника из этого магазина;
- город нахождения магазина;
- количество пользователей, закреплённых в этом магазине.

### Решение 1

Единый запрос может быть таким:

```
SELECT
    CONCAT(s.last_name, ' ', s.first_name) AS staff_name,
    c.city,
    COUNT(cu.customer_id) AS customer_count
FROM store st
JOIN staff s
    ON s.store_id = st.store_id
JOIN customer cu
    ON cu.store_id = st.store_id
JOIN address a
    ON st.address_id = a.address_id
JOIN city c
    ON a.city_id = c.city_id
GROUP BY
    s.staff_id,
    c.city
HAVING COUNT(cu.customer_id) > 300;
```

Результат запроса:

<img width="544" height="495" alt="sakila_12-4-1" src="https://github.com/user-attachments/assets/591274e3-8416-4062-b775-e32ce504f32d" />

Собственно, что этот запрос делает:

1. Собирает интересующие нас данные по таблицам (JOIN):
- магазин -> сотрудники,
- магазин -> пользователи,
- магазин -> адрес -> город.

2. Группирует данные по:
- конкретному сотруднику (staff_id),
- по городу.

3. Считает пользователей по каждому магазину:
- COUNT(cu.customer_id)

4. Фильтрует по условию (> 300):
- HAVING COUNT(cu.customer_id) > 300

---

### Задание 2

Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

### Решение 2

Находим среднюю продолжительность всех фильмов и считаем, сколько фильмов длиннее этого среднего значения.

```
SELECT
    COUNT(*) AS film_count
FROM film
WHERE length > (
    SELECT AVG(length)
    FROM film
);
```
<img width="344" height="308" alt="sakila_12-4-2" src="https://github.com/user-attachments/assets/4cc5f0b2-239e-4568-b543-92cb0f4255b3" />

---

### Задание 3

Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

### Решение 3

Используются таблицы

```
payment -|
         |- amount
         |- payment_date
         |- rental_id

rental - |
         |- rental_id (для подсчета аренд)
```

Запрос может быть таким:

```
SELECT
    DATE_FORMAT(p.payment_date, '%Y-%m') AS payment_month,
    SUM(p.amount) AS total_amount,
    COUNT(DISTINCT r.rental_id) AS rental_count
FROM payment p
JOIN rental r
    ON r.rental_id = p.rental_id
GROUP BY
    DATE_FORMAT(p.payment_date, '%Y-%m')
ORDER BY
    total_amount DESC
LIMIT 1;
```
Что этот запрос должен сделать:
- собирает данные по таблицам `payment` и `rental`,
- группирует их по по году и месяцу, чтоб месяца разных лет не смешались,
- агрегирует общие суммы платежей и аренд за месяц,
- выбирает месяц с максимальной суммой платежей.

Результат запроса:

<img width="613" height="392" alt="sakila_12-4-3" src="https://github.com/user-attachments/assets/b000d38d-c8b0-489a-9c96-d05feea70f1e" />

---

### Задание 4*

Посчитайте количество продаж, выполненных каждым продавцом. Добавьте вычисляемую колонку «Премия». Если количество продаж превышает 8000, то значение в колонке будет «Да», иначе должно быть значение «Нет».

### Решение 4*

Используются таблицы:

```
staff   —|
         | - staff_id
         | - first_name, last_name

payment —|
         | - staff_id
         | - payment_id
```

Запрос может быть таким:

```
SELECT
    s.staff_id,
    CONCAT(s.last_name, ' ', s.first_name) AS staff_name,
    COUNT(p.payment_id) AS sales_count,
    CASE
        WHEN COUNT(p.payment_id) > 8000 THEN 'Да'
        ELSE 'Нет'
    END AS 'Премия'
FROM staff s
LEFT JOIN payment p
    ON p.staff_id = s.staff_id
GROUP BY
    s.staff_id,
    s.last_name,
    s.first_name;
```

Результат запроса:

<img width="573" height="496" alt="sakila_12-4-4" src="https://github.com/user-attachments/assets/26ab3f76-0b55-4d64-a3bb-83f14d829279" />

Логика запроса:

- используется LEFT JOIN, чтобы показать всех продавцов, даже если у кого-то нет продаж,
- считается количество продаж у каждого продавца (COUNT игнорирует NULL, что приемлемо при LEFT JOIN),
- группирую по продавцу,
- рассчитывается, кто заработал премии через описание условия

```
    CASE
        WHEN COUNT(p.payment_id) > 8000 THEN 'Да'
        ELSE 'Нет'
    END AS 'Премия'
```

---

### Задание 5*

Найдите фильмы, которые ни разу не брали в аренду.

### Решение 5*

Связи таблиц выглядят так:

```
film -> inventory -> rental
```

Фильм может быть в `inventory`. Если по `inventory_id` нет записей в `rental`, значит фильм не брали.

Запрос можно реализовать через `LEFT JOIN`. Он сохранит все фильмы, даже если аренды не было.

`r.rental_id = NULL`

И потом выбирает только такие фильмы.

Запрос может быть таким:

```
select
	i.inventory_id,
    i.film_id,
    f.title,
    r.rental_id  
FROM inventory i
LEFT JOIN rental r
    ON r.inventory_id = i.inventory_id
left join film f 
	on f.film_id = i.film_id 
WHERE r.rental_id IS null;
```

Результат запроса:

<img width="603" height="346" alt="sakila_12-4-5" src="https://github.com/user-attachments/assets/75269ba3-3acf-4d10-875c-87fbe24a2101" />

```
inventory_id|film_id|title           |rental_id|
------------+-------+----------------+---------+
           5|      1|ACADEMY DINOSAUR|         |
```
---
