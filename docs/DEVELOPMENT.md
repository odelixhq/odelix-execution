# odelix-execution — руководство разработчика

**r15.11 · 2026-09-28 · PROPOSED / не статус runtime.** В этом файле объединены локальная архитектура, контракты, карта модулей, тесты и выпуск. Подробные предметные спецификации сохранены отдельно.

[Вход в repo](../AGENTS.md) · [Локальная выборка задач](../delivery/CONTEXT.json)


<!-- BEGIN GENERATED IMPLEMENTATION BASIS -->
<a id="implementation-basis"></a>
## Основа реализации: что берём, что пишем и где интегрируем

**r15.11. На текущей PAPER-фазе основа — наш существующий simulator/engine и свои state machines. Новые broker/venue/wallet/chain компоненты из стратегического реестра не становятся автоматически установленной основой.**

Это локальная генерируемая выборка общего решения, а не отдельный редактируемый реестр. Полные metadata — `delivery/CONTEXT.json → implementation_blueprint`. Изменения предлагает владелец repo через Coordinator; генератор обновляет общий и локальные виды вместе. Конкретный upstream/pin и supply-chain проверяются перед включением, не по наличию названия в таблице.

**Три разных зависимости:** библиотека/внешний код; внешний сервис/данные; контракт другого Odelix repo. Последний не разрешает копировать чужую реализацию. Ниже сами архитектурные bindings; реестр документальных источников в конце файла — другая сущность.

