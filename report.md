# Лабораторная работа 1. Проектирование и создание структуры базы данных


Выполнил: Фёдоров Алексей Сергеевич

Студент группы: БВТ2504

Проверил: 


## 1. Анализ предметной области и бизнес-правила

### 1.1. Описание предметной области

В системе действуют **пользователи**. Пользователь может быть покупателем, администратором или продавцом. У пользователя может быть несколько адресов доставки.

**Продавец** выставляет **товары**. Каждый товар принадлежит ровно одному продавцу и относится к одной или нескольким **категориям**; категории образуют дерево.

**Покупатель** оформляет **заказ** на выбранный адрес доставки. Заказ состоит из **позиций** (товар, количество, цена на момент покупки). Заказ оплачивается **платежами** и передаётся в **доставку**. Купленные товары покупатель может оценить и оставить **отзыв**.

### 1.2. Бизнес-правила

| № | Бизнес-правило |
|---|---|
| БП-1 | Пользователь идентифицируется уникальным e-mail; телефон, если указан, тоже уникален. Допустимые роли: customer, seller, admin 
| БП-2 | Продавец — это пользователь с профилем продавца; у пользователя не более одного профиля; ИНН уникален и состоит из 10–12 цифр 
| БП-3 | У пользователя может быть несколько адресов доставки; удаление пользователя удаляет его адреса (если они не используются в заказах) |
| БП-4 | **У товара ровно один продавец**; артикул уникален в пределах продавца | 
| БП-5 | Категории образуют иерархию: у категории не более одного родителя, категория не может быть родителем самой себе; товар может относиться к нескольким категориям, а категория содержать много товаров | 
| БП-6 | Цена и остаток товара не могут быть отрицательными | 
| БП-7 | **Заказ принадлежит ровно одному покупателю**, имеет уникальный номер, адрес доставки | 
| БП-8 | **Заказ содержит хотя бы одну позицию**; позиции нумеруются, количество положительно, цена фиксируется на момент покупки | 
| БП-9 | Заказ можно оплачивать несколькими платежами (повторные попытки, возвраты); сумма платежа положительна; транзакция провайдера уникальна |
| БП-10 | Заказ может доставляться несколькими отправлениями (например, товары разных продавцов); трек-номер уникален | 
| БП-11 | **Отзыв оставляется к конкретному товару конкретным пользователем**, не более одного отзыва на пару; оценка от 1 до 5 | 
| БП-12 | Дата изменения товара, заказа и отзыва обновляется автоматически |
---

## 2. Сущности и ER-диаграмма

### 2.1. Перечень сущностей

В схеме **11 сущностей** (требовалось не менее 10).

| № | Сущность | Тип | Назначение |
|---|---|---|---|
| 1 | `users` | основная | учётные записи |
| 2 | `addresses` | зависимая | адреса пользователей |
| 3 | `sellers` | зависимая (1:1 с `users`) | профили продавцов |
| 4 | `categories` | справочная, иерархическая | дерево категорий |
| 5 | `products` | основная | товары |
| 6 | `product_category` | ассоциативная | связь товар ↔ категория (M:N) |
| 7 | `orders` | основная | заказы |
| 8 | `order_item` | ассоциативная с атрибутами | позиции заказа (заказ ↔ товар, M:N) |
| 9 | `payments` | зависимая | платежи |
| 10 | `deliveries` | зависимая | доставки |
| 11 | `reviews` | ассоциативная с атрибутами | отзывы (пользователь ↔ товар, M:N) |

### 2.2. ER-диаграмма

Диаграмма построена в расширении ERD Editor для VS Code (файл `marketplace.vuerd.json`). 

### 2.3. Связи и кардинальности

