### 1. Таблица `users` (Пользователи)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор пользователя |
| `full_name` | `VARCHAR(255)` | `NOT NULL` | Полное имя / ФИО пользователя |
| `email` | `VARCHAR(255)` | `NOT NULL`, `UNIQUE` | Электронная почта (логин) |
| `phone` | `VARCHAR(30)` | `UNIQUE` | Контактный номер телефона |
| `role` | `VARCHAR(20)` | `NOT NULL`, `DEFAULT 'customer'`, `CHECK (role IN ('customer', 'seller', 'admin'))` | Роль пользователя в системе |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT now()` | Дата и время регистрации |

---

### 2. Таблица `addresses` (Адреса)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор адреса |
| `user_id` | `BIGINT` | `NOT NULL`, `FOREIGN KEY (users.id) ON DELETE CASCADE ON UPDATE CASCADE` | Владелец адреса |
| `city` | `VARCHAR(100)` | `NOT NULL` | Город |
| `street` | `VARCHAR(100)` | `NOT NULL` | Улица |
| `house` | `VARCHAR(20)` | `NOT NULL` | Номер дома/корпуса |
| `apartment` | `VARCHAR(20)` | — | Номер квартиры/офиса |

---

### 3. Таблица `sellers` (Продавцы / Магазины)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор продавца |
| `user_id` | `BIGINT` | `NOT NULL`, `UNIQUE`, `FOREIGN KEY (users.id) ON DELETE CASCADE ON UPDATE CASCADE` | Аккаунт пользователя-владельца |
| `store_name` | `VARCHAR(255)` | `NOT NULL`, `UNIQUE` | Название магазина |
| `description` | `TEXT` | — | Описание магазина |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT now()` | Дата и время создания магазина |

---

### 4. Таблица `categories` (Категории)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор категории |
| `parent_id` | `BIGINT` | `FOREIGN KEY (categories.id) ON DELETE RESTRICT ON UPDATE CASCADE` | Ссылка на родительскую категорию (`NULL` для корневых) |
| `name` | `VARCHAR(255)` | `NOT NULL`, `UNIQUE` | Наименование категории |

---

### 5. Таблица `products` (Товары)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор товара |
| `seller_id` | `BIGINT` | `NOT NULL`, `FOREIGN KEY (sellers.id) ON DELETE RESTRICT ON UPDATE CASCADE` | Продавец товара |
| `category_id` | `BIGINT` | `NOT NULL`, `FOREIGN KEY (categories.id) ON DELETE RESTRICT ON UPDATE CASCADE` | Категория товара |
| `title` | `VARCHAR(255)` | `NOT NULL` | Название товара |
| `description` | `TEXT` | — | Подробное описание товара |
| `price` | `NUMERIC(12,2)` | `NOT NULL`, `CHECK (price > 0)` | Текущая цена товара |
| `is_active` | `BOOLEAN` | `NOT NULL`, `DEFAULT TRUE` | Флаг активности карточки товара |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT now()` | Дата и время создания карточки |

---

### 6. Таблица `stock` (Складские остатки)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `product_id` | `BIGINT` | `PRIMARY KEY`, `FOREIGN KEY (products.id) ON DELETE CASCADE ON UPDATE CASCADE` | Идентификатор товара (связь 1:1) |
| `quantity` | `INTEGER` | `NOT NULL`, `DEFAULT 0`, `CHECK (quantity >= 0)` | Количество товара на складе |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT now()` | Время последнего изменения остатков |

---

