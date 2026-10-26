# Лабораторная работа №4 — пример выполнения

**Тема:** Анализ набора данных с помощью SQL  
**Предметная область примера:** сервис аренды велосипедов  
**СУБД:** SQLite  
**Файл базы:** `lab04_example_bike_rentals_expanded.sqlite`

> Образец показывает оформление лабораторной работы на **другой предметной области**. В основной лабораторной студенты используют образовательную базу и самостоятельно выполняют свой вариант. Скриншоты необходимо получить при выполнении SQL в SandboxSQL; изображения по указанным путям пока не включены.

## Шаг 1. Загрузка базы и первичная проверка

В SandboxSQL загружена SQLite-база. Выполнена проверка числа строк во всех шести таблицах.

**Шаг 1 — количество записей.** 

```sql
SELECT 'customers' AS table_name, COUNT(*) AS rows_count FROM customers UNION ALL SELECT 'stations', COUNT(*) FROM stations UNION ALL SELECT 'bicycles', COUNT(*) FROM bicycles UNION ALL SELECT 'rentals', COUNT(*) FROM rentals UNION ALL SELECT 'payments', COUNT(*) FROM payments UNION ALL SELECT 'maintenance', COUNT(*) FROM maintenance;
```

**Результат:**

| table_name | rows_count |
|---|---|
| customers | 180 |
| stations | 12 |
| bicycles | 45 |
| rentals | 950 |
| payments | 780 |
| maintenance | 150 |

![Шаг 1 — количество записей](../images/step01_counts.png)

**Вывод:** все шесть таблиц доступны, всего 2117 записей.

## Шаг 2. Изучение структуры и формулирование аналитических вопросов

База содержит шесть связанных таблиц:

| Таблица | Назначение | Основные поля | Записей |
|---|---|---|---:|
| `customers` | Клиенты | `id`, `full_name`, `city`, `registered_at` | 180 |
| `stations` | Пункты проката | `id`, `station_name`, `city`, `capacity` | 12 |
| `bicycles` | Велосипеды и тарифы | `id`, `model`, `category`, `daily_price`, `station_id`, `purchase_year` | 45 |
| `rentals` | Факты аренды | `id`, `customer_id`, `bicycle_id`, `rental_days`, `start_date`, `status` | 950 |
| `payments` | Платежи по завершённым арендам | `id`, `rental_id`, `amount`, `payment_method`, `paid_at` | 780 |
| `maintenance` | Обслуживание велосипедов | `id`, `bicycle_id`, `service_date`, `service_type`, `cost` | 150 |

**Связи:** `bicycles.station_id → stations.id`; `rentals.customer_id → customers.id`; `rentals.bicycle_id → bicycles.id`; `payments.rental_id → rentals.id`; `maintenance.bicycle_id → bicycles.id`. В каждой таблице `id` — первичный ключ. Для `payments.rental_id` задана уникальность: одной аренде соответствует максимум один платёж.

**Шаг 2 — просмотр данных.** Просмотр первых пяти аренд.

```sql
SELECT * FROM rentals LIMIT 5;
```

**Результат:**

| id | customer_id | bicycle_id | rental_days | start_date | status |
|---|---|---|---|---|---|
| 1 | 135 | 21 | 8 | 2026-04-01 | completed |
| 2 | 23 | 17 | 6 | 2026-08-05 | completed |
| 3 | 152 | 36 | 3 | 2026-09-14 | completed |
| 4 | 24 | 38 | 6 | 2026-02-26 | completed |
| 5 | 112 | 21 | 4 | 2026-06-18 | completed |

![Шаг 2 — просмотр данных](../images/step02_preview.png)

**Шаг 2 — структура таблицы.** Просмотр полей и типов данных таблицы `rentals`.

```sql
PRAGMA table_info(rentals);
```

**Результат:**

| cid | name | type | notnull | dflt_value | pk |
|---|---|---|---|---|---|
| 0 | id | INTEGER | 0 | NULL | 1 |
| 1 | customer_id | INTEGER | 1 | NULL | 0 |
| 2 | bicycle_id | INTEGER | 1 | NULL | 0 |
| 3 | rental_days | INTEGER | 1 | NULL | 0 |
| 4 | start_date | TEXT | 1 | NULL | 0 |
| 5 | status | TEXT | 1 | NULL | 0 |

![Шаг 2 — структура таблицы](../images/step02_schema.png)

**Сформулированные аналитические вопросы:**

