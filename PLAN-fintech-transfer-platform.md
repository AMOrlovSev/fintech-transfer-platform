# PLAN fintech-transfer-platform — 8 недель

Трекер задач. Каждая задача = отдельный PR. Definition of Done — в CONTRIBUTING.md.

## Как работать

1. Берёте задачу N.M, открываете новый чат с AI-техлидом.
2. Копируете **мастер-контекст** из MASTER-CONTEXT.md + карточку задачи ниже.
3. Работаете до Done, запрашиваете ревью по формату из MASTER-CONTEXT.md.
4. Ревью APPROVE → merge → двигаетесь дальше.

## Прогресс

| Задача | Est. | Status | PR |
|---|---|---|---|
| T1.1 — Инициализация репозитория | 4–6 ч | ⬜ | |
| T1.2 — docker-compose инфраструктуры | 6–8 ч | ⬜ | |
| T1.3 — Скелет transfer-service | 4 ч | ⬜ | |
| T1.4 — common модуль | 3–4 ч | ⬜ | |
| T2.1 — Домен Account + Flyway | 6–8 ч | ⬜ | |
| T2.2 — AccountService debit/credit | 6–8 ч | ⬜ | |
| T2.3 — REST API account-service | 4–6 ч | ⬜ | |
| T3.1 — Домен Transfer + Idempotency + SagaState | 4–6 ч | ⬜ | |
| T3.2 — createTransfer + идемпотентность | 8–10 ч | ⬜ | |
| T3.3 — Read API transfers | 3–4 ч | ⬜ | |
| T4.1 — Outbox pattern | 6–8 ч | ⬜ | |
| T4.2 — notification-service consumer | 4–6 ч | ⬜ | |
| T4.3 — Saga с компенсациями | 6–8 ч | ⬜ | |
| T5.1 — api-gateway | 4–6 ч | ⬜ | |
| T5.2 — Feign + Resilience4j | 4–6 ч | ⬜ | |
| T6.1 — Keycloak realm | 3–4 ч | ⬜ | |
| T6.2 — Resource Server во всех сервисах | 6–8 ч | ⬜ | |
| T6.3 — prod-render JWT (упрощённый) | 3–4 ч | ⬜ | |
| T7.1 — Prometheus + Grafana | 4–6 ч | ⬜ | |
| T7.2 — Structured logs + MDC | 2–3 ч | ⬜ | |
| T7.3 — OTel + Jaeger | 4–6 ч | ⬜ | |
| T8.1 — CI (GitHub Actions) | 4–6 ч | ⬜ | |
| T8.2 — Render deploy | 6–8 ч | ⬜ | |
| T8.3 — README + ADR + демо | 6–8 ч | ⬜ | |

Статусы: ⬜ todo · 🟡 in progress · 🔵 in review · ✅ done · ⛔ blocked

---

## Week 1 — Фундамент

### T1.1 — Инициализация репозитория

**Goal:** Multi-module Gradle проект, 5 модулей (`common`, `account-service`, `transfer-service`, `notification-service`, `api-gateway`), `.gitignore`, CI собирает.

**Acceptance:**
- `./gradlew build` — BUILD SUCCESSFUL
- Push → CI зелёный
- 5 модулей видны в корне

**Ревью-фокус:** version catalog; `common` не зависит от сервисов; CI на PR.

---

### T1.2 — docker-compose инфраструктуры

**Goal:** `docker compose up -d` → PostgreSQL, Kafka (KRaft), Redis, Keycloak — все healthy за 30–60 сек.

**Acceptance:**
- `docker compose ps` — все healthy
- `psql` подключается, БД созданы
- `kafka-topics --list` работает
- `redis-cli ping` → PONG
- Keycloak UI на 8180

**Ревью-фокус:** KRaft (не ZK); volume для persistence; secrets через `.env`; `wait-for-infra.sh` проверяет готовность.

---

### T1.3 — Скелет transfer-service

**Goal:** `transfer-service` стартует на 8082, `/actuator/health` = UP, логи в JSON.

**Acceptance:**
- `curl :8082/actuator/health` → `{"status":"UP"}`
- liveness и readiness → UP
- Логи — валидный JSON

**Ревью-фокус:** порт из конфига; `problem+json`; JSON-логи с `traceId`.

---

### T1.4 — common модуль

**Goal:** Переиспользуемые `ApiError`, иерархия исключений, `Money`.

**Acceptance:**
- `Money` immutable, value object, `equals/hashCode`
- `subtract` разных валют → `IllegalArgumentException`
- Тест на переполнение (`Long.MAX_VALUE`)
- `transfer-service` использует `common`

