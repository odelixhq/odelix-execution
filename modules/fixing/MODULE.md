# EXC-FIX — fixing

**Owner repo:** `odelix-execution` · **Activation gate:** V1 · **Design:** 1.0.0 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/fixing`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Candidate/dispute/final expiry fixing под frozen series policy. Не arbitrary admin price edit.

## Public ports и контракты

ObserveFixingWindow, ProposeFixing, ChallengeFixing, FinalizeFixing → FinalFixingRef; consumer sees candidate vs final.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

EXC-REG settlement spec; OracleRouter observations with source/freshness; restricted emergency governance.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

Pending→candidate→challenge/resolved→final; final value one-way except expressly defined exceptional process with audit. Clock and window bounds explicit.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

Stale feeds do not silently finalize; fallback source/window must have been declared. Chain indexing delay cannot alter final value.

## Acceptance и review

Boundary timestamps, divergent feeds, missing window, challenge race, duplicated finalization, rounding at strike, halted underlying calendar.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.