1. Какие велосипеды имеют тариф от 1000 рублей в день? (`bicycles`)
2. Сколько всего аренд зарегистрировано? (`rentals`)
3. Как распределяются аренды по статусам? (`rentals`)
4. Как соотносятся аренды с клиентами и пунктами проката? (`rentals`, `customers`, `bicycles`, `stations`)
5. Есть ли некорректные длительности аренды и платежи без аренды? (`rentals`, `payments`)
6. Какие категории велосипедов принесли наибольшую выручку? (`payments`, `rentals`, `bicycles`)
7. Какие пункты проката связаны с наибольшей выручкой? (`stations`, `bicycles`, `rentals`, `payments`)

## Шаг 3. Фильтрация и сортировка

**Шаг 3 — фильтрация по тарифу.** Отбираем велосипеды с дневным тарифом не ниже 1000 рублей.

```sql
SELECT id, model, category, daily_price FROM bicycles WHERE daily_price >= 1000 ORDER BY daily_price DESC, model LIMIT 10;
```

**Результат:**

| id | model | category | daily_price |
|---|---|---|---|
| 31 | Urban 100 4 | Городской | 1700 |
| 8 | Trail Mini 1 | Горный | 1650 |
| 1 | Urban 100 1 | Городской | 1650 |
| 11 | Urban 100 2 | Городской | 1650 |
| 21 | Urban 100 3 | Городской | 1600 |
| 2 | City Lite 1 | Городской | 1500 |
| 6 | Rock Rider 1 | Горный | 1500 |
| 36 | Rock Rider 4 | Горный | 1500 |
| 44 | Trail Pro 5 | Горный | 1500 |
| 7 | Commuter 1 | Городской | 1400 |

![Шаг 3 — фильтрация по тарифу](../images/step03_filter.png)

**Шаг 3 — сортировка клиентов.** Выводим клиентов из Казани по алфавиту.

```sql
SELECT id, full_name, city, registered_at FROM customers WHERE city = 'Казань' ORDER BY full_name LIMIT 10;
```

**Результат:**

| id | full_name | city | registered_at |
|---|---|---|---|
| 161 | Алексей Зайцев | Казань | 2025-01-07 |
| 1 | Алексей Иванов | Казань | 2025-04-22 |
| 141 | Алексей Котова | Казань | 2025-06-15 |
| 81 | Алексей Орлова | Казань | 2025-08-23 |
| 176 | Алина Зайцев | Казань | 2025-08-25 |
| 96 | Алина Орлова | Казань | 2025-07-14 |
| 85 | Анна Орлова | Казань | 2025-01-05 |
| 105 | Анна Фомин | Казань | 2025-12-01 |
| 35 | Артём Белова | Казань | 2025-07-03 |
| 50 | Вера Левин | Казань | 2025-03-11 |

![Шаг 3 — сортировка клиентов](../images/step03_sort.png)

**Вывод:** `WHERE` отбирает нужные строки, `ORDER BY` сортирует, `LIMIT` ограничивает объём вывода.

## Шаг 4. Агрегирование и группировка

**Шаг 4 — подсчёт аренд.** Определяем общее число аренд.

```sql
SELECT COUNT(*) AS total_rentals FROM rentals;
```

**Результат:**

| total_rentals |
|---|
| 950 |

![Шаг 4 — подсчёт аренд](../images/step04_count.png)

**Шаг 4 — агрегатные функции.** Рассчитываем минимальный, максимальный, средний и суммарный дневной тариф.

```sql
SELECT MIN(daily_price) AS min_price, MAX(daily_price) AS max_price, ROUND(AVG(daily_price),2) AS avg_price, SUM(daily_price) AS sum_prices FROM bicycles;
```

**Результат:**

| min_price | max_price | avg_price | sum_prices |
|---|---|---|---|
| 550 | 1700 | 1077.78 | 48500 |

![Шаг 4 — агрегатные функции](../images/step04_aggregates.png)

**Шаг 4 — GROUP BY и HAVING.** Группируем аренды по статусу и оставляем статусы, встречающиеся более 100 раз.

```sql
SELECT status, COUNT(*) AS rentals_count FROM rentals GROUP BY status HAVING COUNT(*) > 100 ORDER BY rentals_count DESC;
```

**Результат:**

| status | rentals_count |
|---|---|
| completed | 780 |
| active | 120 |

![Шаг 4 — GROUP BY и HAVING](../images/step04_having.png)

**Вывод:** в базе 950 аренд. `HAVING` отбирает группы после агрегирования; условие применяется к `COUNT(*)`.

## Шаг 5. Объединение связанных таблиц

**Шаг 5 — аренды и клиенты.** Используем `INNER JOIN`, чтобы показать первые 10 аренд вместе с именами клиентов.

```sql
SELECT r.id AS rental_id, c.full_name, r.start_date, r.status FROM rentals r INNER JOIN customers c ON r.customer_id=c.id ORDER BY r.id LIMIT 10;
```

**Результат:**

