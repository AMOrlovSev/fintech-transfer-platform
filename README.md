# fintech-transfer-platform

Платформа переводов между счетами — pet-проект для демонстрации production-grade практик backend-разработки: распределённые транзакции (Saga + Outbox), надёжная доставка событий через Kafka, идемпотентность, наблюдаемость, безопасность через OAuth2/OIDC.

## Стек

Java 21 · Spring Boot 3.3 · PostgreSQL 16 · Apache Kafka (KRaft) · Redis 7 · Keycloak · Spring Cloud Gateway · OpenFeign · Resilience4j · Testcontainers · Gradle (Kotlin DSL) · Docker · Kubernetes (kind) · Prometheus + Grafana · OpenTelemetry + Jaeger

## Быстрый старт

```bash
git clone <repo-url>
cd fintech-transfer-platform
cp .env.example .env
docker compose -f infra/docker-compose.yml up -d
./scripts/wait-for-infra.sh
./gradlew build
./gradlew :transfer-service:bootRun
curl localhost:8082/actuator/health
```

Подробнее — в [SETUP.md](SETUP.md).

## Документация

- [TZ-fintech-transfer-platform.md](TZ-fintech-transfer-platform.md) — техническое задание (источник истины)
- [PLAN-fintech-transfer-platform.md](PLAN-fintech-transfer-platform.md) — план на 8 недель, трекер задач
- [MASTER-CONTEXT.md](MASTER-CONTEXT.md) — контекст для работы с AI-техлидом
- [SETUP.md](SETUP.md) — локальный запуск и troubleshooting
- [CONTRIBUTING.md](CONTRIBUTING.md) — конвенции кода, ветки, коммиты, DoD
- [docs/adr/](docs/adr/) — Architecture Decision Records
- [docs/glossary.md](docs/glossary.md) — глоссарий доменных терминов
- [docs/REVIEW-CHECKLIST.md](docs/REVIEW-CHECKLIST.md) — чек-листы ревью

## Архитектура (high-level)

```
Client → Gateway → transfer-service → account-service
                 ↓
              Kafka → notification-service
```

> Диаграммы появятся в `docs/diagrams/` по мере готовности.

## Live demo

Render: TBD после T8.2. Демо-версия — упрощённая (один web-сервис + PostgreSQL + Upstash Kafka). Полная архитектура разворачивается локально через `docker compose`.

## Лицензия

MIT (для pet-проекта).
