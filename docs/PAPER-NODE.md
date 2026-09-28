# Portable PAPER Node

**r15.11 · PROPOSED / CONDITIONAL / NOT_VERIFIED.** Owner runtime: Execution. Owner пользовательской регистрации/плана: Product. Owner package/CI: Stack.

## 1. Scope

Переносимый пакет для REPLAY/PAPER и локальных/no-egress сценариев, на тех же SimulationPort/PolicyBundle/DecisionEvent, что hosted-система. Отдельной рыночной книги, OMS или второго agent framework не появляется. Текущий пакет не принимает биржевые ключи, не отправляет реальные команды и не содержит способа включить это флагом.

Deterministic runner не требует Pi. Опциональный sanitized Pi client — HAR-019 после собственного допуска; private hosted Skills/rubrics/corpus не входят в image. User model runs изолированы и имеют память/CPU/time/egress budget. BYOK не меняет authority.

## 2. Условие реализации

STK-023 оценивает NODE_PAPER_DEMAND до разработки EXE-011. Предложенные 5 операторов / 60 дней / 2 пилотных согласия — не утверждённые измерения. Нужны квалифицированные локальные use cases и записанный scope/support/privacy approval. Проверка раз в 14 дней; exception Founder — явно ограниченный spike, не обход безопасности.

## 3. Lifecycle пакета

Product создаёт plan с immutable manifest/policy/data refs; user approval привязан к digest. Stack выпускает package manifest; Execution запускает simulation только после проверки совместимости и допустимого состояния. Update предлагается и принимается явно; runtime не перезаписывает себя на latest.

Желаемое состояние и наблюдаемое разделены. Enrollment/heartbeat — минимальные control metadata без private source values. Отсутствие heartbeat означает UNREACHABLE/UNKNOWN, не STOPPED. Никаких облачных token values в manifest, logs или LLM context. Фактическая установка/доступы не выполняются этим документом.

## 4. Отказы

Write-ahead event precedes simulated state transition. StopRequest → StopAttempt → StopConfirmed/StopUnknown. Halt marker durable и не снимается моделью. Реакция задаётся симуляционной policy; универсальный cancel-all не объявляется безопасным для всякого внешнего состояния.

Несколько процессов одного хоста разделяют CPU, память, сеть и отказ питания. Их разделение ограничивает часть ошибок, но не гарантирует выживания safety-process при потере хоста. Приёмка EXE-012 — fault injection в симуляции, без требования real/testnet действия.

## 5. Данные

Пакет может читать заранее разрешённый local fixture/archive или Odelix dataset refs в рамках grants. Нельзя считать feed-identical только по версии normalizer. Direct/native/external record имеет отдельный source cut и observed coverage. Source-specific personal data-программа не означает, что Node вправе запрашивать или пересылать любой её dataset.

## 6. Готовность

PAPER_NODE_READY/STK-018 требует package/recovery/isolation/rights/coverage evidence и конкретный named profile. Agent Validation Profile фиксирует версии, число решений и покрытие сценариев; не переводит в другую среду. Целевые реальные среды остаются стратегической веткой без реализации этого выпуска.

[Policy schema](../contracts/policy.bundle.v1.schema.json) · [Simulation scenario](../contracts/simulation.scenario.v1.schema.json) · [Flight Recorder](FLIGHT-RECORDER.md).

<a id="node-admission"></a>
## Именованные условия и grants

NODE_PAPER_DEMAND — проверяемый спрос, не authorization. Дополнительно нужен NODE_PAPER_SCOPE_APPROVED с точным scope, evidence и signer. Для optional локального brain — NODE_PAPER_LOCAL_BRAIN_APPROVED. Указание волны 5/6 в плане не делает STRATEGIC_BRANCH активной. Portable PAPER не содержит биржевые trade-ключи; отдельный процесс не переживает потерю общего хоста. Agent Validation Profile — свидетельство проверок, не мандат. Product CLI `odelix` и инженерный `./workshop` не взаимозаменяемы.