| Источник | Режим | Модули этого repo | Задачи |
|---|---|---|---|
| [OWN](#reuse-BND-046) | REUSE_OWN | AREA-EXE-CONTRACTS, EXE-OMS, EXE-REC, EXE-RUN | [ODX-EXE-001](#issue-ODX-EXE-001), [ODX-EXE-002](#issue-ODX-EXE-002), [ODX-EXE-003](#issue-ODX-EXE-003), [ODX-EXE-004](#issue-ODX-EXE-004), [ODX-EXE-005](#issue-ODX-EXE-005) |
| [FREQTRADE](#reuse-BND-054) | CONDITIONAL_EXTERNAL_ADAPTER | EXE-FLR | [ODX-EXE-014](#issue-ODX-EXE-014) |

<a id="reuse-BND-046"></a>
### OWN: REUSE_OWN

**Берём:** Наш опубликованный Market/compute contract и fixtures; существующее Rust-ядро остаётся в Market.

**Пишем сами:** Локальный consumer adapter и независимая интеграционная проверка; для Execution — согласованный simulation port.

**Не переносим / граница:** Не копировать book/journal/sim в этот repo и не использовать чужие приватные пути.

**Источник:** `https://github.com/a3ka/hft-platform/tree/4ddd392e14e1dbd4c0b2511e50ddc9e48b8707e0`. **Source pin:** `4ddd392e14e1dbd4c0b2511e50ddc9e48b8707e0`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `INHERITED_DECLARED_SCOPE_NOT_FRESH_SOURCE_AUDIT`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-EXE-001](#issue-ODX-EXE-001) / `AREA-EXE-CONTRACTS` | `Cargo.toml`; `crates/execution-contracts/`; `schemas/paper/`; `tests/paper_contracts.rs` |
| [ODX-EXE-002](#issue-ODX-EXE-002) / `EXE-OMS` | `crates/oms/`; `crates/paper-venue/`; `crates/oms/tests/lifecycle.rs` |
| [ODX-EXE-003](#issue-ODX-EXE-003) / `EXE-RUN` | `crates/runner/`; `crates/policy/`; `crates/sim-adapter/`; `crates/runner/tests/paper_flow.rs` |
| [ODX-EXE-004](#issue-ODX-EXE-004) / `EXE-OMS` | `crates/oms/tests/races.rs`; `crates/paper-venue/tests/reordered_events.rs` |
| [ODX-EXE-005](#issue-ODX-EXE-005) / `EXE-REC` | `crates/reconciliation/`; `crates/runner/tests/crash_recovery.rs` |

<a id="reuse-BND-054"></a>
### FREQTRADE: CONDITIONAL_EXTERNAL_ADAPTER

**Берём:** Documented data API or read-only event import; no execution surface

**Пишем сами:** Provider-owned adapter, coverage/provenance and policy checks

**Не переносим / граница:** No new recorder/OMS, no hidden fallback or permission inference

**Источник:** `https://github.com/freqtrade/freqtrade`. **Source pin:** `NOT_SELECTED`; для ETAPE/OPENBB это выбранная ревизия исходников, для прочих — provenance; не installed package lock. **Проверка:** `DOCUMENTED_INTERFACE_NOT_RUNTIME_VERIFIED`.

**Предлагаемые локальные места из действующих карточек** (не подтверждённый code checkout):

| Issue / модуль | Пути |
|---|---|
| [ODX-EXE-014](#issue-ODX-EXE-014) / `EXE-FLR` | `adapters/freqtrade-readonly/`; `tests/freqtrade-readonly.test.ts` |

### Сервисы и datasets: прямые, общие и отложенные подключения

Только direct Issue означает scope конкретного подключения. Related/generic — класс работ, не выбранный provider. Future/deferred строки не надо реализовывать автоматически; они сохранены, чтобы отдельный repo не потерял архитектурный замысел.

| Источник | Покрытие | Прямые задачи | Связанный общий scope | Решение/условие |
|---|---|---|---|---|
| OKX — external execution adapter | FUTURE_NO_ISSUE |  |  | Первый кандидат внешнего route в стратегическом описании; нынешние EXE-001…006 относятся к PAPER, не к внедрению OKX.; EXT, вне текущих 137 Issues |
| Bybit / Paradigm / Derive / Aevo / Binance options | FUTURE_NO_ISSUE |  |  | Альтернативы/последующие routes, не обязательные параллельные подключения. Отдельных Issues интеграции сейчас нет.; После спроса и отдельных gate |
| Arbitrum Sepolia / Arbitrum One / RPC providers | FUTURE_NO_ISSUE |  |  | Кандидаты сети будущей venue; RPC provider не выбран; не условие раннего read-only Workstation.; V1/V2 |
| Safe / ZeroDev; Turnkey опционально | FUTURE_NO_ISSUE |  |  | Выбор одной account-модели; не внедрение трёх стэков и не реальные ключи в PAPER.; После policy/threat/cost review |
| Pyth / Chainlink | FUTURE_NO_ISSUE |  |  | Будущие oracle/fixing inputs с собственной проверкой; нет отдельной интеграционной карточки текущих волн.; V1 |
| Envio HyperIndex; The Graph/Goldsky резерв | FUTURE_NO_ISSUE |  |  | Managed indexing кандидат, а не источник полномочий collateral. Нет текущей implementation Issue.; При собственных onchain contracts |
| Tenderly | FUTURE_NO_ISSUE |  |  | Диагностика/симуляция/monitoring, не замена нашей state machine; scope пока стратегический.; V1/V2 при полезном spike |
| Foundry / Slither / Echidna; Certora; внешний аудитор | FUTURE_NO_ISSUE |  |  | Tooling, formal review и RFP описаны; named auditor не придуман и отдельные задачи пока не сформированы.; Перед собственной venue / critical-core launch |
| Hypernative | FUTURE_NO_ISSUE |  |  | Кандидат detection/response review, не приобретённая услуга.; При live exposure |
| DXmatch / Exberry | FUTURE_NO_ISSUE |  |  | Procurement alternative, не обязательная замена нашего matching и не расход ранней Workstation.; Institutional venue / достаточный бюджет |
| QuickFIX family | FUTURE_NO_ISSUE |  |  | Выбор реализации по языку gateway. Отдельной текущей задачи нет.; По запросу market maker |
| Conduit / Caldera / managed appchain | DEFERRED |  |  | Не создавать собственную сеть как prerequisite продукта.; Только по измеренным ограничениям текущей сети |

### Унаследованные варианты — не дополнительные зависимости

Связь по source group не доказывает выбор каждого пакета. Ниже сохранены reference-кандидаты, связанные с локальными sources. Более старые решения могут расходиться с текущим scope; не разрешать их молча.

| Компонент | Исторический статус | Роль/граница |
|---|---|---|

**Приёмка внешнего компонента:** existing-code check → точный package/commit и license/NOTICE/dependencies → adapter test с недоступностью/ошибками/качеством → фактический pin и результат. Не создавать второй runtime/численный engine под видом ускорения. Текст этого блока не устанавливает packages, не покупает сервисы и не выдаёт production rights.
<!-- END GENERATED IMPLEMENTATION BASIS -->

<a id="architecture"></a>
## Архитектура


**Version:** 0.2.0 · **Status:** PROPOSED / docs-only until explicit PAPER activation.

Two distinct capabilities: (1) user-side PAPER/external-venue execution adapters,
and (2) own options protocol after V0. They share signed intent, policy/risk,
OMS/reconciliation contracts but not custody authority across venues.

Rust execution host composes EXE-POL/RSK/KS/OMS/REC/RUN; V1 adds RFQ and protocol
adapters. In-process typed calls, no microservices by default. Local signer is
separately isolated and has no LLM; smart-contract runtime is separately audited.
The same future repo may contain Rust crates and Solidity contracts with separate
build/audit manifests and paired conformance; no repo per contract.

For own venue EXC-CLR onchain state is cash/position/lock authority. Rust ledger
is replayable preflight/mirror with pending states; indexer is only projection.
EXC-REG owns frozen listings; EXC-FIX owns precommitted fixing. Product CAP-LED
remains personal capital reporting; never two authorities for user collateral.

Paper/Testnet/Live use separate environments, keys, endpoints and state. Payments,
agent confidence and approved analysis do not activate execution. Existing EXE
invariants plus OPT extension apply. V2 adds independent legal/security/risk/ops
evidence; MM margin exceptions require V3 risk infrastructure before money.

See [Options venue spec](OPTIONS-VENUE-SPEC.md) and
[Threat/conformance](#conformance-threat-model).


### Research/strategy capability requirements

Paper/venue lifecycle, policy, atomic native clearing и finality. Основные owners: EXE-OMS/POL/REC/RSK; EXC-CLR/RFQ/FIX.

Новая поверхность использует эти же contracts и semantic fixtures; численные/торговые engines не копируются в UI или prompts. MODULE описывает actual/planned paths раздельно.


### Workstation-first r15: текущая приёмка

Ранний выпуск только read-only analytics; semantic labels и probability не создают execution permission. PAPER и native остаются отдельными gates. Исполнительные scopes не включаются ради semantic bar.


<a id="contracts"></a>
## Контракты

**Статус этой сборки:** перечисленные семейства — спецификация. Заголовок «Published» в унаследованном тексте означает целевую поверхность публикации, не доказательство существующего release. Реальный pin/digest и conformance требуются до integration acceptance.


Families: existing execution.intent.v1 plus proposed execution.order.v1,
execution.policy.v1, exchange.series.v1, exchange.clearing.v1, exchange.fixing.v1.
Semantic owner is Execution; Stack stores only references. See registry for planned
source paths; no published artifact/version/digest is asserted by this document.

Signed economics use canonical EIP-712/ABI schemas with cross-language hash and
payoff fixtures. REST/WS expose the same command/results; MCP is a permissioned
adapter, never financial authority or tick path. Protobuf may be a new measured
wire projection; existing data schemas are not rewritten.

Instrument/payoff/fixing identity, integer quantity/cash/scale, exact all-in limits,
fees/gas separation, account/chain/contract, session epoch/policy, nonce/deadline,
approval and counterparty binding are mandatory. V0 whole fill-or-kill only.
Canonical chain receipt includes block hash/height/log coordinates and named finality.
No onchain verification claim based only on an opaque policy hash or operator
RiskSnapshot. Complete semantics: [Options venue spec](OPTIONS-VENUE-SPEC.md).


### Research/strategy capability requirements


Общие StrategySpec/ExperimentSpec — Product; calculation/data manifests — Market; agent handoff — Harness; order/policy — Execution. Schema producer owns source, Stack registry discovers it. DRAFT entries не считаются published; copy/paste бизнес-типов между repos запрещён.


### Workstation-first r15: текущая приёмка


Каноническая детализация: [ARCHITECTURE.md](#architecture); [current delivery](#DEP-b335630551).


<a id="modules"></a>
## Модули


| ID | Owns | Activation |
|---|---|---|
| EXE-POL | Delegated trading mandates/session epochs/limits | PAPER |
| EXE-RSK | Deterministic order/current-account risk; no LLM or borrowed MM credit | PAPER |
| EXE-KS | Independent local stop and chain-effective revoke/pause distinction | PAPER |
| EXE-OMS | Order lifecycle/reservation/idempotency; pending != settled | PAPER |
| EXE-REC | Venue/chain reconciliation, ancestry and unknown-outcome recovery | PAPER |
| EXE-RUN | Environment-bound execution composition/rollout | PAPER |
| EXC-REG | Own-protocol listing/payoff/fixing specification | V1 |
| EXC-RFQ | Maker routing/signed quote lifecycle and acceptance coordination | V1 |
| EXC-CLR | Atomic cash/positions/locks/fees and custody claims | V1 |
| EXC-FIX | Candidate/dispute/final fixing under frozen policy | V1 |

Physical IDs clarify earlier conceptual CAP-* roles, not a second implementation.
Full mapping and dependency allowlist: ../../odelix-stack/docs/03-DOMAIN-AND-MODULES.md.
Smart-contract files are components; they are not automatically business modules.
Narrow specifications: [Clearing](../modules/clearing/MODULE.md) and
[Delegation](../modules/delegated-policy/MODULE.md).


### Актуальные подробные спецификации

| ID | Gate | MODULE |
|---|---|---|
| EXE-OMS | PAPER | [EXE-OMS](../modules/oms/MODULE.md) |
| EXE-POL | PAPER | [EXE-POL](../modules/delegated-policy/MODULE.md) |
| EXC-CLR | V1 | [EXC-CLR](../modules/clearing/MODULE.md) |
| EXC-FIX | V1 | [EXC-FIX](../modules/fixing/MODULE.md) |
| EXC-RFQ | V1 | [EXC-RFQ](../modules/rfq/MODULE.md) |


Повторяющийся блок сохранён один раз: [см. раздел](#contracts).


<a id="testing"></a>
## Проверка


**Status:** PROPOSED; no trading implementation tests executed in this bundle.

Retain EXE/SEC risk/approval/kill/environment/fault controls and add OPT-001…018.
Test unit/rounding/payout/cash and EIP-712 Rust/TS/Solidity parity, opposite-party
full backing, expiry/dispute, package close/leg-stripping, signature replay/cancel
race, budget concurrency, chain timeout/reorg/provider mismatch and indexer lag.

Before V2, independent audits of contracts AND signer/Rust/relay surface with
resolved findings/retest and exact deployment manifest; safety drills and legal/
liquidity approval are separate gates. Foundry/Slither/Echidna and formal methods
are candidate tools, not substitutes for these controls.
Detailed [threat/conformance matrix](#conformance-threat-model).


### Research/strategy capability requirements


Meaningful acceptance scenarios: cancel/fill races, UNKNOWN submit, no-withdrawal, replay, fees/locks and duplicate settlement. Здесь перечислены planned tests; executed results должны ссылаться на commit/CI.


Повторяющийся блок сохранён один раз: [см. раздел](#contracts).


<a id="runbook"></a>
## Выпуск и эксплуатация


**Status:** PROPOSED, no production authority.

Before PAPER: isolated environment and no live keys. Before V1: V0 decision and
testnet manifests. Before V2: named independent risk/security/ops reviewers,
audit/remediation, deployment hashes, funded incident response, approved roles/
caps/fixing/finality/token policies and real maker commitments.

Run kill/revoke/cancel races, sequencer/oracle outage, duplicate tx/reorg and
independent recovery drills. Stop local sends immediately; onchain cancel/revoke
is effective only in chain order. Do not promise instant exit or guaranteed close.
Freeze affected new risk, preserve logs, reconcile canonical state, review and
explicitly rearm. Never auto-rearm, use LLM loss allocation, or erase an orphaned
event without preserving its observation history.

The safe-action matrix is in [Options venue spec](OPTIONS-VENUE-SPEC.md), §12;
it may legitimately restrict withdrawals during a custody exploit to prevent loss.


### Research/strategy capability requirements


Начинать с inventory существующего кода/Issues. Глобальный порядок и A0 находятся в `DEP-9359295523` (см. локальный реестр зависимостей); Project metadata включает нормализованный Module согласно r15.3; дополнительные поля не добавляются автоматически. Не добавлять ручной второй статус-журнал.


Повторяющийся блок сохранён один раз: [см. раздел](#contracts).


---
<a id="conformance-threat-model"></a>
## Options conformance and threat model


**Version:** 0.1.0 · **Status:** PROPOSED, tests below are requirements unless explicitly executed.

### Trust boundaries

Untrusted: user/LLM output, website/news/tool text, external agent, RFQ transport,
malicious maker, indexer/RPC availability, mutable vendor configuration.
Privileged: session signer, relayer signer, oracle/fixing submitter, pauser,
upgrader/admin, collateral token administrator, L2 sequencer. These are different
roles/keys. A multisig whose signers are all controlled by the Founder is not
independent operational review.

| Threat | Required test / evidence | Owner |
|---|---|---|
| Cross-chain/account/contract replay | Same signature in other domain fails | EXE-POL |
| Duplicate fill/retry/cancel race | Exactly one whole-package execution; competing operation ordered | EXE-OMS + EXC-CLR |
| Leg manipulation/rounding attack | Wrong ratios/spec hashes/units fail before cash moves | EXC-CLR |
| Agent collateral theft via adverse quote | Price/fee/spend counters and template bound enforced on every path | EXE-POL + EXE-RSK |
| Unsupported policy hash | No opaque-hash-only acceptance | EXE-POL |
| Fake read-only RFQ | Outbound RFQ rate/access audit; no trade until approval | EXC-RFQ |
| Unbacked MM naked call | Reject even whitelisted maker | EXC-CLR |
| Reuse of protective leg | Package lock prevents credit reuse/leg removal | EXC-CLR |
| Stale model used as clearing truth | Full liability check independent of mark; typed distinction | EXE-RSK + EXC-CLR |
| Token donation/fee/rebase | Surplus classified; unsupported token rejected; no inflated claims | EXC-CLR |
| Candidate oracle fixing replaced | Frozen policy governs candidate/dispute/final states | EXC-FIX |
| Reorg/provider disagreement | Restore common ancestor; no spend of orphaned credits | EXE-REC |
| RPC timeout retry double trade | UNKNOWN blocks blind resubmission; reconcile nonce/order | EXE-OMS |
| Pauser/upgrader exploitation | Privilege graph tests; withdrawal-safe subset verified per incident | EXE-KS |
| Prompt injection or Run cancel | Cannot authorize trade or cancel already committed trade implicitly | HAR-CAP + EXE-POL |
| Bytecode/config drift after audit | Release digest and findings/remediation tied to exact deployment | EXE-RUN |

### Required fixture classes

1. Instrument: linear/inverse/quanto distinctions, multiplier, scales, expiry UTC,
   settlement/fixing identity and effective-dated venue mapping.
2. Math: payoff at zero/strikes/tails, fees, liability lock, cash conservation,
   overflow/dust and supported precision. Rust/EVM exact for cash/payout.
3. Pricing: equivalent conventions across reference implementations, documented
   per-output tolerance, finite-difference Greeks and edge cases.
4. Signatures: EOA and smart-account/session key validation, wrong signer/epoch/
   domain, immutable leg ordering, full-quantity fill and fee caps.
5. State: crash between reservation/submit/receipt/finality, duplicate events,
   replaced tx, revert and reorg; replay same canonical state.
6. Policy: request and final post-trade state, cumulative budgets across sessions,
   revoke-before-fill/fill-before-revoke, indirect approve/call attacks.
7. Operations: missing oracle history, stablecoin outage, MM disappearance,
   sequencer halt/grace, empty/failing signer and independent recovery access.

### Release record

All activated capital invariants reference exact test build and config digests,
findings + remediation + retest, failure logs, monitoring drills and owner approval.
Evidence is attached to the corresponding GitHub Issue/PR/release; do not require
the Founder to maintain parallel Markdown proof inventories. CI can export a
release report. Cross-language fixture pass is not a contract audit, nor proof of
market liquidity or legal clearance.

### Fault-recovery sequence

Freeze affected new risk → retain immutable observations/pending intents → query
independent canonical chain state → identify last good checkpoint → rebuild and
compare cash/positions/locks/nonces → resolve all unexplained discrepancies →
run adversarial reproducer → review/rearm. Do not erase old logs, auto-rearm on
process restart, or let an LLM decide a loss allocation or settlement price.
<!-- R159 paper-and-recorder -->
## Generic PAPER, Flight Recorder и условный Node

EXE-001/002/009/010/013 публикуют общий simulation/policy/OMS path. MKT-038 — узкий адаптер существующего sim, не новый движок. EXE-003 добавляет Research binding со StrategyVersion/MKT-013. HOSTED_PAPER_READY и PAPER_READY — разные результаты.

[FLIGHT-RECORDER](FLIGHT-RECORDER.md) владеет producer event/import semantics; [PAPER-NODE](PAPER-NODE.md) — conditional packaging. Product проецирует эти события и управляет пользовательским lifecycle, не повторяет OMS. DecisionEvent запись предшествует modeled transition; запрос остановки, попытка и confirmed state раздельны. Новые tasks не дают real-environment capabilities и не запускают broker APIs.

<!-- BEGIN GENERATED REPO CONTEXT -->

<a id="modules-index"></a>
## Каталог модулей этой области

Один primary owner в каждой задаче; affected modules отдельно. AREA — служебная классификация. Идентификаторы сохранены. Activation — scope текущего плана, не доказательство исполнения; независимый implementation_status в JSON.

| ID | Вид | Activation | Источник в этом repo |
|---|---|---|---|
| `AREA-EXE-CONTRACTS` | DELIVERY_AREA | ACTIVE | [docs/DEVELOPMENT.md](#contracts) |
| `EXC-CLR` | MODULE | STRATEGIC_BRANCH | [modules/clearing/MODULE.md](../modules/clearing/MODULE.md) |
| `EXC-FIX` | MODULE | STRATEGIC_BRANCH | [modules/fixing/MODULE.md](../modules/fixing/MODULE.md) |
| `EXC-REG` | MODULE | STRATEGIC_BRANCH | [docs/DEVELOPMENT.md](#modules) |
| `EXC-RFQ` | MODULE | STRATEGIC_BRANCH | [modules/rfq/MODULE.md](../modules/rfq/MODULE.md) |
| `EXE-KS` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `EXE-OMS` | MODULE | ACTIVE | [modules/oms/MODULE.md](../modules/oms/MODULE.md) |
| `EXE-POL` | MODULE | ACTIVE | [modules/delegated-policy/MODULE.md](../modules/delegated-policy/MODULE.md) |
| `EXE-REC` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `EXE-RSK` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `EXE-RUN` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `EXE-FLR` | MODULE | ACTIVE | [docs/DEVELOPMENT.md](#modules) |
| `EXE-NODE` | MODULE | STRATEGIC_BRANCH | [docs/DEVELOPMENT.md](#modules) |

<a id="work-queue"></a>
## Локальная очередь и следующий шаг

Полный текст каждой задачи — в [delivery/CONTEXT.json](../delivery/CONTEXT.json), это автоматически полученная выборка, не второй редактируемый backlog. Найдите объект по `id`, прочитайте `work`, `acceptance`, `paths`, `depends_on`, `primary_module_id`.

Сначала действующий Issue, actual commit/permissions, затем следующее допустимое действие. Planned wave не статус и не мандат. Внешняя зависимость должна предоставить артефакт/fixture; соседний checkout не предполагается.

| ID | Модуль | Волна | Результат | Зависимости |
|---|---|---|---|---|
| <a id="issue-ODX-EXE-001"></a>`ODX-EXE-001` | `AREA-EXE-CONTRACTS` | 2 | Создать PAPER-only Execution workspace и producer contracts | ODX-STK-008 |
| <a id="issue-ODX-EXE-002"></a>`ODX-EXE-002` | `EXE-OMS` | 3 | Реализовать Paper OMS и append-only execution events | ODX-EXE-001, ODX-MKT-038 |
| <a id="issue-ODX-EXE-003"></a>`ODX-EXE-003` | `EXE-RUN` | 5 | Связать paper runner, versioned strategy и deterministic policy | ODX-EXE-013, ODX-MKT-013, ODX-PRD-019 |
| <a id="issue-ODX-EXE-004"></a>`ODX-EXE-004` | `EXE-OMS` | 4 | Проверить paper lifecycle на partial-fill/cancel/duplicate races | ODX-EXE-013 |
| <a id="issue-ODX-EXE-005"></a>`ODX-EXE-005` | `EXE-REC` | 4 | Реализовать paper recovery и unknown-outcome reconciliation | ODX-EXE-004 |
| <a id="issue-ODX-EXE-006"></a>`ODX-EXE-006` | `EXE-KS` | 4 | Реализовать независимый paper stop/revoke control | ODX-EXE-005 |
| <a id="issue-ODX-EXE-007"></a>`ODX-EXE-007` | `EXE-FLR` | 1 | Опубликовать producer envelope DecisionEvent и независимые fixtures | ODX-STK-001 |
| <a id="issue-ODX-EXE-008"></a>`ODX-EXE-008` | `EXE-FLR` | 1 | Реализовать read-only Flight Recorder import API/SDK | ODX-EXE-007 |
| <a id="issue-ODX-EXE-009"></a>`ODX-EXE-009` | `EXE-POL` | 2 | Опубликовать PAPER PolicyBundle и SimulationScenario contracts | ODX-EXE-001, ODX-EXE-007 |
| <a id="issue-ODX-EXE-010"></a>`ODX-EXE-010` | `EXE-POL` | 2 | Подключить детерминированный PAPER policy kernel без второго движка | ODX-EXE-009, ODX-MKT-038 |
| <a id="issue-ODX-EXE-013"></a>`ODX-EXE-013` | `EXE-RUN` | 3 | Связать generic hosted PAPER runner с общим OMS | ODX-EXE-002, ODX-EXE-010, ODX-EXE-007 |
| <a id="issue-ODX-EXE-014"></a>`ODX-EXE-014` | `EXE-FLR` | 2 | Подключить Freqtrade как отдельный read-only источник Flight Recorder | ODX-EXE-007, ODX-EXE-008 |
| <a id="issue-ODX-EXE-011"></a>`ODX-EXE-011` | `EXE-NODE` | 5 | Собрать переносимый PAPER Node на общем runtime | ODX-EXE-013, ODX-EXE-006, ODX-PRD-036, ODX-STK-017 |
| <a id="issue-ODX-EXE-012"></a>`ODX-EXE-012` | `EXE-KS` | 5 | Проверить PAPER Node на отказах без биржевого testnet | ODX-EXE-011 |

<a id="dependencies"></a>
## Внешние зависимости и источники

**Реестр ниже не является очередью обязательного чтения.** Большинство записей — происхождение решений; артефакт запрашивается только для конкретной необходимой зависимости задачи. **Независимость checkout не означает отсутствие зависимостей продукта.** Здесь записаны логические координаты издателя, а не относительные переходы в соседнюю папку. `source_sha256` в локальном JSON удостоверяет только исходный документ r15.3. Ни одна строка не является доказательством выпуска schema/SDK.

Для конкретной задачи получить через TaskPacket или разрешённый артефактный канал: producer + contract ID, точный release/commit, schema/package digest, fixtures, consumer-conformance и разрешённые операции. Записать фактический локальный путь после получения; пока artifact не предоставлен, зависимая runtime-работа не готова. Локальные fixture/design-задачи возможны по своему мандату.

Не заменять чужую схему её ручной копией. Обновление версии — producer release → consumer pin → conformance → интеграция. Разрешение читать артефакт не даёт права изменять другой repo.

| Ref | Publisher | Логический источник | Статус исходника |
|---|---|---|---|
| <a id="DEP-52ea262d35"></a>`DEP-52ea262d35` | `odelix-stack` | `docs/INTEGRATIONS.md#code-reuse` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-9359295523"></a>`DEP-9359295523` | `odelix-stack` | `docs/DELIVERY-RUNBOOK.md` | DOCUMENT_SNAPSHOT_ONLY |
| <a id="DEP-b335630551"></a>`DEP-b335630551` | `odelix-stack` | `AGENTS.md` | DOCUMENT_SNAPSHOT_ONLY |

`UNRESOLVED_SOURCE_REFERENCE` — отсутствующий документ/якорь исходного пакета явно зарегистрирован; содержание не придумано. Для historical source его можно оставить архивной ссылкой, для обязательной зависимости — запросить источник. Public URLs в предметных документах сохранены как датированные ссылки и не проверялись онлайн этой сборкой.
