# Глоссарий доменных терминов

## Домен

| Термин | Определение |
|---|---|
| **Account** | Счёт пользователя. Источник истины по балансу. Владеет `account-service`. |
| **Balance** | Остаток на счёте в **минорных единицах** (копейки), `long`. |
| **Money** | Value object: `(long minor, Currency currency)`. Не `double`. |
| **Transfer** | Процесс перевода с одного счёта на другой. Владеет `transfer-service`. |
| **Idempotency-Key** | UUID от клиента; повторный POST не создаёт второй перевод. |
| **Saga** | Распределённая транзакция как последовательность локальных с компенсациями. |
| **Compensation** | Обратное действие для отката шага Saga (`compensateDebit`). |
| **Outbox** | Таблица с событиями; пишется в одной транзакции с бизнес-изменением. |
| **Outbox Relay** | Фоновый процесс, публикующий `outbox` в Kafka. |
| **Watchdog** | Фоновый процесс, находящий «зависшие» `PENDING` и завершающий их. |
| **Idempotent Consumer** | Консьюмер, дедуплицирующий события по `event_id`. |
| **DLQ** (Dead Letter Queue) | Топик для «отравленных» сообщений после N неудачных попыток. |

## Статусы

### TransferStatus
| Статус | Значение |
|---|---|
| `PENDING` | Создан, Saga выполняется |
| `COMPLETED` | Успешно завершён |
| `FAILED` | Ошибка, компенсации выполнены |
| `CANCELLED` | Отменён до завершения |

### AccountStatus
| Статус | Значение |
|---|---|
| `ACTIVE` | Нормальный, принимает и отправляет |
| `BLOCKED` | Заблокирован админом |
| `CLOSED` | Закрыт, read-only |

## События (Kafka)

| Событие | Источник | Когда |
|---|---|---|
| `TransferCreated` | transfer | Перевод создан (PENDING) |
| `TransferCompleted` | transfer | Перевод успешно завершён |
| `TransferFailed` | transfer | Перевод не удался |
| `AccountDebited` | account | Деньги списаны |
| `AccountCredited` | account | Деньги зачислены |
| `AccountCompensated` | account | Списание откачено |

## Роли (Keycloak)

| Роль | Права |
|---|---|
| `USER` | Свои счета и переводы |
| `ADMIN` | Все счета и переводы, отмена, top-up |
| `SERVICE` | Внутренние `/internal/**`, m2m |

## Технические термины

| Термин | Определение |
|---|---|
| **Минорные единицы** | Копейки / центы. `10000` = 100.00 RUB. |
| **Problem Details** | RFC 7807, `application/problem+json`. |
| **Cursor pagination** | Пагинация по `(created_at, id)`, не offset. |
| **Partial index** | Индекс с `WHERE` — по части строк. |
| **Optimistic lock** | `@Version` в JPA; при конфликте `OptimisticLockException`. |
| **Pessimistic lock** | `SELECT ... FOR UPDATE`; блокирует строку до конца транзакции. |
| **Circuit Breaker** | При N% ошибок «открывается» и не зовёт сервис. |
| **Trace / Span** | Цепочка спанов одного запроса / один шаг. |
| **SLI / SLO / SLA** | Индикатор / цель / контракт. |
| **RED / USE** | Rate-Errors-Duration / Utilization-Saturation-Errors. |