| Родитель (1) | Дочерняя таблица | Кардинальность | Внешний ключ |
|---|---|---|---|
| `users` | `addresses` | 1 — 0..N | `addresses.user_id` |
| `users` | `sellers` | 1 — 0..1 | `sellers.user_id` (UNIQUE) |
| `users` | `orders` | 1 — 0..N | `orders.user_id` |
| `users` | `reviews` | 1 — 0..N | `reviews.user_id` |
| `sellers` | `products` | 1 — 0..N | `products.seller_id` |
| `categories` | `categories` | 1 — 0..N | `categories.parent_id` (самоссылка) |
| `products` | `product_category` | 1 — 0..N | `product_category.product_id` |
| `categories` | `product_category` | 1 — 0..N | `product_category.category_id` |
| `orders` | `order_item` | 1 — 1..N | `order_item.order_id` |
| `products` | `order_item` | 1 — 0..N | `order_item.product_id` |
| `orders` | `payments` | 1 — 0..N | `payments.order_id` |
| `orders` | `deliveries` | 1 — 0..N | `deliveries.order_id` |
| `addresses` | `deliveries` | 1 — 0..N | `deliveries.address_id` |
| `addresses` | `orders` | 1 — 0..N | `orders.delivery_address_id` |
| `products` | `reviews` | 1 — 0..N | `reviews.product_id` |

### 2.4. Связи «многие-ко-многим» и иерархия

Схема содержит три связи M:N, каждая разложена на две связи через ассоциативную таблицу:

- **товар ↔ категория** — таблица `product_category` с составным первичным ключом `(product_id, category_id)`; собственных атрибутов нет;
- **заказ ↔ товар** — таблица `order_item`; атрибуты связи: `line_no`, `quantity`, `unit_price`;
- **пользователь ↔ товар** — таблица `reviews`; атрибуты связи: `rating`, `comment`; пара `(user_id, product_id)` уникальна, поэтому пользователь оценивает товар не более одного раза.

**Иерархическая связь** — `categories.parent_id → categories.category_id` (самоссылка). Нужна, чтобы хранить дерево категорий любой вложенности в одной таблице без лишних сущностей и при этом сохранять целостность: родителем может быть только существующая категория.

---

## 3. Нормализация до третьей нормальной формы

### 3.1. Исходная ненормализованная таблица

Если бы всё хранилось в одной таблице, то получилась бы такая структура `orders_flat`, где ключ — `(order_id, product_sku)`:
| order_id | order_number | order_date | buyer_email | buyer_name | buyer_city | product_sku | product_name | product_price | seller_name | seller_inn | quantity |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | ORD-2026-0001 | 2026-09-01 | anna@example.com | Анна Иванова | Tallinn | PH-X1 | Смартфон X1 | 399.90 | TechBoris | 7701234567 | 1 |
| 1 | ORD-2026-0001 | 2026-09-01 | anna@example.com | Анна Иванова | Tallinn | TS-B | Футболка базовая | 15.00 | ClaraStyle | 500100732259 | 3 |
| 2 | ORD-2026-0002 | 2026-09-20 | dmitri@example.com | Дмитрий Орлов | Tallinn | NB-14 | Ноутбук Pro 14 | 1199.00 | TechBoris | 7701234567 | 1 |
| 3 | ORD-2026-0003 | 2026-09-24 | anna@example.com | Анна Иванова | Tallinn | PH-X1 | Смартфон X1 | 399.90 | TechBoris | 7701234567 | 2 |


### 3.2. Аномалии 

- **Аномалия обновления.** Название продавца хранится в каждой строке заказа. При обновлении одной строки для одного и того же ИНН оказалось два разных названия — данные противоречивы.
- **Аномалия вставки.** Нельзя добавить товар, который ещё никто не заказал: `order_id` входит в первичный ключ и не может быть NULL.
- **Аномалия удаления.** Удаление единственного заказа на ноутбук стёрло из базы всё, что мы знали о самом товаре `NB-14`.

### 3.3. Приведение к 1НФ → 2НФ → 3НФ

