# 🎬 kirt08-cinema-project

**kirt08-cinema-project** — это полноценный бэкенд для сети кинотеатров, построенный на микросервисной архитектуре. Проект реализует полный цикл работы кинотеатра: от авторизации пользователей и каталога фильмов до составления расписания сеансов, бронирования мест и оплаты билетов.

Проект разработан как практическая платформа для изучения современных технологий и архитектурных подходов, включая применение Domain-Driven Design (DDD), организацию межсервисного взаимодействия с использованием gRPC, а также реализацию асинхронной обработки событий через RabbitMQ.

Language|files|blank|comment|code
:-------|-------:|-------:|-------:|-------:
Python|372|2531|544|8699
YAML|17|100|2|582
TOML|13|28|0|246
INI|2|52|1|245
Text|1|36|0|115
Dockerfile|5|70|20|80
HTML|2|0|0|53
Mako|1|8|0|20
JSON|4|0|0|12
Bourne Shell|1|3|0|10
SQL|1|0|0|6
Markdown|3|0|0|3
--------|--------|--------|--------|--------
SUM:|422|2828|567|10071

## 🛠 Технологический стек

* **Язык и фреймворки:** Python 3.12, FastAPI, aiogram (Telegram Bot).
* **Архитектура:** Microservices, Domain-Driven Design (DDD), gRPC, REST API (Gateway).
* **Базы данных и кэш:** PostgreSQL (asyncpg, SQLAlchemy, Alembic), MongoDB (Beanie), Redis.
* **Брокер сообщений:** RabbitMQ (aio-pika).
* **Инфраструктура и контейнеризация:** ngrok (для локального тестирования Webhooks от ЮKassa), Docker, Docker Compose.
* **Observability (Мониторинг и логирование):** 
  * OpenTelemetry & Jaeger (Distributed Tracing)
  * Prometheus & Grafana (Metrics & Dashboards)
  * Loki & Promtail (Centralized Logging)
* **Пакетный менеджер:** Poetry.
* **CI/CD:** GitHub Actions (автоматический деплой библиотек в PyPi и PyPi-test).

---

## 🏗 Инфраструктура и Observability (Docker)

Все микросервисы и элементы инфраструктуры оркестрируются с помощью `docker-compose`. В проекте настроен стек мониторинга, который позволяет отслеживать состояние системы в реальном времени.

* **Базы данных:** PostgreSQL, MongoDB, Redis (с RedisInsight GUI).
* **Очереди:** RabbitMQ с Management Plugin.
* **Метрики:** Prometheus собирает данные с `postgres-exporter`, `redis-exporter`, `rabbitmq-exporter` и самих микросервисов.
* **Трейсинг:** Jaeger собирает распределенные трейсы через OpenTelemetry.

  <img width="3829" height="2169" alt="image" src="https://github.com/user-attachments/assets/e74006da-79a2-44b3-8588-f950d915e5ad" />
  *Пример распределенной трассировки запроса в Jaeger*

 **Логирование:** Promtail собирает логи сервисов (`auth-service`) и отправляет их в Loki для визуализации в Grafana.
 
  <img width="3829" height="2169" alt="image" src="https://github.com/user-attachments/assets/444b2541-9d03-42e2-93b4-20619203cf9f" />

  <img width="3829" height="2169" alt="image" src="https://github.com/user-attachments/assets/c56a2f84-c8d1-42dd-9610-b29dabd84f77" />
  *Дашборд Grafana с логами и метриками*

---

## 📦 Структура микросервисов
```
└── backend
    ├── auth-service
    ├── booking-service
    ├── docker
    ├── gateway-service
    ├── kirt08_contracts
    ├── kirt08_exceptions
    ├── kirt08_tokens
    ├── media-service
    ├── movie-service
    ├── notification-service
    ├── payment-service
    ├── screening-service
    ├── telegrambot-service
    ├── theater-service
    └── users-service
```
Взаимодействие между сервисами происходит по протоколу **gRPC** (кроме общения Gateway с клиентами по REST). Порты для локальной разработки и Docker-окружения задаются через файлы `.env.development.local` и `.env.production.local`.

### 1. Gateway (API Gateway)
Единая точка входа для всех клиентских приложений. Принимает REST/HTTP запросы и маршрутизирует их в соответствующие микросервисы по gRPC.
* **Особенности:** 
  * Настроен глобальный Exception Handler (через собственную библиотеку `kirt08_exceptions`), который перехватывает gRPC ошибки и конвертирует их в читаемый JSON с правильными HTTP статусами.
  * Интегрирован с OpenTelemetry для сквозного трейсинга запросов.
  * Экспортирует бизнес-метрики в Prometheus.
