# EXE-POL — delegated-policy

**Owner repo:** `odelix-execution` · **Activation gate:** PAPER · **Design:** 1.0.0 / 2026-09-28.
**Implementation status:** SPEC_ONLY / requires code inventory. Proposed path: `modules/delegated-policy`; this path does not assert an existing crate/package. Reuse existing implementation before introducing files.

## Назначение и предел ответственности

Scoped delegated authority и экономические limits, включая revoke/session epoch; не LLM prompt policy.

## Public ports и контракты

EvaluateOrder, ReserveAllowance, CommitFill, ReleaseReservation, RevokeGrant; signed approval binds account/environment/order semantics.

Имена операций задают semantics, не подтверждают опубликованный API. Producer schema/fixtures и точная версия выпускаются вместе с реализацией; Stack registry содержит discovery/pins. Ошибки типизированы, environment/tenant/asOf/unit fields обязательны там, где применимы.

## Разрешённые зависимости

Account/signature primitives, reconciled current exposure, explicit immutable AgentPolicy; policy decision events to Product.

Private imports и запись в чужие таблицы запрещены. Domain logic получает immutable inputs; infrastructure I/O подключается composition root. При новом общем контракте сначала согласуются producer fixtures и consumer migration.

## Состояние, время и восстановление

Grant active/revoked/expired; reservations and fills update counters idempotently. Onchain-enforced limits distinguish offchain-only monitoring.

Version/config/code/dataset/contract refs связывают результат с исходными условиями. Retry действует в пределах operation id; unknown external state сверяется до повторного side effect. Денежные величины и timestamps не теряют точность при сериализации.

## Инварианты и отказ

Master key unavailable to agent; withdrawal not a trading capability. Concurrent orders cannot overspend daily/premium allowance; gateway and all chain paths enforce same hard bounds.

## Acceptance и review

Two parallel allowance reservations; revoke race; stale epoch; price/quantity/unit tampering; alternative clearing entry point; nonce replay; aggregation of leg risk.

До Done нужны реальный code map/commit, выполненные meaningful checks, peer review и ссылки в Issue. Здесь тесты описаны как требования, не отмечены выполненными. Проверить публичные exports, allowed dependencies и отсутствие дублирующей реализации. Нет требования создавать новый runtime или deployment для этого модуля.

## Наблюдаемость и знание проекта

Логировать operation/run/strategy refs, duration, typed outcome, coverage и retry count без secrets/скрытых рассуждений. Метрики применяются к своей ответственности: module-specific failures из раздела инвариантов должны отличаться от infrastructure timeouts. Builder обновляет факты этого MODULE с кодом; Coordinator обновляет index/status. Не копировать полный отчёт в несколько документов.
<!-- R159 paper-policy -->
## PAPER policy bundle

PolicyBundle связывает manifest digest, simulation definition, resource budget, version/expiry и разрешённую среду. Подпись подтверждает происхождение, но не правильность риска или свежесть inputs. Мандаты нескольких агентов разделяют совокупный simulation resource budget. Смена policy/model/data/environment требует revalidation; нулевая активность не evidence успешного test profile. Универсальная cancel-all реакция не задаётся константой.