1. **1НФ.** Все значения атомарны, повторяющихся групп нет, ключ определён. Уточнение при проектировании: адрес разбит на отдельные поля (`country`, `city`, `street`, `building`, …), а несколько адресов пользователя вынесены в отдельную таблицу `addresses`, а не в повторяющиеся столбцы.
2. **2НФ** — устраняем частичные зависимости от части составного ключа. Атрибуты заказа зависят только от `order_id`, атрибуты товара — только от `product_sku`. Они выносятся в `orders` и `products`; в связующей таблице остаётся только то, что зависит от пары, — `order_item(quantity, unit_price)`.
3. **3НФ** — устраняем транзитивные зависимости. Цепочка `order_id → buyer_email → buyer_name` выносит покупателя в `users`; цепочка `product_sku → seller_name → seller_inn` выносит продавца в `sellers`.

---

## 4. Первичные ключи

| Таблица | Первичный ключ | Тип ключа | Альтернативные (естественные) ключи — UNIQUE |
|---|---|---|---|
| `users` | `user_id` | суррогатный | `email`, `phone` |
| `addresses` | `address_id` | суррогатный | — |
| `sellers` | `seller_id` | суррогатный | `user_id`, `inn` |
| `categories` | `category_id` | суррогатный | `(parent_id, slug)` |
| `products` | `product_id` | суррогатный | `(seller_id, sku)` |
| `product_category` | `(product_id, category_id)` | **составной естественный** | — |
| `orders` | `order_id` | суррогатный | `order_number` |
| `order_item` | `order_item_id` | суррогатный | `(order_id, line_no)` |
| `payments` | `payment_id` | суррогатный | `(provider, transaction_id)` |
| `deliveries` | `delivery_id` | суррогатный | `tracking_number` |
| `reviews` | `review_id` | суррогатный | `(user_id, product_id)` |

**Обоснование.**

- **Суррогатный ключ `BIGINT GENERATED ALWAYS AS IDENTITY`** выбран для всех самостоятельных сущностей. Естественные кандидаты (e-mail, телефон, ИНН, номер заказа, артикул) могут меняться, бывают пустыми (`phone`, `inn`), длинными или составными; ссылаться на них из внешних ключей было бы неудобно. Суррогат неизменен, компактен (8 байт) и не несёт бизнес-смысла, который может измениться.
- **`GENERATED ALWAYS`** запрещает вставлять идентификаторы вручную, поэтому значения нельзя случайно продублировать или «сбить» счётчик. **`BIGINT`**, а не `INTEGER`, — запас ёмкости для таблиц, которые растут быстрее всего (`orders`, `order_item`, `payments`).
- **Естественные ключи не пропадают** — они остаются как ограничения `UNIQUE`, поэтому бизнес-уникальность (один e-mail, один номер заказа, один отзыв на пару) защищена на уровне БД.
- **`product_category`** — единственная таблица с составным естественным ключом. Это чистая связующая таблица без собственных атрибутов, на неё никто не ссылается, а пара `(product_id, category_id)` сама по себе гарантирует отсутствие дублей. 
- **`order_item` и `reviews`** могли бы использовать составные ключи `(order_id, line_no)` и `(user_id, product_id)`. Выбран суррогат, потому что на такие сущности в будущем могут ссылаться другие таблицы (возвраты по конкретной позиции, жалобы на отзыв) — ссылка на одну колонку проще, — а составные естественные ключи сохранены как `UNIQUE`.

---

## 5. Реализация схемы в PostgreSQL (DDL)

Реализация — файл `schema.sql`. 

### 5.1. Типы данных