* **Схема ручек (Endpoints):**
  <img width="3206" height="1596" alt="image" src="https://github.com/user-attachments/assets/35ae1bab-6c67-4c21-8029-13a2ab677496" />

  <img width="3206" height="1725" alt="image" src="https://github.com/user-attachments/assets/732a88ea-4d0b-4ada-a021-f03b84431379" />

  <img width="3206" height="1859" alt="image" src="https://github.com/user-attachments/assets/11d2f882-ff55-4e3c-8184-3e693a85eadc" />

  <img width="3206" height="1457" alt="image" src="https://github.com/user-attachments/assets/fe5a67a7-1938-45fc-bd22-15bb98b97e6f" />

* **Структура директорий:**
  ```
  .
  ├── docker-compose.yaml
  ├── Dockerfile
  ├── poetry.lock
  ├── pyproject.toml
  └── src
      ├── apps
      │   ├── account
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   ├── schemas.py
      │   │   └── service.py
      │   ├── auth
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   ├── schemas.py
      │   │   ├── service.py
      │   │   └── utils.py
      │   ├── booking
      │   │   ├── __init__.py
      │   │   └── router.py
      │   ├── categories
      │   │   ├── __init__.py
      │   │   └── router.py
      │   ├── grpc_clients.py
      │   ├── halls
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   └── shemas.py
      │   ├── __init__.py
      │   ├── movie
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   └── shemas.py
      │   ├── payment
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   └── schemas.py
      │   ├── refund
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   └── schemas.py
      │   ├── screening
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   └── schemas.py
      │   ├── seats
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   └── shemas.py
      │   ├── shared
      │   │   ├── __init__.py
      │   │   ├── schemas.py
      │   │   └── service.py
      │   ├── theaters
      │   │   ├── __init__.py
      │   │   ├── router.py
      │   │   └── shemas.py
      │   └── users
      │       ├── __init__.py
      │       ├── router.py
      │       └── shemas.py
      └── core
          ├── config
          │   ├── authConfig.py
          │   ├── cookiesCongig.py
          │   ├── __init__.py
          │   ├── loggerConfig.py
          │   ├── mediaConfig.py
          │   ├── movieConfig.py
          │   ├── passportConfig.py
          │   ├── paymentConfig.py
          │   ├── screeningConfig.py
          │   ├── settings.py
          │   ├── theaterConfig.py
          │   └── usersConfig.py
          ├── grpc_clients
          │   ├── account.py
          │   ├── auth.py
          │   ├── category.py
          │   ├── halls.py
          │   ├── __init__.py
          │   ├── media.py
          │   ├── movie.py
          │   ├── payment.py
          │   ├── refund.py
          │   ├── screening.py
          │   ├── seats.py
          │   ├── theaters.py
          │   └── user.py
          ├── main.py
          ├── prometheus_metrics.py
          ├── schemas.py
          ├── service.py
          ├── tracing.py
          └── utils.py

  ```

### 2. Auth-Service `[Port: 50051]`
Сервис аутентификации и авторизации. Отвечает за регистрацию пользователей в системе, выдает и валидирует токены доступа.
* **База данных:** PostgreSQL + Redis.
* **Логика работы:** Вход осуществляется через 6-значный OTP-код. Код генерируется, хэшируется и кэшируется в Redis. 
* *Примечание:* Изначально система проектировалась для отправки SMS-кодов, однако из-за региональных ограничений провайдеров реализован fallback на отправку кодов по Email (через SMTP) через отправку событий в RabbitMQ.
* **Логирование:** Настроена детальная отправка логов в Loki в формате JSON (`python-json-logger`).

### 3. Telegrambot-Service
Сервис-интерфейс для альтернативного входа и взаимодействия с платформой через Telegram. 
* Написан на `aiogram`. 
* Общается с Gateway и Auth-service (использует логику генерации OTP-кодов) по gRPC.

### 4. Users-Service `[Port: 50052]`
Сервис управления профилями пользователей (ник, имя, фамилия, аватар).
* Требует авторизации (токены валидируются через библиотеку `kirt08-tokens`).
* **База данных:** PostgreSQL.

### 5. Notification-Service
Асинхронный сервис отправки уведомлений (Email/SMS).
* **Стек:** RabbitMQ, aiosmtplib, Jinja2.
* Слушает очереди RabbitMQ. Гарантирует доставку сообщений за счет механизмов `ack`/`nack`.
* Письма верстаются динамически с использованием шаблонов Jinja2. Для тестирования доставки писем используется Mailtrap. Экспортирует метрики рассылок в Prometheus.