**Ревью-фокус:** `long` не `double`; `ApiError` — `problem+json`; нет бизнес-логики.

---

## Week 2 — account-service

### T2.1 — Домен Account + Flyway

**Goal:** Таблица `accounts`, entity с `@Version`, репозиторий, integration-тесты с Testcontainers.

**Acceptance:**
- Flyway применяется при старте
- IT зелёные на реальной PG
- `@Version` инкрементится
- `check` работает (отрицательный баланс → exception)

**Ревью-фокус:** `ddl-auto: validate`; Testcontainers (не H2); `AccountId` в бизнес-коде; индексы.

---

### T2.2 — AccountService debit/credit

**Goal:** Атомарные `debit`/`credit`/`compensateDebit`; баланс не уходит в минус под нагрузкой.

**Acceptance:**
- Concurrency test (10 потоков × 100 при балансе 1000) → 10 успешных, баланс 0
- Нет `double`
- `compensateDebit` идемпотентна
- Транзакции откатываются

**Ревью-фокус:** `@Lock`/`@Modifying UPDATE` (не read-modify-write); нет `synchronized`.

---

### T2.3 — REST API account-service

**Goal:** Эндпоинты из ТЗ, Swagger UI, MockMvc-тесты.

**Acceptance:**
- Swagger UI открывается
- Ошибки — `problem+json`
- `X-Transfer-Id` валидируется (UUID)

**Ревью-фокус:** DTO ≠ entity; `@Valid`; `X-Transfer-Id` обязателен на `/internal/**`.

---

## Week 3 — transfer-service

### T3.1 — Домен Transfer + Idempotency + SagaState

**Goal:** Миграции + entity + репозитории для `transfers`, `idempotency_records`, `saga_state`, `transfer_outbox`.

**Acceptance:**
- Unique index на `transfers.idempotency_key`
- Partial index на `transfers(status) WHERE status='PENDING'`
- Partial index на `outbox(published_at) WHERE published_at IS NULL`

**Ревью-фокус:** partial indexes; `version` в `transfers`/`saga_state`; `failure_reason TEXT`.

---

### T3.2 — createTransfer + идемпотентность

**Goal:** `POST /v1/transfers` с `Idempotency-Key`; replay с тем же ключом → тот же результат; другой body → 422.

**Acceptance:**
- Happy path: 201 COMPLETED, балансы сдвинулись
- Недостаток: 201 FAILED с `failure_reason`
- Replay: тот же `transferId`
- Replay другой body: 422
- 10 concurrent с одним ключом → 1 transfer

**Ревью-фокус:** idempotency в одной транзакции; hash от нормализованных полей; лимит атомарный (`INCRBY`).

---

### T3.3 — Read API transfers

**Goal:** `GET /v1/transfers/{id}`, `GET /v1/transfers` с cursor-пагинацией.

**Acceptance:**
- Cursor стабильный
- Чужой → 403 (или 404 — обосновать)
- Лимит `limit ≤ 100`

**Ревью-фокус:** cursor vs offset; N+1 (EntityGraph).

---

## Week 4 — Outbox, Kafka, Saga

### T4.1 — Outbox pattern

**Goal:** При создании transfer в **той же транзакции** пишется `outbox_event`. Relay публикует в Kafka и помечает `published_at`.

**Acceptance:**
- Транзакция `transfer + outbox` атомарна
- Relay публикует и помечает
- Kafka down → `published_at` null, retry
- Метрика `outbox_pending` видна

**Ревью-фокус:** транзакция атомарна; `acks=all`, `enable.idempotence=true`; Awaitility вместо sleep.

---

### T4.2 — notification-service consumer

**Goal:** Consumer с идемпотентностью, DLQ, ручной commit.

**Acceptance:**
- Событие → уведомление в БД
- Дубликат → skip
- Ошибка → DLQ
- Ручной commit (`enable.auto.commit=false`)

**Ревью-фокус:** `processed_events` unique; ручной commit после обработки; `concurrency` = число партиций.

---

### T4.3 — Saga с компенсациями

**Goal:** При ошибке credit → compensateDebit, transfer FAILED, событие TransferFailed.

**Acceptance:**
- При ошибке credit — debit компенсирован
- TransferFailed в Kafka
- Watchdog находит зависшие
- Повторный запуск идемпотентен

**Ревью-фокус:** компенсация идемпотентна; watchdog не дерётся с активной saga; `SagaState` персистентна; лимит retry.