| Тип | Где | Почему |
|---|---|---|
| `BIGINT` | ключи | BIGINT, а не INTEGER, — запас ёмкости для таблиц, которые растут быстрее всего (orders, order_item, payments). |
| `VARCHAR(n)` | e-mail, названия, коды | ограничение длины защищает от мусора и документирует ожидаемый размер |
| `TEXT` | `description`, `comment` | свободный текст без разумного предела длины |
| `NUMERIC(12,2)` / `NUMERIC(14,2)` | цены, суммы | точная десятичная арифметика; `REAL`/`DOUBLE PRECISION` дают ошибки округления, тип `MONEY` привязан к локали. Суммы заказов и платежей шире (14 разрядов), чем цена единицы (12) |
| `INTEGER` | `stock`, `quantity`, `line_no` | целые значения в разумных пределах |
| `SMALLINT` | `rating` | значения 1–5 |
| `BOOLEAN` | `is_active`, `is_default` | флаги |
| `TIMESTAMPTZ` | все даты | хранит момент времени независимо от часового пояса клиента; `TIMESTAMP` без пояса приводит к ошибкам при работе из разных регионов |

### 5.2. Ограничения

- **NOT NULL** — на всех обязательных атрибутах и внешних ключах, которые выражают обязательную связь (например, `products.seller_id`, `orders.user_id`). NULL допускается только там, где значение действительно может отсутствовать: `phone`, `inn`, `parent_id` корневой категории, `paid_at`, `shipped_at`, `delivered_at`.
- **UNIQUE** — бизнес-ключи 
- **CHECK** — области допустимых значений: неотрицательные цены и остатки, положительные количества и суммы платежей, диапазон оценки, форматы e-mail и ИНН.
- **DEFAULT** — для служебных и типовых значений: `now()` для дат создания, `TRUE`/`FALSE` для флагов, начальные статусы (`'customer'`, `'draft'`, `'created'`, `'pending'`, `'active'`), `0` для остатка и суммы.

### 5.3. Ссылочные действия ON DELETE / ON UPDATE

| Внешний ключ | Дочерняя → родительская | ON DELETE | ON UPDATE | Обоснование |
|---|---|---|---|---|
| `fk_addresses_user` | addresses.user_id → users.user_id | CASCADE | CASCADE | Адрес — часть профиля пользователя и не имеет смысла без него. Если на адрес ссылаются заказы или доставки, их RESTRICT не позволит удалить пользователя. |
| `fk_sellers_user` | sellers.user_id → users.user_id | RESTRICT | CASCADE | Профиль продавца не должен исчезать вместе с учётной записью: за ним закреплены товары и история продаж. |
| `fk_categories_parent` | categories.parent_id → categories.category_id | SET NULL | CASCADE | При удалении родительской категории дочерние не пропадают, а становятся корневыми (parent_id = NULL). |
| `fk_products_seller` | products.seller_id → sellers.seller_id | RESTRICT | CASCADE | Товары нельзя терять при удалении продавца — на них ссылаются заказы; продавца «закрывают» статусом. |
| `fk_pc_category` | product_category.category_id → categories.category_id | CASCADE | CASCADE | Строка связи не имеет смысла без категории — удаляется вместе с ней (товар остаётся). |
| `fk_pc_product` | product_category.product_id → products.product_id | CASCADE | CASCADE | Строка связи не имеет самостоятельного смысла без товара — удаляется вместе с ним. |
| `fk_orders_address` | orders.delivery_address_id → addresses.address_id | RESTRICT | CASCADE | Адрес, использованный в заказе, нельзя удалить — иначе потеряется адрес доставки. |
| `fk_orders_user` | orders.user_id → users.user_id | RESTRICT | CASCADE | История заказов неприкосновенна: пользователя с заказами удалить нельзя (используется is_active). |
| `fk_order_item_order` | order_item.order_id → orders.order_id | CASCADE | CASCADE | Позиции — составная часть заказа и удаляются вместе с ним. |
| `fk_order_item_product` | order_item.product_id → products.product_id | RESTRICT | CASCADE | Товар, попавший в заказ, удалить нельзя — иначе потеряется состав заказа; товар архивируют статусом. |
| `fk_payments_order` | payments.order_id → orders.order_id | CASCADE | CASCADE | Платежи привязаны к заказу и удаляются вместе с ним (см. раздел 10 про финансовые записи). |
| `fk_deliveries_address` | deliveries.address_id → addresses.address_id | RESTRICT | CASCADE | Адрес, по которому выполняется доставка, нельзя удалить. |
| `fk_deliveries_order` | deliveries.order_id → orders.order_id | CASCADE | CASCADE | Доставка не существует без заказа и удаляется вместе с ним. |
| `fk_reviews_product` | reviews.product_id → products.product_id | CASCADE | CASCADE | Отзыв не имеет смысла без товара. |
| `fk_reviews_user` | reviews.user_id → users.user_id | CASCADE | CASCADE | Отзывы удаляются вместе с учётной записью автора (если её вообще удаляют). |


