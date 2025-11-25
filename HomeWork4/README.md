### Архитектурная ката [Going Green](http://nealford.com/katas/kata?id=GoingGreen "Going Green")

#### 1. **Кейс**

За основу взят кейс из [предыдущего домашнего задания](https://github.com/Anton-Grebenkin/GoingGreen/tree/main/HomeWork1) 

#### 2. **Диаграмма развертывания**
Вся инфраструктура размещена в одном ЦОД-е. Сервисы разворачиваются в k8s кластере и общаются между собой по http, кластер СУБД Postgreql разворачивается отдельно. У каждого сервиса отдельная база данных внутри кластера. Взаимодействия с сервисами из вне осуществляются с помощью API Gateway.

![enter image description here](https://github.com/Anton-Grebenkin/GoingGreen/blob/main/HomeWork4/hm4.png)

#### 3. **Диаграмма контейнеров**
![enter image description here](https://github.com/Anton-Grebenkin/GoingGreen/blob/main/HomeWork4/4.png)

#### 4. **Декомпозиция слоя данных**
##### 1. Devices Db
Отвечает за данные об устройствах, которые компания готова принять.

**Таблица: `categories`** 
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор категории |
| `name` | VARCHAR | Название категории |
| `description` | TEXT | Описание |

**Таблица: `manufacturers`** 
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор производителя |
| `name` | VARCHAR | Название |

**Таблица: `device_types`** 
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор модели |
| `manufacturer_id` | UUID (FK) | Ссылка на производителя |
| `category_id` | UUID (FK) | Ссылка на категорию |
| `model_name` | VARCHAR | Название модели |
| `specs` | JSONB | Технические характеристики |
| `image_url` | VARCHAR | Ссылка на изображение |
| `is_active` | BOOLEAN | Принимаем ли мы сейчас этот товар |

---

##### 2. Customers Db
Хранит данные пользователей.

**Таблица: `customers`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор клиента |
| `email` | VARCHAR | email |
| `password_hash` | VARCHAR | Хеш пароля |
| `first_name` | VARCHAR | Имя |
| `last_name` | VARCHAR | Фамилия |
| `phone` | VARCHAR | Телефон |
| `created_at` | TIMESTAMP | Дата регистрации |

**Таблица: `customer_addresses`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор адреса |
| `customer_id` | UUID (FK) | Ссылка на клиента |
| `city` | VARCHAR | Город |
| `address_line` | VARCHAR | Улица, дом |
| `zip_code` | VARCHAR | Индекс |
| `is_default` | BOOLEAN | Основной адрес для доставки |

---

##### 3. Quoting Db
Хранит правила расчета стоимости.

**Таблица: `quoting_rules`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор правила |
| `device_type_id` | UUID | Ссылка на Device Catalog |
| `condition_grade` | VARCHAR | Состояние |
| `base_price` | NUMERIC | Базовая цена выкупа |
| `currency` | VARCHAR(3) | Валюта |
| `valid_from` | TIMESTAMP | Начало действия цены |
| `valid_to` | TIMESTAMP | Конец действия |

---

##### 4. Applications Db
Хранит информацию о заявках клиента

**Таблица: `applications`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор заявки |
| `customer_id` | UUID | Ссылка на Customer Mgmt |
| `device_type_id` | UUID | Ссылка на Device Catalog |
| `device_snapshot` | JSONB | Копия названия устройства  |
| `declared_condition`| VARCHAR | Состояние, заявленное клиентом |
| `quoted_price` | NUMERIC | Цена, предложенная при создании  |
| `status` | VARCHAR | Статус  |
| `created_at` | TIMESTAMP | Дата создания |
| `updated_at` | TIMESTAMP | Дата последнего изменения |

---

##### 5. Logistic Db
Управляет доставкой

**Таблица: `shipments`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор отправления |
| `application_id` | UUID | Ссылка на заявку |
| `tracking_code` | VARCHAR | Трек номер внешней системы доставки |
| `carrier` | VARCHAR | Перевозчик |
| `direction` | VARCHAR | INBOUND (от клиента), OUTBOUND (возврат) |
| `status` | VARCHAR | Статус доставки |
| `shipped_at` | TIMESTAMP | Когда передано в доставку |
| `received_at` | TIMESTAMP | Когда получено на склад |

---

##### 6. Evaluation Db
Используется сотрудниками склада для проверки реального состояния.

**Таблица: `evaluation_rules`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор набора правил |
| `device_type_id` | UUID | К какому типу устройства относится правило |
| `checklist` | JSONB | Список критериев проверки |

**Таблица: `evaluations`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор оценки |
| `application_id` | UUID | Ссылка на заявку |
| `inspector_id` | UUID | ID сотрудника склада |
| `actual_condition` | VARCHAR | Реальное состояние после проверки |
| `final_price` | NUMERIC | Итоговая цена |
| `check_results` | JSONB | Результат |
| `notes` | TEXT | Комментарий оценщика |
| `completed_at` | TIMESTAMP | Время завершения оценки |

---

##### 7. Payments Db
Отвечает за перечисление денег клиенту

**Таблица: `payment_methods`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор метода |
| `customer_id` | UUID | Ссылка на клиента |
| `type` | VARCHAR | Тип оплаты |
| `details_encrypted` | BYTEA | Зашифрованные реквизиты |

**Таблица: `payouts`**
| Поле | Тип | Описание |
|---|---|---|
| `id` | UUID (PK) | Идентификатор транзакции |
| `application_id` | UUID | За что платим |
| `amount` | NUMERIC | Сумма |
| `currency` | VARCHAR | Валюта |
| `status` | VARCHAR | Статус выплаты |
| `external_ref_id` | VARCHAR | ID транзакции в платежном шлюзе |
| `processed_at` | TIMESTAMP | Когда платеж прошел |