### 6. Movie-Service `[Port: 50053]`
Сервис каталога фильмов.
* Отвечает за предоставление информации о фильмах.
* Взаимодействует с отдельным Golang-сервисом (Media-service, `Port: 50059`) для работы с файлами в S3 хранилище.
* Для ускорения отдачи списков фильмов реализовано кэширование на базе Redis.

### 7. Theater-Service `[Port: 50054]`
Управление кинотеатрами, залами и схемами рассадки.
* **Особенности архитектуры:** Спроектирован с применением **Domain-Driven Design (DDD)**. Четкое разделение на слои (Domain, Application, Infrastructure), изоляция бизнес-логики.
* **База данных:** PostgreSQL.
* **Структура директорий:**
  ```
  .
  ├── alembic
  │   ├── env.py
  │   ├── README
  │   ├── script.py.mako
  │   └── versions
  │       ├── 63b228d97cde_initial.py
  │       └── cc5197a7d894_add_new.py
  ├── alembic.ini
  ├── docker-compose.yaml
  ├── Dockerfile
  ├── poetry.lock
  ├── pyproject.toml
  └── src
      ├── application
      │   ├── __init__.py
      │   └── usecases
      │       ├── hall
      │       │   ├── create_hall.py
      │       │   ├── get_hall.py
      │       │   ├── __init__.py
      │       │   └── list_halls.py
      │       ├── seat
      │       │   ├── get_seat.py
      │       │   ├── __init__.py
      │       │   └── list_seats.py
      │       └── theater
      │           ├── create_theater.py
      │           ├── get_theater.py
      │           ├── __init__.py
      │           └── list_theaters.py
      ├── domain
      │   ├── entities
      │   │   ├── hall.py
      │   │   ├── __init__.py
      │   │   ├── seat.py
      │   │   └── theater.py
      │   ├── exceptions
      │   │   ├── hall.py
      │   │   ├── __init__.py
      │   │   ├── seat.py
      │   │   └── theater.py
      │   ├── __init__.py
      │   └── repositories
      │       ├── hall_repository.py
      │       ├── __init__.py
      │       ├── seats_repository.py
      │       └── theater_repository.py
      ├── infrastructure
      │   ├── config
      │   │   ├── dbConfig.py
      │   │   ├── grpcConfig.py
      │   │   ├── __init__.py
      │   │   ├── loggerConfig.py
      │   │   └── settings.py
      │   ├── db
      │   │   ├── database.py
      │   │   ├── hall_repository_sqlalchemy.py
      │   │   ├── __init__.py
      │   │   ├── models
      │   │   │   ├── base_model.py
      │   │   │   ├── halls.py
      │   │   │   ├── __init__.py
      │   │   │   ├── seats.py
      │   │   │   └── theaters.py
      │   │   ├── seat_repository_sqlalchemy.py
      │   │   └── theater_repository_sqlalchemy.py
      │   └── __init__.py
      ├── main.py
      └── presentation
          ├── grpc
          │   ├── hall_controller.py
          │   ├── __init__.py
          │   ├── seat_controller.py
          │   ├── server.py
          │   ├── theater_controller.py
          │   └── theater_mapped.py
          └── __init__.py
  ```