Принцип выбора: **CASCADE** — для «составных частей» (позиции заказа, связи товар-категория, адреса, отзывы); **RESTRICT** — для ссылок на «самостоятельные» сущности, чьё удаление разрушило бы историю (пользователь, продавец, товар, адрес); **SET NULL** — там, где объект переживает удаление родителя (дочерняя категория становится корневой).

**ON UPDATE CASCADE** указан единообразно. Поскольку все ключи — `GENERATED ALWAYS`, изменить их обычным `UPDATE` нельзя (PostgreSQL вернёт ошибку `column can only be updated to DEFAULT`), так что на практике правило «спит», но оно делает схему безопасной для миграций и переноса данных, если ключи когда-либо придётся пересчитывать.

---

## 6. Словарь данных

### 6.1. `users`

Учётные записи всех участников: покупатели, продавцы, администраторы.

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `user_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `email` | varchar(255) | NOT NULL | — | CHECK: формат e-mail (регулярное выражение); UNIQUE | Адрес электронной почты (логин) |
| `phone` | varchar(20) | NULL | — | UNIQUE | Телефон (необязательно) |
| `full_name` | varchar(200) | NOT NULL | — | — | Имя и фамилия |
| `role` | varchar(20) | NOT NULL | 'customer' | CHECK: role ∈ {customer, seller, admin} | Роль: customer / seller / admin |
| `is_active` | boolean | NOT NULL | true | — | Признак активной учётной записи (вместо физического удаления) |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата и время регистрации |

### 6.2. `addresses`

Адреса доставки пользователей.

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `address_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `user_id` | bigint | NOT NULL | — | FK → users.user_id; ON DELETE CASCADE; ON UPDATE CASCADE | Владелец адреса |
| `country` | varchar(100) | NOT NULL | — | — | Страна |
| `city` | varchar(100) | NOT NULL | — | — | Город |
| `street` | varchar(200) | NOT NULL | — | — | Улица |
| `building` | varchar(20) | NOT NULL | — | — | Дом / строение |
| `apartment` | varchar(20) | NULL | — | — | Квартира / офис (необязательно) |
| `postal_code` | varchar(20) | NULL | — | — | Почтовый индекс (необязательно) |
| `is_default` | boolean | NOT NULL | false | — | Адрес по умолчанию |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата добавления адреса |

### 6.3. `sellers`

Профиль продавца (расширение учётной записи, связь 1:1).

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `seller_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `user_id` | bigint | NOT NULL | — | FK → users.user_id; ON DELETE RESTRICT; ON UPDATE CASCADE; UNIQUE | Учётная запись владельца (профиль продавца 1:1) |
| `display_name` | varchar(200) | NOT NULL | — | — | Отображаемое название продавца / магазина |
| `inn` | varchar(12) | NULL | — | CHECK: NULL или 10–12 цифр; UNIQUE | ИНН, 10–12 цифр (необязательно) |
| `status` | varchar(20) | NOT NULL | 'active' | CHECK: status ∈ {active, suspended, closed} | Статус: active / suspended / closed |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата регистрации продавца |

### 6.4. `categories`

Иерархический каталог категорий (самоссылка).

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `category_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `parent_id` | bigint | NULL | — | FK → categories.category_id; ON DELETE SET NULL; ON UPDATE CASCADE | Родительская категория; NULL — корневая |
| `name` | varchar(150) | NOT NULL | — | — | Название категории |
| `slug` | varchar(150) | NOT NULL | — | — | Короткий идентификатор для URL |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата создания категории |