### 7. Таблица `carts` (Корзины)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор корзины |
| `user_id` | `BIGINT` | `NOT NULL`, `UNIQUE`, `FOREIGN KEY (users.id) ON DELETE CASCADE ON UPDATE CASCADE` | Владелец корзины |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT now()` | Дата и время создания корзины |

---

### 8. Таблица `cart_items` (Позиции корзины)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `cart_id` | `BIGINT` | `PRIMARY KEY`, `FOREIGN KEY (carts.id) ON DELETE CASCADE ON UPDATE CASCADE` | Корзина |
| `product_id` | `BIGINT` | `PRIMARY KEY`, `FOREIGN KEY (products.id) ON DELETE CASCADE ON UPDATE CASCADE` | Товар, добавленный в корзину |
| `quantity` | `INTEGER` | `NOT NULL`, `DEFAULT 1`, `CHECK (quantity > 0)` | Количество выбранных единиц |

---

### 9. Таблица `orders` (Заказы)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор заказа |
| `user_id` | `BIGINT` | `NOT NULL`, `FOREIGN KEY (users.id) ON DELETE RESTRICT ON UPDATE CASCADE` | Покупатель |
| `status` | `VARCHAR(30)` | `NOT NULL`, `DEFAULT 'created'`, `CHECK (status IN ('created', 'paid', 'shipped', 'delivered', 'cancelled'))` | Текущий статус заказа |
| `total_amount` | `NUMERIC(12,2)` | `NOT NULL`, `DEFAULT 0.00`, `CHECK (total_amount >= 0)` | Общая стоимость заказа |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT now()` | Дата и время оформления |

---

### 10. Таблица `order_items` (Позиции заказа)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `order_id` | `BIGINT` | `PRIMARY KEY`, `FOREIGN KEY (orders.id) ON DELETE CASCADE ON UPDATE CASCADE` | Заказ |
| `product_id` | `BIGINT` | `PRIMARY KEY`, `FOREIGN KEY (products.id) ON DELETE RESTRICT ON UPDATE CASCADE` | Заказанный товар |
| `quantity` | `INTEGER` | `NOT NULL`, `CHECK (quantity > 0)` | Заказанное количество единиц |
| `unit_price` | `NUMERIC(12,2)` | `NOT NULL`, `CHECK (unit_price > 0)` | Фиксированная цена за единицу товара на момент заказа |

---

### 11. Таблица `payments` (Оплата)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор платежа |
| `order_id` | `BIGINT` | `NOT NULL`, `UNIQUE`, `FOREIGN KEY (orders.id) ON DELETE CASCADE ON UPDATE CASCADE` | Оплачиваемый заказ |
| `amount` | `NUMERIC(12,2)` | `NOT NULL`, `CHECK (amount > 0)` | Сумма платежа |
| `status` | `VARCHAR(20)` | `NOT NULL`, `DEFAULT 'pending'`, `CHECK (status IN ('pending', 'completed', 'failed'))` | Статус платежной транзакции |
| `paid_at` | `TIMESTAMPTZ` | — | Дата и время прохождения оплаты |

---

### 12. Таблица `reviews` (Отзывы)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор отзыва |
| `user_id` | `BIGINT` | `NOT NULL`, `FOREIGN KEY (users.id) ON DELETE CASCADE ON UPDATE CASCADE` | Автор отзыва |
| `product_id` | `BIGINT` | `NOT NULL`, `FOREIGN KEY (products.id) ON DELETE CASCADE ON UPDATE CASCADE` | Оцениваемый товар |
| `score` | `INTEGER` | `NOT NULL`, `CHECK (score BETWEEN 1 AND 5)` | Численная оценка товара (от 1 до 5) |
| `comment` | `TEXT` | — | Текстовый отзыв покупателя |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT now()` | Дата и время публикации |

---

### 13. Таблица `deliveries` (Доставка)
| Столбец | Тип данных | Ограничения | Назначение |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Уникальный идентификатор доставки |
| `order_id` | `BIGINT` | `NOT NULL`, `UNIQUE`, `FOREIGN KEY (orders.id) ON DELETE CASCADE ON UPDATE CASCADE` | Доставляемый заказ |
| `address_id` | `BIGINT` | `NOT NULL`, `FOREIGN KEY (addresses.id) ON DELETE RESTRICT ON UPDATE CASCADE` | Выбранный адрес доставки |
| `delivery_status` | `VARCHAR(30)` | `NOT NULL`, `DEFAULT 'processing'`, `CHECK (delivery_status IN ('processing', 'in_transit', 'delivered'))` | Текущий статус отправления |
| `delivery_date` | `TIMESTAMPTZ` | — | Дата и время фактической или планируемой вручения |