# EXC-RFQ — rfq

**Owner repo:** `odelix-execution` · **Activation gate:** V1 · **Design:** 1.0.0 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/rfq`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

RFQ lifecycle, eligible maker fan-out, firm quote collection, transparent ranking и atomic package acceptance.

## Public ports и контракты

RequestRFQ, SubmitFirmQuote, AcceptQuote, ExpireRFQ; quote binds legs/ratio/qty/net price/fees/expiry/account/nonce.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

EXC-REG, EXE-POL/RSK, EXE-OMS submit, maker transport adapters; clearing receipt port.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

Requested→quoting→selected→pending→settled/failed/expired; accept idempotency key; received quote ≠ fill.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

All-in ranking cannot secretly prefer referral fee. Indicative quotes cannot be accepted as firm. Quantity/legs cannot change after signature.

## Acceptance и review

Expired quote after user click; two accepts; maker disappearance; signed leg swap; chain revert; stale preflight; partial fills rejected in v0.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.
