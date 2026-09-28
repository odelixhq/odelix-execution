# EXE-OMS — oms

**Owner repo:** `odelix-execution` · **Activation gate:** PAPER · **Design:** 1.0.0 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/oms`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Order lifecycle, accepted intents, cancel/fill races and observed venue state. Не заменяет strategy registry или pricing.

## Public ports и контракты

SubmitOrder(ApprovedIntent, PolicyDecision), CancelOrder, ObserveExecutionEvent, ReconcileOrder → OrderState/Fill/Unknown.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

EXE-POL/RSK, venue/paper adapter, EXE-REC observation; immutable instruments and approved strategy refs.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

Created→validated→submitted→ack/open/partial/filled/cancelled/expired/rejected; uncertain sends become UNKNOWN. clientOrderId dedup and cumulative filled qty.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

HTTP timeout не разрешает повторный blind send. Cancel request не доказательство cancel. Partial-fill remaining qty and policy counters reconciled.

## Acceptance и review

Fill during cancel; duplicate ack; websocket gap; unknown recovered by venue lookup; adapter atomic_strategy=false; no signing scope escalation.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.
<!-- R159 event-ordering -->
## Общая event discipline

Durable intent precedes modeled action; commit/result events и ACK раздельны. Operation ID повторяется идемпотентно, конфликтующий digest не перезаписывается. Crash points до/после WAL и неизвестный outcome проверяются независимо. HOSTED/portable PAPER используют одну semantics. Событие STOP_REQUESTED не эквивалентно STOP_CONFIRMED; недоступность хоста оставляет UNKNOWN.