| rental_id | full_name | start_date | status |
|---|---|---|---|
| 1 | Артём Соколова | 2026-04-01 | completed |
| 2 | Игорь Белова | 2026-08-05 | completed |
| 3 | Никита Котова | 2026-09-14 | completed |
| 4 | Елена Белова | 2026-02-26 | completed |
| 5 | Никита Фомин | 2026-06-18 | completed |
| 6 | Ольга Котова | 2026-01-25 | completed |
| 7 | Алексей Котова | 2026-04-10 | completed |
| 8 | Вера Иванов | 2026-04-01 | completed |
| 9 | Денис Иванов | 2026-02-14 | completed |
| 10 | Роман Иванов | 2026-07-18 | completed |

![Шаг 5 — аренды и клиенты](../images/step05_join_customers.png)

**Шаг 5 — велосипеды и станции.** Используем `INNER JOIN`, чтобы показать первые 10 велосипедов и пункты проката.

```sql
SELECT b.id, b.model, s.station_name, s.city FROM bicycles b INNER JOIN stations s ON b.station_id=s.id ORDER BY b.id LIMIT 10;
```

**Результат:**

| id | model | station_name | city |
|---|---|---|---|
| 1 | Urban 100 1 | Пункт проката №9 | Пермь |
| 2 | City Lite 1 | Пункт проката №4 | Пермь |
| 3 | Speed X 1 | Пункт проката №7 | Самара |
| 4 | Trail Pro 1 | Пункт проката №12 | Самара |
| 5 | Road One 1 | Пункт проката №11 | Казань |
| 6 | Rock Rider 1 | Пункт проката №1 | Казань |
| 7 | Commuter 1 | Пункт проката №4 | Пермь |
| 8 | Trail Mini 1 | Пункт проката №1 | Казань |
| 9 | Cross 200 1 | Пункт проката №7 | Самара |
| 10 | Metro Bike 1 | Пункт проката №12 | Самара |

![Шаг 5 — велосипеды и станции](../images/step05_join_stations.png)

**Шаг 5 — все станции и велосипеды.** Используем `LEFT JOIN`, чтобы сохранить в результате все пункты проката, включая пункты без велосипедов.

```sql
SELECT s.id, s.station_name, b.model FROM stations s LEFT JOIN bicycles b ON b.station_id=s.id ORDER BY s.id,b.id;
```

**Результат:**

| id | station_name | model |
|---|---|---|
| 1 | Пункт проката №1 | Rock Rider 1 |
| 1 | Пункт проката №1 | Trail Mini 1 |
| 1 | Пункт проката №1 | Trail Mini 4 |
| 1 | Пункт проката №1 | Urban 100 5 |
| 2 | Пункт проката №2 | Trail Mini 2 |
| 2 | Пункт проката №2 | Metro Bike 2 |
| 2 | Пункт проката №2 | Road One 3 |
| 2 | Пункт проката №2 | Urban 100 4 |
| 3 | Пункт проката №3 | NULL |
| 4 | Пункт проката №4 | City Lite 1 |
| 4 | Пункт проката №4 | Commuter 1 |
| 4 | Пункт проката №4 | Cross 200 2 |

*Показаны первые 12 строк результата.*

![Шаг 5 — все станции и велосипеды](../images/step05_left_join.png)

**Вывод:** `INNER JOIN` выводит совпавшие записи, а `LEFT JOIN` сохраняет строки из левой таблицы даже при отсутствии соответствий.

## Шаг 6. Проверка корректности данных

**Шаг 6 — проверка длительности аренды.** Проверяем, нет ли аренд с нулевой, отрицательной или отсутствующей длительностью.

```sql
SELECT COUNT(*) AS invalid_rental_days FROM rentals WHERE rental_days <= 0 OR rental_days IS NULL;
```

**Результат:**

| invalid_rental_days |
|---|
| 0 |

![Шаг 6 — проверка длительности аренды](../images/step06_invalid.png)

**Шаг 6 — проверка связей платежей.** Проверяем, нет ли платежей, ссылающихся на несуществующую аренду.

```sql
SELECT COUNT(*) AS orphan_payments FROM payments p LEFT JOIN rentals r ON p.rental_id=r.id WHERE r.id IS NULL;
```

**Результат:**

| orphan_payments |
|---|
| 0 |

![Шаг 6 — проверка связей платежей](../images/step06_orphans.png)

**Вывод:** оба запроса вернули 0. В рамках проведённых проверок нарушений не обнаружено. Это не означает, что проверены абсолютно все возможные ошибки в данных.

## Шаг 7. Индивидуальное аналитическое исследование (пример)

**Исследовательский вопрос:** какие категории велосипедов и пункты проката связаны с наибольшей фактической выручкой по зарегистрированным платежам?

