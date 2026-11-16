# TZ fintech-transfer-platform — Fintech Transfer Platform (конспект)

> Полная версия ТЗ — в чате разработки (сообщение «Техническое задание»). Эта страница — конспект для быстрого доступа.

## Суть

Платформа переводов между счетами пользователей. Упрощённый, но архитектурно честный аналог внутреннего финтех-сервиса. Pet-проект для портфолио Middle.

## Функциональные требования (MVP)

Роли: `USER`, `ADMIN`.

Ключевые сценарии:
- Просмотр своих счетов и балансов
- Создание перевода между счетами с `Idempotency-Key`
- Просмотр статуса и истории переводов
- Асинхронные уведомления об успехе/неуспехе
- Админ: просмотр всех счетов/переводов, отмена PENDING, эмуляция пополнения

## NFR

- RPS (write): 50–100, RPS (read): 200–500
- p99 `POST /v1/transfers` < 500 ms до `PENDING`, < 3 s до `COMPLETED`
- SLO: доступность 99.5%, error rate 5xx < 0.1%
- Доставка событий: at-least-once, дедупликация на консьюмере

## Архитектура

| Сервис | Ответственность | Своя БД |
|---|---|---|
| `api-gateway` | Роутинг, JWT на периферии | нет |
| `account-service` | Источник истины по счетам | `accounts` |
| `transfer-service` | Оркестратор, Saga, Outbox, идемпотентность | `transfers`, `outbox`, `saga_state`, `idempotency` |
| `notification-service` | Асинхронные уведомления | `notifications`, `processed_events` |

Инфраструктура: PostgreSQL 16, Kafka (KRaft), Redis 7, Keycloak, Prometheus, Grafana, Jaeger.

## Ключевые паттерны

- **Saga (orchestration)** — распределённая транзакция с компенсациями
- **Outbox** — атомарность «БД + событие»
- **Idempotency-Key** — защита от дублей при retry
- **CQRS-lite** — read-модель в transfer-service
- **API Gateway** — единая точка входа
- **Circuit Breaker** — защита от каскадного отказа
- **Optimistic Locking** — на accounts и transfers

## API (high-level)

`account-service`: `GET /v1/accounts`, `GET /v1/accounts/{id}`, `POST /internal/accounts/{id}/debit|credit|compensate-debit`, `POST /admin/accounts/{id}/top-up`

`transfer-service`: `POST /v1/transfers` (с `Idempotency-Key`), `GET /v1/transfers/{id}`, `GET /v1/transfers`, `POST /v1/transfers/{id}/cancel`

`notification-service`: `GET /v1/notifications`, `GET /v1/notifications/{id}`

Деньги — в минорных единицах (`long`, копейки), не `double`. Ошибки — RFC 7807 (`application/problem+json`). Пагинация — cursor-based. Версионирование — `/v1/`.

## Вне scope v1

Мультивалютные конвертации, fraud detection, KYC, мобильный/веб-клиент, multi-region, Event Sourcing, GraphQL.
