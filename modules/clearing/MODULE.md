# EXC-CLR — clearing

**Owner repo:** `odelix-execution` · **Activation gate:** V1 · **Design:** 1.0.0 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/clearing`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Authoritative native collateral, cash flows, positions, fees, locks/nonces в одной финансовой transaction boundary. Product portfolio ledger не authority.

## Public ports и контракты

ClearSignedPackage, ReadCollateralAccount, SettleExpiredPackage, WithdrawFreeCollateral → atomic receipt/events.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

EXC-REG frozen specs, EXC-FIX final fixing, EXE-POL verifier, constrained token transfer adapters; no LLM/indexer authority.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

Separate cash double-entry, positions and encumbrance ledgers with reconciliation. Package fills all-or-none; signed caps and post-state collateral checked atomically.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

v0 fully backed same-expiry bounded package for both counterparts. Positive premium can contribute once to required lock; no credit double counting. Withdraw cannot remove locked liabilities.

## Acceptance и review

Quantity replay; insolvency adversarial package; fees/rounding conservation; same tx all legs or none; collateral depeg policy; duplicate fixing payout; paused withdraw under actual vault threat.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.