### 6.5. `products`

Товары, выставленные продавцами.

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `product_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `seller_id` | bigint | NOT NULL | — | FK → sellers.seller_id; ON DELETE RESTRICT; ON UPDATE CASCADE | Единственный продавец товара |
| `sku` | varchar(64) | NOT NULL | — | — | Артикул (складской код) товара |
| `name` | varchar(300) | NOT NULL | — | — | Название товара |
| `description` | text | NULL | — | — | Описание товара (необязательно) |
| `price` | numeric(12,2) | NOT NULL | — | CHECK: price ≥ 0 | Текущая цена |
| `stock` | integer | NOT NULL | 0 | CHECK: stock ≥ 0 | Остаток на складе |
| `status` | varchar(20) | NOT NULL | 'draft' | CHECK: status ∈ {draft, active, archived} | Статус: draft / active / archived |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата создания карточки |
| `updated_at` | timestamptz | NOT NULL | now() | — | Дата последнего изменения (обновляется триггером) |


### 6.6. `product_category`

Ассоциативная таблица товар ↔ категория (M:N).

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `product_id` | bigint | NOT NULL | — | FK → products.product_id; ON DELETE CASCADE; ON UPDATE CASCADE; PK (составной) | Товар (часть составного PK) |
| `category_id` | bigint | NOT NULL | — | FK → categories.category_id; ON DELETE CASCADE; ON UPDATE CASCADE; PK (составной) | Категория (часть составного PK) |


### 6.7. `orders`

Заказы покупателей.

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `order_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `order_number` | varchar(32) | NOT NULL | — | UNIQUE | Публичный номер заказа |
| `user_id` | bigint | NOT NULL | — | FK → users.user_id; ON DELETE RESTRICT; ON UPDATE CASCADE | Покупатель — единственный владелец заказа |
| `delivery_address_id` | bigint | NOT NULL | — | FK → addresses.address_id; ON DELETE RESTRICT; ON UPDATE CASCADE | Адрес доставки |
| `status` | varchar(20) | NOT NULL | 'created' | CHECK: status ∈ {created, paid, shipped, delivered, cancelled} | Статус: created / paid / shipped / delivered / cancelled |
| `total_amount` | numeric(14,2) | NOT NULL | 0 | CHECK: total_amount ≥ 0 | Итоговая сумма заказа (денормализованное значение, см. раздел 4.4) |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата оформления |
| `updated_at` | timestamptz | NOT NULL | now() | — | Дата последнего изменения (обновляется триггером) |

### 6.8. `order_item`

Позиции заказа (ассоциативная таблица заказ ↔ товар, M:N).

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `order_item_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `order_id` | bigint | NOT NULL | — | FK → orders.order_id; ON DELETE CASCADE; ON UPDATE CASCADE | Заказ, которому принадлежит позиция |
| `line_no` | integer | NOT NULL | — | CHECK: line_no > 0 | Номер строки внутри заказа |
| `product_id` | bigint | NOT NULL | — | FK → products.product_id; ON DELETE RESTRICT; ON UPDATE CASCADE | Заказанный товар |
| `quantity` | integer | NOT NULL | — | CHECK: quantity > 0 | Количество единиц |
| `unit_price` | numeric(12,2) | NOT NULL | — | CHECK: unit_price ≥ 0 | Цена единицы на момент покупки (историческое значение) |


### 6.9. `payments`

Платежи по заказам (возможны несколько попыток и возвраты).

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `payment_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `order_id` | bigint | NOT NULL | — | FK → orders.order_id; ON DELETE CASCADE; ON UPDATE CASCADE | Оплачиваемый заказ |
| `provider` | varchar(50) | NOT NULL | — | — | Платёжный провайдер |
| `transaction_id` | varchar(100) | NULL | — | — | Идентификатор транзакции у провайдера (необязательно) |
| `amount` | numeric(14,2) | NOT NULL | — | CHECK: amount > 0 | Сумма платежа |
| `status` | varchar(20) | NOT NULL | 'pending' | CHECK: status ∈ {pending, succeeded, failed, refunded} | Статус: pending / succeeded / failed / refunded |
| `paid_at` | timestamptz | NULL | — | — | Фактическая дата оплаты |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата создания платежа |


