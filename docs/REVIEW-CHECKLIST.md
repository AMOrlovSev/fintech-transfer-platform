# REVIEW-CHECKLIST

Используйте в запросе на ревью: «Пройди Review Checklist для T_N.M».

## Общий чек-лист (любой PR)

- [ ] `./gradlew build` — SUCCESS
- [ ] Все тесты зелёные
- [ ] Нет TODO без ссылки на issue
- [ ] Нет `System.out.println`, `printStackTrace`
- [ ] Нет секретов / PII / токенов в коде и логах
- [ ] Commit message по Conventional Commits
- [ ] PR ≤ 400 строк diff (не считая тестов)
- [ ] README / ADR обновлены (если применимо)
- [ ] `docker compose up` — всё поднимается
- [ ] DoD выполнен (см. CONTRIBUTING.md)

## Week 1

### T1.1 — Инициализация
- [ ] Version catalog используется (нет хардкода версий)
- [ ] `common` не зависит от сервисов
- [ ] CI на `push` и `pull_request`
- [ ] `.gitignore` закрывает `.env`, `*.log`, `build`, `.idea`
- [ ] README ссылается на TZ и PLAN
- [ ] `gradlew` закоммичен (wrapper jar)

### T1.2 — docker-compose
- [ ] Kafka в KRaft (не ZooKeeper)
- [ ] Volume для PG и Kafka
- [ ] Healthcheck у всех сервисов
- [ ] Нет паролей — через `.env`
- [ ] `wait-for-infra.sh` проверяет готовность (не «running»)
- [ ] Два listener'а Kafka (9092 / 29092)

### T1.3 — Скелет transfer-service
- [ ] Порт из `application.yml`, не хардкод
- [ ] Actuator: health, liveness, readiness, prometheus
- [ ] JSON-логи с `traceId` (пусть null пока)
- [ ] `GlobalExceptionHandler` → `problem+json`
- [ ] `HealthIT` зелёный

### T1.4 — common
- [ ] `Money` immutable, `equals/hashCode`
- [ ] `subtract` разных валют → `IllegalArgumentException`
- [ ] Тест переполнения (`Long.MAX_VALUE`)
- [ ] `ApiError` — record, `problem+json`
- [ ] Нет бизнес-логики

## Week 2

### T2.1 — Домен Account
- [ ] `ddl-auto: validate`
- [ ] Testcontainers PostgreSQL (не H2)
- [ ] `AccountId` — value object, не `long`
- [ ] Index на `owner_id`
- [ ] `Money` из common
- [ ] `CHECK balance_minor >= 0`

### T2.2 — AccountService
- [ ] Атомарность: `@Lock(PESSIMISTIC_WRITE)` или `@Modifying UPDATE`
- [ ] Concurrency test ловит баг (убрать lock → красный)
- [ ] `compensateDebit` идемпотентна по `transferId`
- [ ] Нет `synchronized`
- [ ] `@Transactional` с явной изоляцией

### T2.3 — REST API account
- [ ] DTO ≠ entity
- [ ] `@Valid` + Bean Validation
- [ ] `X-Transfer-Id` обязателен на `/internal/**` (UUID)
- [ ] Ошибки — `problem+json`
- [ ] Swagger UI на `/swagger-ui.html`

## Week 3

### T3.1 — Домен Transfer
- [ ] Partial index на `transfers(status) WHERE status='PENDING'`
- [ ] Unique index на `transfers.idempotency_key`
- [ ] Partial index на `outbox(published_at) WHERE published_at IS NULL`
- [ ] `version` в `transfers` и `saga_state`
- [ ] `failure_reason TEXT`
- [ ] `completed_at` nullable

### T3.2 — createTransfer
- [ ] **Идемпотентность в одной транзакции** (unique constraint)
- [ ] `hash` от нормализованных полей (не JSON.stringify)
- [ ] Feign с явным URL и fallback
- [ ] Feign ошибки обрабатываются
- [ ] Лимит в Redis атомарный (`INCRBY`)
- [ ] 10 параллельных POST с одним ключом → 1 transfer
- [ ] Replay с другим body → 422

