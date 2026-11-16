# SETUP — локальный запуск

## Требования

| Инструмент | Версия | Проверка |
|---|---|---|
| JDK | 21 (LTS) | `java -version` |
| Gradle | wrapper (8.10+) | `./gradlew --version` |
| Docker Desktop | 4.x+ | `docker --version` |
| Docker Compose | v2 | `docker compose version` |
| Git | 2.30+ | `git --version` |

**RAM:** ≥ 8 GB для Docker Desktop. 12 GB — комфортно.

## Первый запуск

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

## Порты

| Сервис | Порт |
|---|---|
| api-gateway | 8080 |
| account-service | 8081 |
| transfer-service | 8082 |
| notification-service | 8083 |
| PostgreSQL | 5432 |
| Kafka (снаружи) | 29092 |
| Redis | 6379 |
| Keycloak | 8180 |
| Prometheus | 9090 |
| Grafana | 3000 |
| Jaeger UI | 16686 |

## Полезные команды

### Инфраструктура
```bash
docker compose -f infra/docker-compose.yml logs -f postgres
docker compose -f infra/docker-compose.yml restart kafka
docker compose -f infra/docker-compose.yml down -v   # снести с данными
docker compose -f infra/docker-compose.yml ps
```

### PostgreSQL
```bash
docker exec -it postgres psql -U postgres -d accounts
```

### Kafka
```bash
docker exec -it kafka kafka-topics.sh --bootstrap-server localhost:9092 --list

docker exec -it kafka kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic transfers.events.v1 \
  --from-beginning

docker exec -it kafka kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --describe --group notification-service
```

### Redis
```bash
docker exec -it redis redis-cli
> KEYS idem:*
```

### Keycloak — получить токен
```bash
curl -s -X POST "http://localhost:8180/realms/fintech-transfer-platform/protocol/openid-connect/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=ft-backend" \
  -d "client_secret=<из realm-export>" \
  -d "username=alice" \
  -d "password=password" | jq -r .access_token
```

### Gradle
```bash
./gradlew build
./gradlew test
./gradlew :transfer-service:test
./gradlew :transfer-service:integrationTest
./gradlew spotlessApply
./gradlew :transfer-service:dependencies
```

## Troubleshooting

### `port already allocated`
```bash
lsof -i :5432
kill -9 <PID>
```
Или изменить порт в `.env`.

### Kafka не стартует / Testcontainers падают
Проверить RAM в Docker Desktop ≥ 8 GB.

### Testcontainers медленно
`~/.testcontainers.properties`:
```
testcontainers.reuse.enable=true
```
И `.withReuse(true)` на контейнерах.

### Flyway migration failed
```bash
docker exec -it postgres psql -U postgres -d accounts -c "SELECT * FROM flyway_schema_history;"
docker compose -f infra/docker-compose.yml down -v   # только dev!
```

### OOM при запуске сервисов + инфры
```bash
GRADLE_OPTS="-Xmx512m" ./gradlew :transfer-service:bootRun
```

### Keycloak долго стартует
Нормально для `start-dev` — 30–60 секунд.

## Следующие шаги

- [PLAN-fintech-transfer-platform.md](PLAN-fintech-transfer-platform.md) — начните с T1.1
- [MASTER-CONTEXT.md](MASTER-CONTEXT.md) — как работать с AI-техлидом
