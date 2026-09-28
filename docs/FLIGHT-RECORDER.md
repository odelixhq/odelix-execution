# Flight Recorder — producer события и независимый импорт

**r15.11 · PROPOSED · NOT_IMPLEMENTED_BY_BUNDLE.** Owner: `odelix-execution`; Product хранит пользовательскую проекцию. EXE-007/008, PRD-032, WKS-022, STK-020. Freqtrade — отдельный условный EXE-014, не обязательный источник.

## 1. Что записываем

`decision.event.v1` — immutable producer envelope (по смыслу DecisionEvent); логический DecisionRecord — агрегированное представление в Product. Не создаём два владельца одной mutable записи.

Событие содержит `producer_id`, `stream_id`, `stream_epoch`, `sequence`, `event_id`, `operation_id`, `event_kind`, `predecessor_event_id`, `source_environment`, временные поля и `input_refs` с качеством; точные поля задаёт локальная схема. Payload ограничен schema/version. Входы, proposal, отказ, abstention, approval, моделируемый outcome и поздняя reflection — разные события. Промежуточные скрытые рассуждения модели не сохраняются и не требуются.

Source environment может описывать импортированную историю, но не открывает исполняемые permissions. Текущий writer и SDK не отправляют торговых команд.

## 2. Приём событий

SDK, файл и HTTP ingest вызывают одну валидацию: tenant/producer grant, schema/version, record-size/batch-size, dedup `(producer_id,event_id)` и sequence monotonicity. Повтор с тем же ID и другим digest — конфликт/quarantine; не overwrite. Пакет может прийти поздно или с пробелом: gap explicit, не reorder с потерей provenance.

Сначала durable append, затем ACK. Cursor публикуется после commit. Ошибка между commit и ACK допускает retry без дубликата. Отсутствие всех прошлых events не мешает импортировать имеющиеся, но ограничивает completeness.

## 3. Временная семантика

`event_at` — что сообщает источник; `recorded_at` — запись у producer; `known_at` — первое достоверное получение в данной системе. Outcome/reflection получают собственное knowledge time. Date-only не превращается в точный instant. Исход с прошлым event_at, импортированный позже, не попадает в prior as-known replay.

Импорт не изменяет canonical journal Market. `input_ref` может ссылаться на native journal cut, immutable external receipt или локальный разрешённый artifact. Равенство normalizer version не гарантирует равенства набора событий/sequence/пропусков.

## 4. Уровни полноты

| Уровень | Достаточное доказательство |
|---|---|
| IMPORTED | Валидные имеющиеся records; весь вход неизвестен |
| RECONSTRUCTABLE | Есть доступные разрешённые inputs, версии преобразования и clock/cut, необходимые для заявленной области |
| VERIFIED_AS_SHOWN | Независимая проверка воспроизводит показанное тогда состояние на точном наборе inputs; scope явно указан |

Наличие hash/reference само по себе не даёт второй/третий уровень. Callback внешнего бота может не видеть ни все inputs, ни rejected alternatives. UI обязан показывать эти пределы.

## 5. Local full и центральный egress

Полная запись остаётся там, где разрешено её хранение. Центральный `decision.egress.v1` — allowlist metadata/opaque refs и одобренные минимальные descriptors. Не отправляем автоматически market values, derived features или narrative, содержащие чужие данные. Rights evaluation имеет собственный digest и evidence.

Deribit public universe и ограниченные именованные FRED/ALFRED personal files включены в разные профили data-программы. Никакого FRED wholesale/API-store; права на конкретный payload/операцию проверяются независимо. Hash-only timeline остаётся полезной историей событий; это не лазейка для скрытого refetch и не обещание реконструкции.

Immutable event не означает право бессрочного хранения. Retention/revocation выполняются по applicable policy: доступ прекращается/данные удаляются при необходимости, audit показывает недоступность, а не выдуманные прежние bytes.

## 6. Freqtrade

EXE-014 получает выбранную версию API и licence screen. Предлагаемый adapter — внешний read-only клиент существующего HTTP/file интерфейса. Не загружаем закрытый плагин в процесс Freqtrade, не используем управляющие endpoints и не считаем весь REST API безопасным чтением.

GPL obligations зависят от фактического заимствования и объединения, а не только HTTP. Pin/license/NOTICE и необходимые исходники при распространении проверяются отдельно. UI и data scope не блокируются до этого коннектора; generic fixture/import достаточны ранней приёмке.

## 7. Независимые проверки

Положительная timeline + late import + duplicate + conflicting ID + missing source + revoked rights + unknown event type + sequence gap + bounded payload + different tenant. Преднамеренное удаление записи отказа должно менять expected timeline. Проверка schemas не доказывает полноту реальных logs.

Схемы: [event](../contracts/decision.event.v1.schema.json), [egress](../contracts/decision.egress.v1.schema.json). Подробные задания: [локальная очередь](../delivery/CONTEXT.json).

<a id="event-fields"></a>
## Producer event и проверяемая полнота

`decision.event.v1` — имя единственного producer envelope; `DecisionRecord` — его пользовательская проекция. Порядок задаёт stream_id + stream_epoch + sequence (decimal string). Event time nullable, precision отдельно; DAY не превращается в точную полночь. Critic имеет NOT_RUN / PASS / PASS_WITH_LIMITATIONS / REPAIR / BLOCK_NO_CONCLUSION / ERROR; NOT_RUN не имеет result ref.

Import не доказывает полный источник событий. Повтор идентичного id/digest идемпотентен, конфликт digest — ошибка. PERSONAL_USE записи не читаются продуктовой проекцией без отдельного продуктового разрешения. Внешний Freqtrade — CONDITIONAL_EXTERNAL_ADAPTER, read-only HTTP/file allowlist, grant FREQTRADE_CONNECTOR_APPROVED. Это не перенос внутреннего Python runtime и не разрешение его функций исполнения.

## Согласованность event и egress

`decision.egress.v1` переносит разрешённые `stream_id`, `stream_epoch`, `sequence`, `event_at_precision`, `critic_status` и `critic_result_ref` без изменения их смысла. Дата DAY не повышается до instant, UNKNOWN остаётся null, NOT_RUN не заменяется PASS. Локальный результат Critic может быть представлен только разрешённой непрозрачной ссылкой; ссылка не содержит исходный текст и не разрешает автоматическое чтение. При запрете передачи этих metadata отправка события блокируется, а не маскируется другим фактом.

Единственный runtime-источник этих полей — producer. Egress сохраняет sequence источника; пропущенные из-за политики события могут образовать явные пробелы. `coverage` описывает заявленную проверенную область у источника, не возможность восстановить inputs в control plane. Date-time проверяется с обязательным RFC3339 checker, даты — с проверкой календаря; отсутствие format-checker является ошибкой проверки комплекта.