### 8. Screening-Service `[Port: 50055]`
Управление расписанием сеансов.
* **Особенности архитектуры:** Второй сервис в проекте, реализованный по методологии **Domain-Driven Design (DDD)**.
* **База данных:** MongoDB. Для асинхронной работы и строгой типизации (Pydantic) используется ODM `Beanie`.
* **Структура директорий:**
  ```
  .
  ├── docker-compose.yaml
  ├── Dockerfile
  ├── poetry.lock
  ├── pyproject.toml
  └── src
      ├── __init__.py
      ├── main.py
      ├── screening
      │   ├── application
      │   │   ├── create_usecase.py
      │   │   ├── get_by_movie_usecase.py
      │   │   ├── get_screenings_usecase.py
      │   │   ├── get_screening_usecase.py
      │   │   └── __init__.py
      │   ├── domain
      │   │   ├── entities
      │   │   │   ├── hall.py
      │   │   │   ├── __init__.py
      │   │   │   ├── movie.py
      │   │   │   ├── screening.py
      │   │   │   ├── seat.py
      │   │   │   └── theater.py
      │   │   ├── exceptions
      │   │   │   ├── __init__.py
      │   │   │   └── screening_exceptions.py
      │   │   ├── __init__.py
      │   │   └── repositories
      │   │       ├── hall.py
      │   │       ├── __init__.py
      │   │       ├── movie.py
      │   │       ├── screening.py
      │   │       ├── seat.py
      │   │       └── theater.py
      │   ├── infrastructure
      │   │   ├── config
      │   │   │   ├── grpcConfig.py
      │   │   │   ├── __init__.py
      │   │   │   ├── loggerConfig.py
      │   │   │   ├── mongoConfig.py
      │   │   │   └── settings.py
      │   │   ├── db
      │   │   │   ├── database.py
      │   │   │   ├── __init__.py
      │   │   │   ├── mappers
      │   │   │   │   ├── __init__.py
      │   │   │   │   └── screening_mapper.py
      │   │   │   ├── repository
      │   │   │   │   ├── __init__.py
      │   │   │   │   └── screening_mongo.py
      │   │   │   └── shemas
      │   │   │       ├── __init__.py
      │   │   │       └── screening.py
      │   │   ├── grpc
      │   │   │   ├── hall_adapter.py
      │   │   │   ├── hall_mapper.py
      │   │   │   ├── __init__.py
      │   │   │   ├── movie_adapter.py
      │   │   │   ├── movie_mapper.py
      │   │   │   ├── seat_adapter.py
      │   │   │   ├── theater_adapter.py
      │   │   │   └── theater_mapper.py
      │   │   └── __init__.py
      │   ├── __init__.py
      │   └── interfaces
      │       ├── __init__.py
      │       ├── mapper.py
      │       ├── screening_controller.py
      │       └── server.py
      └── shared
          ├── __init__.py
          └── utils
              ├── __init__.py
              ├── resolve_day.py
              └── to_datetime.py
  ```

### 9. Payment-Service `[Port: 50056]`
Сервис проведения оплат и возвратов.
* Интегрирован с ЮKassa (через open-source библиотеку `aioyookassa`).
* Webhooks: Реализована обработка уведомлений об изменении статуса платежа.
* Dev-Ops: Для обеспечения доступности локального сервера внешним запросам от ЮKassa использован ngrok, что позволило полноценно тестировать сценарии успешной оплаты и отмены в среде разработки.
* Ведет надежный учет транзакций в PostgreSQL для исключения потери данных о платежах.

### 10. Booking-Service `[Port: 50057]`
Сервис бронирования билетов и мест в зале.
* Взаимодействует с Theater, Screening и Payment сервисами по gRPC для обеспечения консистентности брони (проверка свободных мест -> холд -> оплата -> выпуск билета).
* **База данных:** PostgreSQL.

---

## 📚 Custom Libraries (Самописные библиотеки)

Для устранения дублирования кода и обеспечения единых контрактов взаимодействия, общая логика вынесена в отдельные пакеты. Все библиотеки автоматически публикуются в PyPI / Test-PyPI с помощью настроенных **GitHub Actions CI/CD workflows**.

1. **`kirt08-contracts`** 
   * **[https://pypi.org/project/kirt08_contracts/]**
   * Репозиторий с описанием Protobuf контрактов. Содержит сгенерированные классы и типы Python для gRPC-взаимодействия между всеми микросервисами.
2. **`kirt08-tokens`**
   * **[https://pypi.org/project/kirt08-tokens/]**
   * Инкапсулирует логику создания, валидации и декодирования собственных токенов (собственная реализация jwt токена для углубленного изучения)/Auth токенов. Используется в Gateway, Auth и Users сервисах.
3. **`kirt08-exceptions`**
   * **[https://pypi.org/project/kirt08-exceptions/]**
   * Библиотека для маппинга gRPC исключений. Позволяет Gateway элегантно перехватывать ошибки внутренних сервисов и отдавать клиенту правильный REST-ответ с нужным HTTP-кодом.

---

## 🚀 Запуск проекта

1. Клонируйте репозиторий:
   ```bash
   git clone https://github.com/kirt08-cinema-tickets-project
   cd kirt08-cinema-project

2. Настройте переменные окружения:
   - Скопируйте примеры env-файлов и заполните их актуальными данными (пароли, API-ключи ЮKassa, SMTP).
   - Файлы `.env.production.local` используются для запуска в Docker.
   - Файлы `.env.development.local` используется для ручного запуска, например через `poetry run python3 -m src.main`

3. Настройка внешних интеграций (ЮKassa)
   Для работы с платежами в режиме разработки:
   Запустите `ngrok: ngrok http 8000` (порт вашего Gateway).
   
4. Запустите инфраструктуру и сервисы:
   ```docker
   cd docker
   docker-compose up -d --build
   ```
   
5. Для каждого из микросервисов написан собственный docker-compose
   `docker compose up --build -d`