**Метод:** учитываем только аренды, у которых имеется запись в `payments`; выручка рассчитывается как `SUM(payments.amount)`. Незавершённые и отменённые аренды без платежа не включаются. Расходы на обслуживание не вычитаются.

**Шаг 7 — аналитика по категориям.** Группируем оплаченные аренды по категориям велосипедов.

```sql
SELECT b.category, COUNT(*) AS completed_rentals, SUM(p.amount) AS revenue_rub, ROUND(AVG(p.amount),2) AS avg_payment_rub FROM payments p INNER JOIN rentals r ON p.rental_id=r.id INNER JOIN bicycles b ON r.bicycle_id=b.id GROUP BY b.category ORDER BY revenue_rub DESC;
```

**Результат:**

| category | completed_rentals | revenue_rub | avg_payment_rub |
|---|---|---|---|
| Городской | 309 | 1601550 | 5183.01 |
| Горный | 231 | 1118500 | 4841.99 |
| Шоссейный | 165 | 605600 | 3670.3 |
| Гибридный | 75 | 308500 | 4113.33 |

![Шаг 7 — аналитика по категориям](../images/step07_categories.png)

**Шаг 7 — аналитика по станциям.** Сравниваем пункты проката по сумме платежей.

```sql
SELECT s.station_name, COUNT(*) AS paid_rentals, SUM(p.amount) AS revenue_rub FROM payments p INNER JOIN rentals r ON p.rental_id=r.id INNER JOIN bicycles b ON r.bicycle_id=b.id INNER JOIN stations s ON b.station_id=s.id GROUP BY s.id,s.station_name ORDER BY revenue_rub DESC LIMIT 10;
```

**Результат:**

| station_name | paid_rentals | revenue_rub |
|---|---|---|
| Пункт проката №7 | 102 | 507600 |
| Пункт проката №11 | 99 | 461000 |
| Пункт проката №4 | 85 | 458300 |
| Пункт проката №1 | 85 | 415200 |
| Пункт проката №9 | 70 | 370850 |
| Пункт проката №2 | 74 | 350600 |
| Пункт проката №6 | 92 | 276900 |
| Пункт проката №12 | 57 | 272800 |
| Пункт проката №5 | 53 | 253750 |
| Пункт проката №8 | 53 | 229550 |

![Шаг 7 — аналитика по станциям](../images/step07_station_revenue.png)

**Дополнительная проверка — велосипеды без аренд:**

**Шаг 7 — неиспользуемые велосипеды.** Используем `LEFT JOIN` для поиска велосипедов без единой записи в `rentals`.

```sql
SELECT b.id, b.model, b.category FROM bicycles b LEFT JOIN rentals r ON b.id=r.bicycle_id WHERE r.id IS NULL ORDER BY b.id;
```

**Результат:**

| id | model | category |
|---|---|---|
| 43 | Speed X 5 | Шоссейный |
| 44 | Trail Pro 5 | Горный |
| 45 | Road One 5 | Шоссейный |

![Шаг 7 — неиспользуемые велосипеды](../images/step07_unused.png)

**Аналитические выводы:**

1. Наибольшую сумму платежей принесла категория **Городской** — **1,601,550 руб.** (309 оплаченных аренд).
2. Общая зарегистрированная выручка по 780 платежам составляет **3,634,150 руб.**; это сумма платежей, а не прибыль сервиса.
3. Сравнение пунктов проката показывает различия в выручке. Рейтинг зависит как от числа аренд, так и от тарифа и длительности аренды.
4. Велосипедов без единой аренды — **3**. Их наличие можно использовать как повод для отдельного анализа спроса.

## Итог

В работе изучена структура SQLite-базы из шести связанных таблиц, выполнены запросы с `WHERE`, `ORDER BY`, `LIMIT`, агрегатными функциями, `GROUP BY`, `HAVING`, `INNER JOIN`, `LEFT JOIN`; проведены проверки целостности и отдельное аналитическое исследование.

**Ограничения:** платежи отражают зарегистрированную выручку, но не учитывают все возможные расходы и возвраты. Выводы относятся только к данному учебному набору данных.

## Структура файлов на GitHub

```text
lab04/
├── README.md
├── lab04_example_bike_rentals_expanded.sqlite
├── report/
│   └── analysis.md
└── images/
    ├── step01_counts.png
    ├── step02_preview.png
    ├── step02_schema.png
    ├── step03_filter.png
    ├── step03_sort.png
    ├── step04_count.png
    ├── step04_aggregates.png
    ├── step04_having.png
    ├── step05_join_customers.png
    ├── step05_join_stations.png
    ├── step05_left_join.png
    ├── step06_invalid.png
    ├── step06_orphans.png
    ├── step07_categories.png
    ├── step07_station_revenue.png
    └── step07_unused.png
```
