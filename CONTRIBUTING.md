# CONTRIBUTING — конвенции fintech-transfer-platform

## Ветки

Формат: `<type>/<task-id>-<short-name>`

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `infra`.

Примеры: `feat/T1.1-init-repo`, `fix/T3.2-idempotency-hash`.

Ветки живут ≤ 2–3 дней. Большие задачи дробим.

## Коммиты (Conventional Commits)

Формат: `<type>(<scope>): <subject>`

Scope = модуль или область: `account`, `transfer`, `notification`, `gateway`, `common`, `infra`, `ci`.

Примеры:
```
feat(account): add atomic debit operation with pessimistic lock
fix(transfer): normalize idempotency hash by sorting fields
test(transfer): add concurrency test for idempotency replay
docs(adr): add ADR-0002 for Outbox pattern
```

Правила: императив, английский, ≤ 72 символов, без точки. Footer: `Refs: T1.1`.

## PR

**Размер:** ≤ 400 строк diff (не считая тестов).

**Заголовок:** `[T1.1] Initialize multi-module Gradle project`

**Описание:**
```markdown
## Что
- [список изменений]

## Зачем
Refs: PLAN-fintech-transfer-platform.md, T1.1

## Как проверить
1. ./gradlew build
2. docker compose up -d
3. curl localhost:8082/actuator/health

## Чек-лист
- [ ] Тесты зелёные
- [ ] ./gradlew build — SUCCESS
- [ ] README/ADR обновлены
- [ ] DoD выполнен
```

Merge только после APPROVE.

## Definition of Done (DoD)

- [ ] Код собирается (`./gradlew build`)
- [ ] Unit + integration тесты зелёные
- [ ] Flyway-миграции применены (если схема менялась)
- [ ] OpenAPI обновлён (если API менялся)
- [ ] Логи структурированные, есть `traceId`
- [ ] Нет `TODO` без ссылки на issue
- [ ] `docker compose up` — всё поднимается
- [ ] README / ADR обновлены
- [ ] PR прошёл ревью (APPROVE)

## Стиль кода

### Java
- Java 21
- Records для DTO
- `final` где возможно
- `var` где тип очевиден
- Нет `null` в `Collection` — пустая коллекция
- `Optional` — только для возвращаемых значений
- Нет `System.out.println` — только SLF4J

### Naming
- Пакеты: `ru.<yourname>.ftplatform.<module>.<layer>`
- Классы: `*Service`, `*Repository`, `*Controller`, `*Mapper`, `*Client`
- DTO: `*Request`, `*Response`
- Тесты: `*Test` (unit), `*IT` (integration)

### Lombok
- Разрешён: `@Getter`, `@Builder`, `@RequiredArgsConstructor`
- **Не** используем `@Data` на entity

### Импорты
- Без `import *`
- Порядок: java.* → jakarta.* → org.* → com.* → локальные
- Spotless (Google Java Format) в CI

### SQL
- Flyway: `V<n>__<snake_case>.sql`, применённые миграции не редактируем
- Индексы на FK и поля фильтрации/сортировки
- `CHECK` для инвариантов
- Partial indexes для «горячих» запросов

### YAML
- 2 пробела, без табов
- Env: `${VAR:default}` (default только в dev)
- Secrets — не в git

## Тесты

- **Unit** (`*Test`): без Spring, Mockito, быстрые
- **Integration** (`*IT`): `@SpringBootTest` + Testcontainers
- Один тест = одна причина падения
- Имена: `should_<behavior>_when_<condition>`
- AssertJ, не JUnit assertions

## Что НЕ делать

- Не пушить в `main` напрямую
- Не коммитить секреты, `.env`, `*.pem`
- Не мержить PR без APPROVE
- Не начинать следующую задачу до APPROVE текущей
- Не использовать `double` для денег
- Не логировать PII / токены
- Не использовать `Thread.sleep` в тестах (Awaitility)
- Не отключать тесты «чтобы CI прошёл»