### 6.10. `deliveries`

Доставки заказов.

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `delivery_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `order_id` | bigint | NOT NULL | — | FK → orders.order_id; ON DELETE CASCADE; ON UPDATE CASCADE | Доставляемый заказ |
| `address_id` | bigint | NOT NULL | — | FK → addresses.address_id; ON DELETE RESTRICT; ON UPDATE CASCADE | Адрес, по которому выполняется доставка |
| `tracking_number` | varchar(100) | NULL | — | UNIQUE | Трек-номер отправления (необязательно) |
| `status` | varchar(20) | NOT NULL | 'pending' | CHECK: status ∈ {pending, shipped, in_transit, delivered, returned} | Статус: pending / shipped / in_transit / delivered / returned |
| `shipped_at` | timestamptz | NULL | — | — | Дата отправки |
| `delivered_at` | timestamptz | NULL | — | — | Дата вручения |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата создания записи |

### 6.11. `reviews`

Отзывы пользователей о товарах.

| Столбец | Тип | NULL | DEFAULT | Ограничения | Назначение |
|---|---|---|---|---|---|
| `review_id` | bigint | NOT NULL | IDENTITY (ALWAYS) | PK | Суррогатный первичный ключ |
| `user_id` | bigint | NOT NULL | — | FK → users.user_id; ON DELETE CASCADE; ON UPDATE CASCADE | Автор отзыва |
| `product_id` | bigint | NOT NULL | — | FK → products.product_id; ON DELETE CASCADE; ON UPDATE CASCADE | Товар, о котором отзыв |
| `rating` | smallint | NOT NULL | — | CHECK: rating BETWEEN 1 AND 5 | Оценка от 1 до 5 |
| `comment` | text | NULL | — | — | Текст отзыва (необязательно) |
| `created_at` | timestamptz | NOT NULL | now() | — | Дата публикации |
| `updated_at` | timestamptz | NOT NULL | now() | — | Дата последнего изменения (обновляется триггером) |


---

## 7. Скрипты и воспроизведение

```text
lab01/
├── schema.sql                 -- DDL: схема, таблицы, ограничения, индексы, триггеры (воссоздаёт схему с нуля)
├── marketplace.vuerd.json     -- ER-диаграмма (ERD Editor)
└── report.md                  -- этот отчёт
```

---

## 8. Вывод

В ходе лабораторной работы была спроектирована схема базы данных маркетплейса, в которой три связи M:N корректно разложены на ассоциативные таблицы: product_category с составным первичным ключом (product_id, category_id) для связи товаров и категорий, order_item с атрибутами line_no, quantity, unit_price для связи заказов и товаров, а также reviews с атрибутами rating, comment и уникальной парой (user_id, product_id), гарантирующей не более одного отзыва пользователя на товар; иерархия категорий реализована через самоссылку parent_id, что обеспечивает отсутствие избыточности, целостность данных за счёт первичных, внешних ключей и ограничений уникальности, гибкость расширения и соответствие нормальным формам, поэтому построенная модель адекватно отражает предметную область и может служить основой для дальнейшей реализации запросов и прикладной логики.