---

## Week 5 — Gateway + Feign + Resilience4j

### T5.1 — api-gateway

**Goal:** Gateway на 8080 маршрутизирует `/api/accounts/**` и `/api/transfers/**`.

**Acceptance:**
- `curl :8080/api/...` доходит до сервиса
- `X-Request-Id` в логах обоих сервисов
- Не зависает при недоступном backend

**Ревью-фокус:** нет бизнес-логики в gateway; timeout backend'ов.

---

### T5.2 — Feign + Resilience4j

**Goal:** Circuit Breaker, Retry (только GET), Timeout на Feign-клиенте к account.

**Acceptance:**
- CB открывается при 50%
- Fallback отдаёт `ACCOUNT_SERVICE_UNAVAILABLE`
- Timeout 2s работает
- Retry только на GET

**Ревью-фокус:** retry только на GET; CB на read и write, но retry разный; метрики Resilience4j.

---

## Week 6 — Security

### T6.1 — Keycloak realm

**Goal:** Realm `fintech-transfer-platform` с ролями и клиентами, экспорт в репо.

**Acceptance:**
- Token с `realm_access.roles`
- Роли `USER/ADMIN/SERVICE` есть
- Тестовые users работают

**Ревью-фокус:** realm в репо; dev-пароли помечены; `ft-backend` confidential.

---

### T6.2 — Resource Server

**Goal:** Все `/v1/**` и `/admin/**` за JWT; `@PreAuthorize` по ролям; `sub` = userId.

**Acceptance:**
- Без токена 401
- USER на `/admin/**` → 403
- SERVICE на `/internal/**` → 200
- Чужой → 403

**Ревью-фокус:** `denyAll()` default; `sub` = userId (не body); `/actuator/**` закрыт кроме health.

---

### T6.3 — prod-render JWT (упрощённый)

**Goal:** Профиль `prod-render` — самописный JWT (HS256), `POST /auth/login`.

**Acceptance:**
- `POST /auth/login` → JWT
- JWT работает
- Default (dev) — Keycloak

**Ревью-фокус:** профили не пересекаются; secret из env; README про упрощение.

---

## Week 7 — Observability

### T7.1 — Prometheus + Grafana

**Goal:** Все сервисы `/actuator/prometheus`, дашборд RED + JVM + Kafka + Outbox.

**Acceptance:**
- Grafana :3000, дашборд открывается
- RPS, errors, p95/p99 видны
- `outbox_pending` gauge
- Дашборд — JSON в репо

**Ревью-фокус:** нет cardinally-bad labels; histogram buckets осмысленные; алерты ≥ 2.

---

### T7.2 — Structured logs + MDC

**Goal:** Логи в JSON во всех сервисах, `traceId`/`spanId`/`userId` в MDC.

**Acceptance:**
- `traceId` одинаков в логах всех сервисов для одного запроса
- JSON валидный
- Нет PII/секретов

**Ревью-фокус:** `MDC.clear()` в finally; нет `System.out`.

---

### T7.3 — OTel + Jaeger

**Goal:** Трейс от `POST /v1/transfers` виден в Jaeger.

**Acceptance:**
- Трейс в Jaeger содержит все спаны
- Kafka-хоп виден
- `traceId` Jaeger = `traceId` в логах

**Ревью-фокус:** sampling не 100% в prod (для dev ок); нет PII в span.

---

## Week 8 — CI/CD, Render, Docs

### T8.1 — CI

**Goal:** На PR — build + test. На main — docker build + push GHCR.

**Acceptance:**
- CI < 10 мин
- Testcontainers в GH Actions работают
- Образы в GHCR
- Badge в README

**Ревью-фокус:** secrets через GH Secrets; branch protection на main.

---

### T8.2 — Render deploy

**Goal:** Живой деплой: web (transfer), private (account), PostgreSQL, Upstash Kafka.

**Acceptance:**
- `/actuator/health` = UP на Render
- Полный перевод через Swagger UI
- README ссылка

**Ревью-фокус:** free-tier ограничения задокументированы; secrets не в git; `prod-render` не ломает dev.

---

### T8.3 — Docs + ADR + Demo

**Goal:** README «продаёт», 4 ADR, диаграмма, демо-видео.

**Acceptance:**
- README читается за 5 мин
- 4 ADR с Context/Decision/Consequences
- Диаграмма
- Видео

**Ревью-фокус:** README про **решения** (не «я использовал Kafka»); ADR с trade-offs; ограничения честно.