### T3.3 — Read API transfers
- [ ] Cursor = base64(`created_at`|`id`), не offset
- [ ] N+1 нет (`EntityGraph` или `join fetch`)
- [ ] Тест на «чужой» ресурс (403 или 404 — обосновано)
- [ ] `limit ≤ 100`

## Week 4

### T4.1 — Outbox
- [ ] Транзакция `transfer + outbox` атомарна
- [ ] Relay публикует и помечает `published_at`
- [ ] Relay идемпотентен (at-least-once)
- [ ] `acks=all`, `enable.idempotence=true`
- [ ] Метрика `outbox_pending_gauge`
- [ ] Тест: Kafka down → создать → Kafka up → доехало
- [ ] Awaitility вместо sleep

### T4.2 — notification-service
- [ ] `processed_events(event_id)` unique
- [ ] `processed_events + notifications` в одной транзакции
- [ ] Ручной commit (`enable.auto.commit=false`)
- [ ] DLQ для poison pill
- [ ] `concurrency` = число партиций (3)
- [ ] `max.poll.interval.ms` достаточный

### T4.3 — Saga
- [ ] Компенсация идемпотентна по `transferId`
- [ ] `SagaState` персистентна (переживает рестарт)
- [ ] Watchdog не «дерётся» с активной Saga
- [ ] Retry с backoff
- [ ] Лимит retry + `FAILED`
- [ ] Событие `TransferFailed` публикуется

## Week 5

### T5.1 — api-gateway
- [ ] Нет бизнес-логики в Gateway
- [ ] Timeout на backend
- [ ] `traceId` в логах
- [ ] `curl :8080/api/...` доходит

### T5.2 — Feign + Resilience4j
- [ ] **Retry только на GET** (не на POST `debit`!)
- [ ] Circuit Breaker с осмысленным порогом
- [ ] Fallback не «глотает» ошибку
- [ ] Метрики Resilience4j в Prometheus
- [ ] Timeout 2s работает

## Week 6

### T6.1 — Keycloak realm
- [ ] Realm **в репо** (воспроизводимость)
- [ ] Dev-пароли помечены в README
- [ ] `ft-backend` — confidential (есть secret)
- [ ] Service account включён

### T6.2 — Resource Server
- [ ] **`denyAll()` как default**
- [ ] `sub` = `userId` (не из body)
- [ ] `/actuator/health` открыт, `/actuator/**` закрыт
- [ ] CSRF disabled (stateless JWT)
- [ ] Роли из `realm_access.roles`
- [ ] Чужой ресурс → 403

### T6.3 — prod-render JWT
- [ ] Профиль `prod-render` не пересекается с dev
- [ ] Secret из env
- [ ] README про упрощение
- [ ] Нет bypass auth

## Week 7

### T7.1 — Prometheus + Grafana
- [ ] Нет cardinally-bad labels (не `userId` в метках!)
- [ ] Histogram buckets осмысленные
- [ ] Дашборд — JSON в репо
- [ ] Алерты ≥ 2 (error rate, outbox backlog)
- [ ] `outbox_pending_gauge` есть

### T7.2 — Structured logs
- [ ] `MDC.clear()` в `finally`
- [ ] Нет `System.out`
- [ ] Logger по классу (не `root`)
- [ ] `INFO`/`WARN`/`ERROR` осмысленно
- [ ] Нет PII/секретов

### T7.3 — OTel + Jaeger
- [ ] Sampling не 100% в prod (dev — ок)
- [ ] Нет PII в span
- [ ] Feign interceptor с `traceparent`
- [ ] Kafka headers с `traceparent`

## Week 8

### T8.1 — CI
- [ ] Secrets через GH Secrets
- [ ] `pull_request` триггерит CI
- [ ] Branch protection на `main`
- [ ] Кэш Gradle работает
- [ ] Testcontainers в GH Actions

### T8.2 — Render
- [ ] Free-tier ограничения задокументированы
- [ ] Secrets не в git
- [ ] `prod-render` не ломает dev
- [ ] Swagger UI доступен
- [ ] README честно про «simplified vs full»

### T8.3 — Docs + ADR
- [ ] README про **решения** (не «я использовал Kafka»)
- [ ] ADR с **trade-offs**
- [ ] Диаграмма читаемая
- [ ] Ограничения честно
