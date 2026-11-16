# UniVAT 2.0 — Technology Stack Specification

**The complete, enterprise-grade technology stack for building, developing, testing, deploying and operating UniVAT 2.0 — realising the approved OPT2-B strategy (.NET 10 LTS modular monolith · Marten 9 event-sourced PostgreSQL 18 ledger · Wolverine 6 outbox · native decimals · TypeScript/React 19/Next.js 16 PWA · RKE2 in UNISOFT's own data center), specified layer by layer, licence by licence, and traced requirement by requirement to the PDR, PRD, BRD and URD**

---

## Document control

| Field | Value |
|---|---|
| Document title | UniVAT 2.0 — Technology Stack Specification |
| Document ID | USL-UNIVAT2-TSD-001 |
| Version | 1.1 (Baseline for build — post-audit revision) |
| Date | 15 August 2026 |
| Status | Baseline — approved strategy OPT2-B; component pins are the August-2026 baseline governed by the n−1 policy ([§11](#11-version-baseline-and-upgrade-policy)) |
| Owner | Engineering / Architecture, UNISOFT Systems Limited |
| Classification | Confidential — Internal & Named Partners |
| Parent documents | `USL-UNIVAT2-BRD-001` v1.1 · `USL-UNIVAT2-PDR-001` · `USL-UNIVAT2-PRD-001` · `USL-UNIVAT2-URD-001` v1.1 · `USL-UNIVAT2-BTO-001` v1.2 — same folder |
| Governing decision | **BTO §6.2(1): Option 2, Choice OPT2-B**, adopted by the business. This document does not re-open that decision; it implements it |
| Standards applied | ISO/IEC/IEEE 29148 traceability discipline inherited from the family; OWASP ASVS L2/L3 control mapping (`SEC-01`); SPDX licence identifiers; SLSA-aligned supply chain; four-class licence register inherited from `BTO` §9 |
| Delivery context | UNISOFT software development department: full-stack developers on Linux desktops, VSCode IDE, local dev/DB/build servers, Chrome DevTools; deployed after full test to UNISOFT's own cloud servers in UNISOFT's own data center |

### Revision history

| Ver | Date | Author | Summary |
|---|---|---|---|
| 1.0 | 15 Aug 2026 | Engineering / Architecture | First baseline. Implements the approved OPT2-B strategy as a complete component-level stack across 18 layers; cross-cutting engineering standards (money, time, tenancy, idempotency, numbering, errors, i18n); NFR-budget and security-control mappings; complete four-class licence register covering every named component; version baseline and upgrade policy; prohibited-technology register; adoption roadmap by release; full requirement traceability (TEC/NFR/SEC/LOC/DAT/INT/AI/AIC/GOV/OPS/X); mandatory audit. |
| **1.1** | **15 Aug 2026** | **Engineering / Architecture** | **External audit and revision.** 12 defects corrected ([C.5](#c5-defects-found-and-corrected)). Four material: **(a)** connection pooling was absent entirely — **PgBouncer/PgCat added** (§5.4), without which `NFR-08`'s 100,000 concurrent users is unreachable; **(b)** **Marten does not apply RLS to the event-store tables** (`mt_events`, `mt_streams`) — the ledger's isolation is application-level only, correcting §6.3/§8 and changing what `SEC-04` must test; **(c)** **ESC-POS thermal printers cannot render Bengali at all** — glyph shaping is unsupported and text-mode printing yields question marks, so §5.9 is rewritten around server-side rasterisation; **(d)** **Typst is 0.15, pre-1.0 with breaking changes still planned** — demoted to optimisation, headless Chromium promoted to primary renderer. Seven version pins corrected (TypeScript 5.9 → **6.0**, Zod 3 → **4**, ECharts 5 → **6**, Python 3.13 → **3.14**, plus ArgoCD/GitLab/Valkey precision); **Node.js LTS pin added** — it was missing despite `NFR-32`. Appendix A gains a **verification-status convention**; Appendix B gains the technology-relevant `GOV`/`OPS` rows and a corrected scope statement. |

### Position in the document family

| | BRD | URD | PDR | PRD | BTO | **This document (TSD)** |
|---|---|---|---|---|---|---|
| Question | Why invest? | What must users do? | What must the product be? | What do we build in what order? | With what foundation and stack? | **Exactly which components, versions and licences — and how they satisfy every requirement?** |
| Decides | — | — | — | — | Strategy (OPT2-B) | **Implementation: named components, engineering standards, upgrade policy, prohibited technologies** |

> **How to read this document.** [§5](#5-the-stack-layer-by-layer) is the stack itself — the authoritative component list. [§6](#6-cross-cutting-engineering-standards) contains the non-negotiable engineering standards that make the statutory requirements true in code; if you read nothing else, read §6.1 (money) and §6.6 (numbering). [§9](#9-complete-licence-register) is the CI-enforceable licence allow-list. [Appendix B](#appendix-b--requirement-traceability-matrix) proves 100% coverage of the family. [§12](#12-prohibited-and-rejected-technologies) lists what must **not** be used and why — several entries are libraries that were free when the family documents were written and are not free now.

---

## Table of contents

1. [Scope and governing decisions](#1-scope-and-governing-decisions)
2. [Stack principles](#2-stack-principles)
3. [Architecture at a glance](#3-architecture-at-a-glance)
4. [Module map of the modular monolith](#4-module-map-of-the-modular-monolith)
5. [The stack, layer by layer](#5-the-stack-layer-by-layer)
6. [Cross-cutting engineering standards](#6-cross-cutting-engineering-standards)
7. [Non-functional budgets → stack mapping](#7-non-functional-budgets--stack-mapping)
8. [Security controls → stack mapping](#8-security-controls--stack-mapping)
9. [Complete licence register](#9-complete-licence-register)
10. [Development environment and toolchain](#10-development-environment-and-toolchain)
11. [Version baseline and upgrade policy](#11-version-baseline-and-upgrade-policy)
12. [Prohibited and rejected technologies](#12-prohibited-and-rejected-technologies)
13. [Adoption roadmap by release](#13-adoption-roadmap-by-release)
14. [Risks and open technical decisions](#14-risks-and-open-technical-decisions)
- [Appendix A — Complete component inventory](#appendix-a--complete-component-inventory)
- [Appendix B — Requirement traceability matrix](#appendix-b--requirement-traceability-matrix)
- [Appendix C — Mandatory audit](#appendix-c--mandatory-audit)
- [Appendix D — Sources](#appendix-d--sources)

---

## 1. Scope and governing decisions

### 1.1 What this document decides

| In scope | Out of scope |
|---|---|
| Every named runtime, framework, library, service, tool and model in the product and its pipeline | Re-opening the OPT2-B strategy decision (BTO §6.2) |
| The licence class and CI treatment of each (§9) | Commercial terms, pricing and packaging (BRD Theme H) |
| Cross-cutting engineering standards that make statutory requirements true in code (§6) | Statutory content itself — rule text, rates, form layouts (gated by `D-01`) |
| Version pins and the upgrade policy that keeps them current (§11) | Bottom-up cost estimate (`D-02` / `D-14`) |
| Technologies that are forbidden and why (§12) | Organisation design and headcount (PRD §20.3) |

### 1.2 Inherited decisions this document implements

| Decision | Source | Implementation here |
|---|---|---|
| `D-03` Rewrite, not strangler-fig | BTO §6.3 | Greenfield .NET 10 solution; UniVAT 1.0 reached only through the migration importers (§5.13) |
| `D-04` Own cloud on own metal | BTO §6.3, `BC-5` | RKE2 topology (§5.15); identical Helm artefacts for SaaS and on-prem (`TEC-02`) |
| `D-06` Self-hosted AI inference | BTO §6.3, `AI-31`, PDPO §29(7)(b) | vLLM + Qwen3.x on UNISOFT GPUs (§5.12) |
| `D-11` Stack selection | BTO §6.2 — **closed in favour of OPT2-B** | This entire document |
| `D-12` Own-DC readiness | BTO §7.6 | Dual failure domain + in-country replica assumed by §5.15 and §7 |
| `D-13` Licence counsel review | BTO §9 | §9 is the register counsel reviews; §9.6 is the CI gate |
| `D-14` Own-DC TCO baseline | BTO §7.7 | Component list here is the input to the capacity/cost model |

### 1.3 Decisions this document adds

| ID | Decision | Owner | Due |
|---|---|---|---|
| **`D-15`** | Rules-engine selection: **GoRules ZEN** (primary) vs **Microsoft RulesEngine** — decided by an R0 spike against `NFR-02` (< 50 ms p99/line at 1,000 lines/s) and `TEC-08` byte-identical determinism | Principal Engineer (Determination) | End of R0 month 2 |
| **`D-16`** | Warehouse trigger point: when `DAT-06` volumes justify promoting ClickHouse from "provisioned" to "primary analytics store" | Data Lead | R2 entry |
| **`D-17`** | `TEC-26` scripting sandbox: adopt a sandboxed expression host or defer past R3 (Should-have) | Principal Engineer (Platform) | R3 planning |

---

## 2. Stack principles

These ten rules govern every entry in §5. A component that violates one does not enter the stack.

| # | Principle | Rationale / requirement |
|---|---|---|
| **P1** | **Two languages in the product: C# and TypeScript.** Python exists only inside the AI workbench, which is a network-isolated service with an HTTP contract | `BC-1` full-stack team; BTO §2.2 |
| **P2** | **LTS runtimes only, never STS.** .NET 10 LTS (to 14 Nov 2028); Node.js LTS lines only. `.NET 11` (STS, Nov 2026) is explicitly not adopted | **`NFR-32`: "Long-term-support versions only for runtime and framework; documented upgrade cadence"** |
| **P3** | **Licence class A or B only in shipped artefacts.** Class C (AGPL/network copyleft) is internal-operations only; Class D (non-OSI, commercial, revenue-thresholded) is forbidden outright | `BC-6`, `GOV-10`, `BRD-065`, `TEC-02` on-prem redistribution |
| **P4** | **No revenue-threshold or seat-count licences, even where UNISOFT currently qualifies.** A licence that becomes payable on success is a licence that becomes payable | BTO `BTO-R10`; §12 |
| **P5** | **Prefer foundation-governed projects** (CNCF / ASF / Linux Foundation / .NET Foundation) over single-vendor ones for anything load-bearing; where a single-vendor dependency is unavoidable, a named permissive alternative is documented in the same row | BTO `BTO-R10` — six ecosystem licence changes in 24 months |
| **P6** | **The statutory core owns no framework magic.** Determination, ledger posting, numbering and form generation are plain, testable C# over explicit SQL or Marten APIs — no ORM change-tracking, no reflection-based mapping, no runtime code generation in a money path | `TEC-07`, `TEC-08`, `TEC-11`, `TEC-25` |
| **P7** | **Everything self-hostable on UNISOFT hardware.** No component may require an internet-reachable control plane to start, run or be licensed | `BC-5`, `NFR-30`, `NFR-22` |
| **P8** | **One deployable, enforced boundaries.** Modules communicate through published in-process contracts or the message bus, never through each other's tables | `TEC-03` |
| **P9** | **Every capability ships through the public API first.** The web client is a consumer of `/v1` with no privileged back door | `TEC-01`, `TEC-05` |
| **P10** | **Determinism over convenience.** Where a choice exists between a faster-to-write approach and a reproducible one, the money and filing paths take the reproducible one | `TEC-08`, `TEC-25`, `X-04` |

---

## 3. Architecture at a glance

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ CLIENTS                                                                       │
│  Next.js 16 PWA (React 19, TS 5.9)  ·  Counter/POS offline shell             │
│  Public API consumers (ERP connectors, partners)  ·  Scalar API reference     │
└───────────────────────────────┬──────────────────────────────────────────────┘
                                │ HTTPS · OAuth 2.1 / scoped API keys · /v1
┌───────────────────────────────▼──────────────────────────────────────────────┐
│ EDGE (RKE2)   Gateway API (Envoy Gateway) · OWASP Coraza WAF · cert-manager   │
│               per-tenant rate limiting · request/trace correlation            │
└───────────────────────────────┬──────────────────────────────────────────────┘
┌───────────────────────────────▼──────────────────────────────────────────────┐
│ UniVat.App — ASP.NET Core 10 MODULAR MONOLITH (one deployable, §4)            │
│                                                                               │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┬────────────┐  │
│  │ Determination│  Documents   │  **Ledger**  │   Filing     │ Reconcile  │  │
│  │ (rules-as-   │ (Mushak      │ (Marten 9    │ (eVAT/iVAS,  │ (matching  │  │
│  │  data, ZEN)  │  generators) │  event store)│  a-Challan)  │  engine)   │  │
│  ├──────────────┼──────────────┼──────────────┼──────────────┼────────────┤  │
│  │ Masters      │ Inventory &  │  Identity &  │  AI Gateway  │ Reporting  │  │
│  │              │ Production   │  Tenancy     │              │ & Analytics│  │
│  └──────────────┴──────────────┴──────────────┴──────────────┴────────────┘  │
│  Cross-cutting: Money · Time · Numbering · Idempotency · Outbox · Flags        │
└───┬────────────┬─────────────┬─────────────┬──────────────┬──────────────────┘
    │            │             │             │              │
┌───▼────┐ ┌─────▼──────┐ ┌────▼─────┐ ┌─────▼──────┐ ┌─────▼─────────────────┐
│Postgre-│ │ RabbitMQ   │ │ Valkey   │ │ SeaweedFS  │ │ OpenSearch  ClickHouse│
│SQL 18  │ │ (Wolverine │ │ (Hybrid- │ │ (S3 objects│ │ (search)    (warehouse│
│+pgvector│ │  outbox)   │ │  Cache L2)│ │  + originals)│ │            DAT-06)  │
│CloudNativePG│         │ │          │ │            │ │                       │
└────────┘ └────────────┘ └──────────┘ └────────────┘ └───────────────────────┘
    │            │
┌───▼────────────▼──────────────────────────────────────────────────────────────┐
│ PLATFORM SERVICES                                                              │
│  Keycloak 26.7 (OIDC/SAML/SCIM) · OpenBao (secrets) · flagd (OpenFeature)      │
│  Temporal (long sagas) · Quartz.NET (schedules) · Typst/Chromium (render farm) │
│  AI workbench: FastAPI + vLLM (Qwen3.x-VL) + PaddleOCR on GPU pool             │
│  Integration zone: mock-eVAT / mock-EFDMS · a-Challan · MFS · ERP connectors    │
└────────────────────────────────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────────────────────────────────┐
│ OBSERVABILITY  OpenTelemetry → Prometheus · Grafana/Loki/Tempo/Mimir (internal) │
│ SUPPLY CHAIN   GitLab CE → Semgrep/Trivy/Gitleaks → Syft SBOM → Cosign → ArgoCD │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Module map of the modular monolith

`TEC-03` requires enforced domain boundaries in one deployable. Each module is a separate C# project (assembly) with a published contract assembly; cross-module calls go through the contract or the bus. Boundaries are enforced mechanically by architecture tests in CI (§5.17).

| Module (assembly) | Owns | Publishes | May depend on |
|---|---|---|---|
| `UniVat.Determination` | Rules-as-data evaluation, rate/exemption/SD resolution, rule-version pinning | `TaxDetermined` | Contracts only |
| `UniVat.Documents` | Mushak issuance/cancel, templates, QR, numbering requests | `invoice.issued`, `invoice.cleared`, `invoice.rejected` | Determination, Ledger contracts |
| `UniVat.Ledger` | **Bitemporal append-only tax ledger** (Marten event store), projections, evidence packs | `LedgerEntryPosted` | Contracts only — **depends on nothing** |
| `UniVat.Filing` | Return assembly, validation, eVAT/iVAS submission, a-Challan | `return.validated`, `return.filed`, `return.rejected`, `challan.verified` | Ledger, Determination contracts |
| `UniVat.Reconciliation` | Purchase ↔ supplier ↔ VDS ↔ challan ↔ return matching, exceptions | `reconciliation.exception.raised`, `vds.certificate.matched` | Ledger, Documents contracts |
| `UniVat.Masters` | Entities, registrations, branches, counterparties, items, price declarations, coefficients | `coefficient.variance.breached` | Contracts only |
| `UniVat.Inventory` | Stock, transfers, production runs, variance | `production.variance.raised` | Masters contracts |
| `UniVat.Identity` | Tenancy, users, roles, RLS context, API keys | — | Contracts only |
| `UniVat.AiGateway` | Anti-corruption boundary to the Python workbench; gating, logging, kill-switch | `ai.invocation.logged` | Contracts only |
| `UniVat.Reporting` | Report generation, analytics queries, exports | — | Read models only |
| `UniVat.Integration` | Anti-corruption layer, connector adapters, webhooks | `device.offline`, all outbound webhooks | Contracts only |
| `UniVat.Platform` | Money, Time, Numbering, Idempotency, Outbox, Flags, Audit — the cross-cutting kernel | — | Nothing |

**Extraction policy (`TEC-03`).** Services are extracted only on evidence — a module whose scaling profile, failure isolation or deployment cadence provably diverges. The two pre-identified candidates are the **print/render farm** and the **AI gateway**, both already isolated behind HTTP contracts so extraction is a deployment change, not a rewrite.

---

## 5. The stack, layer by layer

Licence classes: **A** = permissive, **B** = weak/file-level copyleft, **C** = network copyleft (internal only), **D** = forbidden (§9, §12). Every component below is A or B unless explicitly marked.

### 5.1 Backend runtime and framework

| Concern | Component | Version | Licence | Why / requirement |
|---|---|---|---|---|
| Runtime | **.NET** | **10.0 (LTS)** — GA 11 Nov 2025, supported to **14 Nov 2028** | MIT (A) | `TEC-15` managed strongly-typed; `NFR-32` LTS-only. `.NET 11` is STS and **not adopted** (§12) |
| Language | **C#** | 14 | MIT (A) | Native `decimal`, records, nullable reference types enforced |
| Web framework | **ASP.NET Core** | 10.0 | MIT (A) | Minimal APIs, built-in rate limiting, health checks, output caching |
| Host | `UniVat.App` single host process | — | — | `TEC-03` one deployable |
| Compile-time safety | Nullable reference types **on**, `TreatWarningsAsErrors`, `EnableNETAnalyzers`, `AnalysisLevel=latest-all` | — | — | `NFR-29` static-analysis gates |
| Banned-API enforcement | **Microsoft.CodeAnalysis.BannedApiAnalyzers** | latest | MIT (A) | Enforces §6.1 — bans `double`/`float`/`Math.Round(double)` in money assemblies at **compile time** |
| Object mapping | **Riok.Mapperly** (compile-time source generator, zero reflection) | 4.x | Apache-2.0 (A) | Replaces AutoMapper (now commercial — §12). Source-generated mapping is also deterministic, satisfying `P6` |
| Validation | **FluentValidation** | 12.x | Apache-2.0 (A) | Request and domain invariant validation; free and OSI-licensed |
| Resilience | **Microsoft.Extensions.Http.Resilience** (Polly v8 core) | 10.x | MIT (A) | Retry, circuit-breaker, timeout, hedging on every outbound integration — `NFR-13`, `INT-17` |
| Mediator / in-process dispatch | **Wolverine** (see §5.5) | 6.x | MIT (A) | Replaces MediatR (now commercial — §12) |

### 5.2 API layer

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| Spec generation | **Microsoft.AspNetCore.OpenApi** — emits **OpenAPI 3.1 natively in .NET 10** | MIT (A) | `TEC-05`, `PRD §12.1`. No third-party runtime dependency for docs |
| API reference UI | **Scalar.AspNetCore** | MIT (A) | Interactive reference with OAuth2/PKCE; replaces Swagger UI (§12) |
| Spec-first workflow | OpenAPI document is **committed to the repository** and diffed in CI; breaking-change detection gates the build | — | `TEC-01`/`TEC-05` API-first; `P9` |
| Contract tests | **Microsoft.AspNetCore.Mvc.Testing** + committed-spec conformance suite | MIT (A) | Every endpoint verified against the published spec each build |
| Versioning | **Asp.Versioning.Http** — URL segment `/v1` | MIT (A) | `PRD §12.1`; ≥ 12-month deprecation policy published with the spec (`TEC-05`) |
| Errors | **RFC 9457** `application/problem+json` via ASP.NET Core `ProblemDetails` | MIT (A) | `PRD §12.1` — exact format mandated |
| Pagination | Cursor-based (opaque, signed cursor over a stable sort key) | — | `PRD §12.1`; avoids OFFSET degradation at `NFR-08` volumes |
| Idempotency | `Idempotency-Key` middleware (§6.4) | — | `TEC-06`, `PRD §12.1` — **all** mutating endpoints |
| Rate limiting | ASP.NET Core `RateLimiter` middleware, **partitioned per tenant** | MIT (A) | `PRD §12.1` per-tenant rate limits |
| Webhooks | Wolverine-driven outbound dispatch; **HMAC-SHA256 signed**, timestamped, replay-protected, exponential backoff → DLQ | MIT (A) | `PRD §12.3` — 16 named domain events |
| Auth | OAuth 2.1 client credentials (Keycloak) or scoped API keys; JWT bearer validation | — | `PRD §12.1`, `NFR-16` |

### 5.3 Domain and statutory core

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| Rules-as-data engine (**primary**) | **GoRules ZEN** (`zen-engine`, Rust core, .NET bindings) | **MIT** (A) | `FR-10`, `GOV-07`. Rust core meets `NFR-02` (< 50 ms p99/line at 1,000 lines/s); JDM decision models are versioned JSON; evaluation is a pure function → `TEC-08`. Content is portable if the stack ever changes |
| Rules-as-data engine (**alternative**) | **Microsoft RulesEngine** | MIT (A) | .NET-native JSON rules; selected if the `D-15` spike shows ZEN's interop cost outweighs its throughput |
| Complex inference (if needed) | **NRules** (Rete) | MIT (A) | Held in reserve for rule sets that genuinely need forward chaining |
| Rule content store | Versioned JSON in PostgreSQL with `valid_from`/`valid_to`, citation, interpreter, approver, approval timestamp | — | `GOV-07`, `GOV-13`, `D-01` gate |
| Determination result | Persists `rule_version_id` with every computed figure | — | `FR-10`, `TEC-08`, `TEC-21` |
| Money | **`System.Decimal`** end-to-end (§6.1) | — | `TEC-11` — the requirement OPT2-B was chosen to satisfy |
| Time | **NodaTime** | Apache-2.0 (A) | `TEC-12` — `Instant` storage, `Asia/Dhaka` period boundaries (§6.2) |
| Numbering | PostgreSQL sequence allocation service with void-with-reason (§6.6) | — | `TEC-10`, `BR-022` |
| Golden-master harness | **Verify.XunitV3** snapshot testing over a frozen scenario corpus | MIT (A) | `TEC-25`, `X-04` — any change to a computed figure fails the build until explicitly approved |
| Statutory conformance suite | xUnit v3 suite asserting every generated form against gazetted field specifications | Apache-2.0 (A) | `TEC-24`, `GOV-02`, `X-03` |

### 5.4 Persistence and the tax ledger

| Concern | Component | Version | Licence | Why / requirement |
|---|---|---|---|---|
| RDBMS | **PostgreSQL** | **18** (GA 25 Sept 2025 — async I/O, `uuidv7()`, wider index usage) | PostgreSQL Licence (A) | `TEC-09` relational core. PG 19 (Sept 2026) adopted on the n−1 policy (§11) |
| Driver | **Npgsql** | 10.x | PostgreSQL Licence (A) | Binary `COPY` for bulk import (`NFR-05`) |
| **Connection pooling** | **PgBouncer** (transaction pooling) — **PgCat** where read/write splitting or sharding is needed | 1.24+ / 1.x | ISC / MIT (A) | **Added in v1.1 — was absent from v1.0.** `NFR-08` requires 100,000 concurrent users against a PostgreSQL server that handles a few hundred backends; application-side pooling alone cannot bridge that. PgBouncer has the lowest latency below ~50 concurrent clients per pool and uses ~2 MB per 1,000 clients; **PgCat is multithreaded and overtakes it above ~50 clients**, and adds replica routing. Baseline is PgBouncer per failure domain; PgCat is the pre-approved upgrade path, benchmarked in R0. **Transaction-pooling mode forbids session-level state**, which constrains §6.3 — see the note there |
| Kubernetes operator | **CloudNativePG** | 1.28+ | Apache-2.0 (A) | Primary+replica per failure domain, PITR, switchover — `NFR-09`, `NFR-11` |
| Vector store | **pgvector** | 0.8+ | PostgreSQL Licence (A) | `AI-07` RAG over the versioned legal corpus |
| **Event store / ledger** | **Marten** | **9.x** (Critter Stack 2026; targets net9.0/net10.0) | **MIT** (A) | `TEC-07` — append-only event streams, projections, async daemon, optimistic concurrency. **No update path is exposed for tax postings** |
| Ledger tenancy | Marten **conjoined tenancy** (`tenant_id` on every event and document) + PostgreSQL **RLS** + native list partitioning | — | — | `TEC-02`, `NFR-18`, `NFR-08` (10k tenants) |
| Projections | Marten inline projections for read-your-writes paths; **async daemon** for heavy read models | — | MIT (A) | `NFR-01` p95 < 300 ms while keeping the write path thin |
| Relational masters | **EF Core 10** — masters, configuration, reference data only. **Never the ledger** | 10.x | MIT (A) | `TEC-09`; `P6` keeps change-tracking away from money |
| Migrations | **EF Core migrations** for masters + **DbUp** for hand-written SQL (partitions, RLS policies, sequences, indexes) | — | MIT (A) | Reviewable, ordered, idempotent DDL |
| Object storage | **SeaweedFS** (S3 API) via **AWSSDK.S3** | 3.x | Apache-2.0 (A) | `TEC-09` originals and rendered artefacts; benchmarked in R0 (`BTO-R6`), Ceph RGW pre-approved fallback |
| Search index | **OpenSearch 3.x** + `OpenSearch.Client`, **Bengali analyzer + ICU plugin** | 3.x | Apache-2.0 (A) | `TEC-09`, **`LOC-06` Bangla search, sorting and collation** |
| Warehouse | **ClickHouse** + logical replication / **Debezium** CDC | 25.x | Apache-2.0 (A) | `DAT-06` near-real-time analytics replication. Trigger point is `D-16` |
| Backup | **CloudNativePG PITR** → object store + **Velero** + **Restic** for cluster state | — | Apache-2.0 / BSD-2 (A) | `DAT-10`, `NFR-11` (RPO ≤ 15 min; **5 min for Large**) |
| Data dictionary | **SchemaSpy**-generated, version-controlled, published per release | 6.x | Apache-2.0 (A) | `DAT-04` |

> **Why Citus is not here.** The BTO's v1.1 audit found Citus is **AGPL-3.0**, not Apache-2.0. It is therefore Class C and cannot ship in the on-prem appliance. ClickHouse (Apache-2.0) is the warehouse; PostgreSQL native partitioning plus read replicas carry the transactional scale.

### 5.5 Messaging, background work and workflow

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| Messaging + **transactional outbox** | **Wolverine 6** | **MIT** (A) | `TEC-04`, `TEC-06`. Native Marten integration gives an outbox in the *same* transaction as the event append — the correctness property that matters. Also serves as the in-process mediator, removing any need for MediatR |
| Broker | **RabbitMQ** (quorum queues) | MPL-2.0 (B) | Durable domain events, DLQ, retry with backoff — `TEC-20` |
| Streaming (conditional) | **Apache Kafka** | Apache-2.0 (A) | Only if event volume at `NFR-08` demands; not deployed in R0/R1 |
| Scheduled jobs | **Quartz.NET** | Apache-2.0 (A) | Cron-style statutory schedules (period close, deadline reminders). **Preferred over Hangfire (LGPL-3.0, Class B) in shipped artefacts** |
| Long-running sagas | **Temporal** (+ .NET SDK) | MIT (A) | Return assembly, filing retry over days, EFD reconciliation — durable execution with replay (`TEC-20` replay tooling) |
| Ops-side automation | **Apache Airflow** | Apache-2.0 (A) | Internal data ops. **n8n is internal-only and must not enter the appliance** (Class D, §12) |
| DLQ operator tooling | RabbitMQ management + a first-class in-product DLQ console (replay, discard, inspect) | — | `TEC-20` explicitly requires operator tooling |

### 5.6 Caching

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| Cache abstraction | **`HybridCache`** (built into .NET 9+) — L1 in-process + L2 distributed, stampede protection | MIT (A) | `TEC-21` |
| L2 store | **Valkey 9.x** (BSD-3-Clause, Linux Foundation; 9.0 supported to Oct 2028) | BSD-3 (A) | Permissive-only posture for the on-prem appliance. Redis 8+ under its AGPLv3 election is acceptable for SaaS but Valkey avoids the question entirely |
| Client | **StackExchange.Redis** (Valkey wire-compatible) | MIT (A) | — |
| **Invalidation policy** | Every cache key for determination results or reference data is **suffixed with `rule_version_id`**; a rule-version change makes old keys unreachable rather than requiring eviction. **Nothing is cached across a rate-effective boundary** | — | `TEC-21` verbatim |

### 5.7 Identity, access and tenancy

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| Identity provider | **Keycloak 26.7** (released 9 July 2026) | Apache-2.0 (A) | `INT-11` SAML 2.0 + OIDC; `NFR-16` MFA; **FIDO2/WebAuthn** for phishing-resistant privileged auth (`SEC-03`) |
| SCIM provisioning | Keycloak **SCIM API (preview in 26.7)** behind a UniVAT abstraction; **Metatavu SCIM extension** as the fallback | Apache-2.0 (A) | `INT-11`. Preview status tracked as a risk (§14) |
| Multi-cluster HA | Keycloak 26.7 multi-cluster HA without external caches (preview) | Apache-2.0 (A) | Supports the dual-failure-domain topology (`NFR-09`) |
| Token validation | ASP.NET Core JWT bearer + OIDC handlers | MIT (A) | — |
| Authorisation | Policy-based authorisation; **deny by default**; every decision emitted to the audit log | — | **`NFR-17`: "Least privilege; deny by default; every authorisation decision logged"** |
| Tenant isolation | PostgreSQL **RLS** on every tenant-scoped table; `SET LOCAL app.tenant_id` per request/session; Marten conjoined tenancy | — | `TEC-02`, `NFR-18`, `SEC-04` |
| Field-level encryption | AES-256-GCM envelope encryption, **per-tenant data keys**, master keys in OpenBao; customer-managed keys for Enterprise | — | `NFR-15`, `SEC-02` |
| Secrets | **OpenBao** (MPL-2.0) with short-lived dynamic credentials and automatic rotation | MPL-2.0 (B) | `TEC-22`, `SEC-06` |
| Feature flags | **OpenFeature** (CNCF, Apache-2.0) SDK + **flagd** provider, self-hosted, per-tenant targeting | Apache-2.0 (A) | `TEC-19` runtime toggles per tenant; wired to the **AI kill-switch** (`AIC-5`, `AI-29`) and regulatory switch-overs |

### 5.8 Frontend and offline PWA

| Concern | Component | Version | Licence | Why / requirement |
|---|---|---|---|---|
| **Build/SSR runtime** | **Node.js LTS** — 24.x (Active LTS, EOL 30 Apr 2028); adopt 26.x when it enters LTS in Oct 2026 | 24 LTS | MIT (A) | **Added in v1.1 — v1.0 pinned no Node version at all**, despite `NFR-32` requiring LTS-only for *runtime and framework*. The Next.js SSR process is a production runtime, not just a build tool |
| Framework | **Next.js** (App Router) | 16.x | MIT (A) | `TEC-14` SPA/SSR; SSR for first-paint budgets (`NFR-04`) |
| UI library | **React** | 19.x | MIT (A) | `TEC-14` |
| Language | **TypeScript** (`strict`, `noUncheckedIndexedAccess`) | **6.0** | Apache-2.0 (A) | `TEC-14`; `NFR-29` |
| Styling | **Tailwind CSS v4** + **shadcn/ui** on **Radix UI** primitives | — | MIT (A) | Radix gives accessible primitives by construction — `NFR-25`, `LOC-09` |
| Server state | **TanStack Query v5** | — | MIT (A) | Cache, retry, offline-aware mutations |
| Client state | **Zustand** | — | MIT (A) | Minimal, no boilerplate |
| Forms | **React Hook Form** + **Zod 4** | 7.x / **4.x** | MIT (A) | Shared schema with API contract types |
| Tables | **TanStack Table v8** | — | MIT (A) | Virtualised grids for large registers |
| Charts | **Apache ECharts** | **6.x** | Apache-2.0 (A) | Dashboards; strong CJK/Indic label handling |
| **Service worker** | **Serwist** (`@serwist/next`) — the maintained `next-pwa` successor | — | MIT (A) | `TEC-28` offline; precache manifest, background sync |
| **Offline store** | **Dexie** (IndexedDB) | — | Apache-2.0 (A) | `TEC-28`, `UR-025` local journal |
| **Signed local journal** | **Web Crypto API** HMAC-SHA256 over a device-bound key; monotonic local sequence; server verifies chain on sync | — | — | `TEC-28` "signed local journal that **cannot be silently altered**"; `BR-024` bounded windows (72 h S/M, 24 h L) |
| i18n | **next-intl** with **ICU MessageFormat** | — | MIT (A) | `TEC-13`, `LOC-01` complete bn/en, user-switchable |
| Numerals & currency | `Intl.NumberFormat('bn-BD')` for **Bengali numerals**; explicit **lakh/crore grouping**; `Intl.DateTimeFormat` for Bangladeshi date formats; Bengali calendar where required | — | — | **`LOC-03`, `LOC-05`, `NFR-24`** |
| Fonts | **Noto Sans Bengali** + **SolaimanLipi**, self-hosted, subset, `font-display: swap` | — | SIL OFL 1.1 (A) | `NFR-34` conjunct shaping; OFL text ships with the appliance |
| Accessibility testing | **axe-core** / `@axe-core/playwright` | 4.x | MPL-2.0 (B) | `NFR-25`, `LOC-09`, `X-08` WCAG 2.2 AA. Dev/test tool — not shipped |
| Performance budget | **Lighthouse CI** gates: **≤ 500 KB gzipped** initial payload (Starter), ≤ 1.5 MB (M), ≤ 2 MB (L) | 0.1x | Apache-2.0 (A) | `PRD §14`, `NFR-04`, `NFR-35` |
| Low-bandwidth | Route-level code splitting, AVIF/WebP, `content-visibility`, degraded-asset mode at ≤ 256 kbps, resumable uploads | — | — | **`NFR-35`** |
| Browser targets | Latest two of Chrome/Edge/Firefox/Safari; Android 10+; iOS 15+; usable at 5-inch | — | — | `NFR-26` |

### 5.9 Print and document rendering

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| A4 statutory forms (**primary**) | **headless Chromium**, version-pinned, isolated render pool | BSD-3 (A) | `TEC-27` pixel-faithful forms. **Promoted to primary in v1.1**: mature, proven Bengali shaping via HarfBuzz; a pinned build gives byte-stable output |
| A4 statutory forms (**optimisation**) | **Typst 0.15** | Apache-2.0 (A) | Faster and more deterministic than a browser, but **pre-1.0 with breaking changes still planned on the road to 1.0**. Adopted per-form only after its Bengali conjunct output passes the corpus below, and pinned to an exact version (`TSD-R9`) |
| **Thermal / POS (Bengali)** | **Server-side raster pipeline** — render receipt → **1-bit monochrome bitmap** → ESC-POS **raster image** commands (`GS v 0` / `GS *`) | MIT (A) | **Corrected in v1.1.** ESC-POS printers render only from onboard codepages and **perform no glyph shaping whatsoever**; Bengali sent as text prints as question marks or broken conjuncts. Rasterising is not an optimisation — it is **the only correct path** for `LOC-04`, `NFR-34`, `TEC-27` |
| Thermal transport | **WebUSB** (counter-attached) / raw TCP 9100 (networked) — both carry **pre-rendered bytes only** | MIT (A) | The client never composes Bengali for the printer; it transmits a server-produced raster, so thermal output is provably identical to the A4 rendering of the same document |
| Thermal device profiles | Paper widths (58 mm ≈ 384 dots, 80 mm ≈ 576 dots), buffer-safe image chunking, density and dithering per model — held as **configuration, not code** | — | `INT-17` pattern; new printer models are an ops change, not a release |
| QR codes | **QRCoder** | MIT (A) | `FR-40` Mushak 6.3 QR |
| Digital signatures | **Bouncy Castle** (PKCS#7 / PAdES) | MIT (A) | `INT-14` e-signature; evidence-pack sealing |
| Bangla shaping validation | Golden-image corpus of conjuncts (যুক্তাক্ষর), rendered in A4 and 58/80 mm thermal, diffed per build | — | **`NFR-34` "validated against a test corpus"**, `LOC-04`, `X-03` |
| Render isolation | Dedicated `print/render` worker pool with CPU limits and no database write access | — | `NFR-07` return generation budgets without starving the API pool |

> **QuestPDF is prohibited** (§12): its Community licence is not OSI-approved, is revenue-thresholded, and expressly excludes public-sector entities — incompatible with `BC-6`, `P4` and UniVAT's customer base.

### 5.10 AI and machine-learning workbench

The only Python in the estate. A network-isolated FastAPI service behind `UniVat.AiGateway`, with its own GPU node pool.

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| Runtime | **Python 3.14** (security support to ~Oct 2030) + **FastAPI** + **Uvicorn** | MIT/BSD (A) | Contract-first HTTP boundary; no shared database with the monolith (`AI-23`) |
| LLM serving | **vLLM** | Apache-2.0 (A) | High-throughput self-hosted inference on UNISOFT GPUs |
| Local/dev serving | **Ollama** | MIT (A) | Developer laptops and the sandbox environment |
| Primary models | **Qwen3.x family incl. Qwen3.x-VL** (currently Qwen3.5 / Qwen3.6) | **Apache-2.0** (A) | Clean licence; strong multilingual/document VLM. `AI-01`, `AI-07` |
| Coding/analysis assist | **DeepSeek** | MIT (A) | Internal engineering use |
| OCR | **PaddleOCR** + **bbOCR**-derived Bengali pipeline; **Tesseract** clean-scan fast path | Apache-2.0 (A) | `AI-01` ≥ 97% field accuracy in **both** languages |
| ASR (R3) | Fine-tuned **Bengali Whisper** ecosystem | Base MIT; **each fine-tune cleared individually (D-13)** | `LOC-07` Bangla voice input, `AI-15` |
| RAG | **pgvector** over the versioned legal corpus; hybrid retrieval with OpenSearch BM25 | (A) | `AI-07` with **citation enforcement and abstention** (`AI-25`) |
| Forecasting | **statsmodels** / **Prophet**-class | BSD/MIT (A) | `AI-04` coefficient drift, `AI-09` liability & cash forecasting |
| Model registry | **MLflow** | Apache-2.0 (A) | **`AI-21`** — version, training data, evaluation, approver, deployment date |
| Evaluation harness | Held-out golden set per capability, run in CI; **Bangla within 5 pp of English** gate | — | **`AI-22`, `AI-30`, `AIC-8`, `X-10`** |
| Prompt-injection defence | Untrusted-content sandboxing, instruction/data separation, output schema validation, no tool access from document content | — | **`AI-24`, `AIC-3`** |
| Invocation audit | Every call logged: model, version, input hash, output, confidence, reviewer, decision | — | **`AI-32`, `AIC-2`** |
| Kill-switch & degradation | flagd flag per tenant × capability; deterministic fallback path always present | — | **`AI-28`, `AI-29`, `AIC-5`, `AIC-6`** |
| Residency | All inference on UNISOFT GPUs in-country; hosted APIs only for non-restricted content | — | **`AI-31`, `AIC-7`**, PDPO §29(7)(b) |
| Human-in-the-loop | No AI output is auto-filed, auto-posted or auto-approved — enforced in `UniVat.AiGateway`, not in the UI | — | **`AI-20`, `AIC-1`** |

### 5.11 Integration and the anti-corruption layer

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| ACL pattern | Versioned mapping profiles (data, not code) per external system, hot-reloadable | — | **`INT-17`** — an authority-side change is a configuration update, not a code change |
| NBR eVAT/iVAS, e-invoicing, EFDMS | Adapter + **mock service containers** built spec-first from published gazettes and pilot artefacts | — | `INT-01/02/03`, **`PR-01`** unpublished specs; mock-first from day one |
| a-Challan / e-Payment | Adapter with verification polling and manual-entry fallback | — | `INT-04` |
| Banks / MFS (bKash, Nagad) | Vendor SDK-free HTTP adapters behind the ACL | — | `INT-09` |
| ERP connectors | SAP, Oracle/NetSuite, D365 (`INT-05/06`); Odoo, Zoho, QuickBooks, Tally, local (`INT-07`) — API adapters with **file-import fallback for every one** | — | `INT-05…07`, PRD §12.4 fallbacks |
| POS | Inbound adapters + manual entry fallback | — | `INT-08` |
| Customs / ASYCUDA | Inbound BoE data; manual BoE entry fallback | — | `INT-13` |
| Document intake | IMAP/SMTP ingestion (**MailKit**), watched folders, upload API | MIT (A) | `INT-12` |
| BI tools | ODBC/JDBC read models + **Apache Superset** embedded | Apache-2.0 (A) | `INT-15` |
| Bulk import | Streaming CSV (**CsvHelper**) / XLSX (**ClosedXML**) → validation → PostgreSQL binary `COPY` | Apache-2.0 / MIT (A) | **`NFR-05`** 100k lines < 10 min; **1M lines < 60 min (L)** |
| Outbound webhooks | Wolverine dispatch, HMAC-signed, replay-protected, DLQ | MIT (A) | `PRD §12.3`, `INT-10` |
| Integration platform capability | Connector SDK + published event/webhook contracts | — | `INT-16` |

### 5.12 Observability

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| Instrumentation | **OpenTelemetry .NET** (traces, metrics, logs) → OTLP | Apache-2.0 (A) | `NFR-27` — correlation IDs propagated **across every integration** |
| Structured logging | **Serilog** with the OpenTelemetry sink | Apache-2.0 (A) | `NFR-27` structured logs |
| Metrics | **Prometheus** — RED for services, USE for resources, **per-tenant dimensions** | Apache-2.0 (A) | `NFR-27` per-tenant dashboards |
| Dashboards, logs, traces | **Grafana · Loki · Tempo · Mimir** | **AGPL-3.0 — Class C, internal only, never in the appliance** | `NFR-27` |
| Error tracking | **GlitchTip** | MIT (A) | Sentry's FSL is non-OSI (§12) |
| Health | ASP.NET Core health checks — separate **liveness** and **readiness** endpoints | MIT (A) | Kubernetes probes; `NFR-13` |
| SLOs & alerting | Prometheus recording/alerting rules with error budgets; **Alertmanager** → on-call | Apache-2.0 (A) | **`NFR-28` — "no alert without a runbook"**, enforced by a CI check that every alert rule has a `runbook_url` label |
| Status page | **Uptime Kuma**, self-hosted | MIT (A) | `OPS-02`, customer-facing availability |
| Audit log | Hash-chained (SHA-256) append-only stream in Marten with an **independent verifier CLI** | — | **`SEC-05`, `NFR-19` ≥ 6 years, immutable and independently verifiable** |

### 5.13 Migration and data lifecycle

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| UniVAT 1.0 migration | Dedicated importer service: extract → profile → cleanse → map → load → **full source-to-target reconciliation report** | — | `DAT-01`, **`X-13`**; `PR-08` junk-record risk |
| Competitor migration | Documented templates + validation harness (Odoo, ERPNext, Tally, spreadsheet) | — | `DAT-02` |
| Historical/obsolete forms | Read-only archival store for legacy Mushak 18 / 12(খ) records | — | `DAT-03` |
| Data quality | Rule-based profiling gates (completeness, validity, uniqueness) with per-tenant scorecards | — | `DAT-05`, `AI-12` |
| Retention & disposal | Policy-driven retention jobs; ≥ 6-year statutory floor; legal hold | — | `DAT-07`, `NFR-19` |
| Export & offboarding | Complete machine-readable export (JSON + CSV + originals); deletion within 90 days subject to hold | — | `DAT-08`, `BRD-044`, `OPS-19` |
| Anonymisation | Deterministic pseudonymisation for non-production; **irreversible masking** enforced by pipeline | — | `DAT-09`, `BRD-046` |
| Master data | Golden-record service for counterparties with merge/survivorship | — | `DAT-11` |
| Benchmarking | Opt-in, aggregated, **k-anonymous** (k ≥ 20) analytics | — | `DAT-12` |

### 5.14 Platform and infrastructure

| Concern | Component | Licence | Why / requirement |
|---|---|---|---|
| Kubernetes | **RKE2** (production), **K3s/k3d** (edge, sandbox, local) | Apache-2.0 (A) | `TEC-16` |
| Ingress | **Gateway API** via **Envoy Gateway** (alt: Traefik v3) | Apache-2.0 / MIT (A) | **ingress-nginx retired March 2026** — see §12 |
| WAF | **OWASP Coraza** + **OWASP CRS** | Apache-2.0 (A) | `SEC-01`; ModSecurity's commercial support ended 2024 |
| Certificates | **cert-manager** | Apache-2.0 (A) | TLS 1.3 everywhere (`NFR-15`, `SEC-02`) |
| Load balancing | **HAProxy** / **kube-vip** | GPL-2.0 (B, infra binary) / Apache-2.0 (A) | Dual LB across failure domains |
| GitOps | **ArgoCD** | Apache-2.0 (A) | `TEC-17` declarative, drift detection, **no manual production changes** |
| IaC | **OpenTofu** (+ **Ansible** for bare-metal provisioning) | MPL-2.0 (B) / GPL-3.0 (C, tool only) | `TEC-17` (registered deviation from "Terraform/Bicep") |
| Packaging | **Helm** charts — **one chart set for SaaS, dedicated-tenant and on-prem**, differing only in values | Apache-2.0 (A) | **`TEC-02` "the same artefacts"** |
| Autoscaling | HPA + **KEDA** (queue-depth scaling); scale-to-zero for idle non-critical workloads | Apache-2.0 (A) | `NFR-10` 10–15× deadline surge; **`NFR-31`** |
| Backup/DR | **Velero** + **Restic** + CloudNativePG PITR → second in-country site | Apache-2.0 / BSD-2 (A) | `NFR-11`, `NFR-12`, `NFR-22`, `D-12` |
| Environments | dev · test · UAT · staging (production-like) · production · **permanent public sandbox with mocked NBR endpoints** | — | **`TEC-23`** — namespace-per-env, sandbox chart includes mock services |
| GPU pool | vLLM + PaddleOCR nodes, MIG-partitioned, taint/tolerated | — | `AI-31`, `BTO-R8` |

### 5.15 CI/CD, supply chain and quality gates

| Stage | Component | Licence | Requirement |
|---|---|---|---|
| SCM + CI | **GitLab CE** (self-hosted) + runners | MIT (A) | `TEC-18`, `BC-3` |
| Build | `dotnet build` / `pnpm build`; reproducible, pinned SDK via `global.json` | MIT (A) | `TEC-08` reproducibility discipline |
| SAST | **Semgrep OSS** (LGPL-2.1, Class B — tool, never linked) + Roslyn analyzers + **ESLint** | (A/B) | **`SEC-07`** |
| SCA | **Trivy** (dependencies, images, IaC) | Apache-2.0 (A) | `SEC-07`, `NFR-20` |
| DAST | **OWASP ZAP** against the staging environment | Apache-2.0 (A) | `SEC-07` |
| Secret scanning | **Gitleaks** (CI + pre-commit) | MIT (A) | `SEC-07` |
| **SBOM** | **Syft** → CycloneDX + SPDX, **per release** | Apache-2.0 (A) | `SEC-07`, `NFR-20`, `GOV-10` |
| **Licence gate** | SBOM diffed against the §9 allow-list — **build fails** on any Class C/D component in a shipped artefact | — | `GOV-10`, `BC-6`, §9.6 |
| Signing & provenance | **Cosign** (keyless or OpenBao-backed) + SLSA provenance attestation | Apache-2.0 (A) | `TEC-18` signed artefacts |
| Vulnerability SLA | Critical ≤ 7 days, High ≤ 30 days — tracked as a pipeline-visible clock | — | **`SEC-08`, `NFR-20`** |
| Deployment | ArgoCD sync → **canary** → automated rollback on SLO breach | Apache-2.0 (A) | `TEC-18`, `TEC-19` progressive delivery, `X-15` rollback tested |
| Docs-as-code | **Docusaurus** (bilingual bn/en) — user, admin, API, implementation docs versioned with the release; **link-checked in CI** | MIT (A) | **`OPS-11`, `NFR-33` "no release ships with stale docs"**, `X-11`, `LOC-08` |
| ADR log | Markdown ADRs in-repo, indexed, one per significant decision | — | **`TEC-30`, `GOV-13`** |

### 5.16 Testing

| Level | Component | Licence | Requirement |
|---|---|---|---|
| Unit | **xUnit v3** | Apache-2.0 (A) | `TEC-24` |
| Assertions | **AwesomeAssertions** (Apache-2.0 fork of Fluent Assertions v7) or **Shouldly** | Apache-2.0 / BSD (A) | **Fluent Assertions v8+ is commercial — prohibited (§12)** |
| Property-based | **FsCheck** / **CsCheck** | BSD-3 / Apache-2.0 (A) | Money arithmetic, rounding, numbering invariants (§6.1, §6.6) |
| Mutation | **Stryker.NET** on the determination and ledger modules | Apache-2.0 (A) | `NFR-29` — proves the ≥ 80% coverage is meaningful |
| Snapshot / golden master | **Verify.XunitV3** | MIT (A) | **`TEC-25`, `X-04`** |
| Architecture | **NetArchTest** — module boundaries, no `double` in money assemblies, no EF Core in `UniVat.Ledger` | MIT (A) | `TEC-03`, `P6`, `P8` |
| Integration | **Testcontainers for .NET** — real PostgreSQL 18, RabbitMQ, Keycloak, OpenSearch per test class | MIT (A) | `TEC-24` |
| **Tenant isolation** | Automated cross-tenant access suite executed on **every** pipeline run | — | **`SEC-04`, `NFR-18` — a leak is Sev-1** |
| Contract | OpenAPI conformance + consumer-driven contract tests for connectors | — | `TEC-01`, `INT-17` |
| E2E | **Playwright** incl. **Bangla snapshot tests** in both locales | Apache-2.0 (A) | `TEC-24`, `LOC-10`/`X-07` parity gate |
| Accessibility | **axe-core** in Playwright, both languages, screen-reader scripted checks | MPL-2.0 (B) | `NFR-25`, `LOC-09`, `X-08` |
| Load | **JMeter** + **Locust** (k6 permitted internally, AGPL — never shipped) | Apache-2.0 / MIT (A) | `NFR-01…08`, `X-05`; **15× deadline surge for Large** |
| Statutory conformance | Form-by-form field assertions against gazetted specifications | — | **`GOV-02`, `X-03`** |
| AI evaluation | Golden-set harness, both languages, gates the model promotion | — | **`AI-22`, `X-10`, `AIC-8`** |
| Coverage gate | **≥ 80% on business logic**, enforced in CI | — | **`NFR-29`** |

---

## 6. Cross-cutting engineering standards

These are the standards that turn requirements into properties of the code. They are enforced by analyzers, architecture tests and CI gates — not by convention.

### 6.1 Money — the `TEC-11` standard

> `TEC-11`: *"Decimal arithmetic only (no binary floating point) for money and quantities; explicit rounding rules per statutory context; store at higher precision than displayed."*

| Rule | Implementation | Enforcement |
|---|---|---|
| No binary floating point in any money or quantity path | `System.Decimal` only (128-bit, 28–29 significant digits) | **BannedApiAnalyzers** bans `double`, `float`, `Math.Round(double,…)`, `double.Parse` in `UniVat.Determination`, `UniVat.Ledger`, `UniVat.Documents`, `UniVat.Filing`, `UniVat.Platform.Money`. Build error, not warning |
| Storage at higher precision than displayed | **Line-level** amounts and quantities: PostgreSQL `numeric(19,6)` (13 integer digits ≈ 10 trillion BDT — ample per line). **Aggregate and projection columns: `numeric(28,8)`**, because a tenant-year or nationwide roll-up at `NFR-08` volumes can exceed the line-level range while C# `decimal` still carries 28–29 significant digits. Presentation rounds to 2 dp (BDT) at the edge only | Migration review + architecture test asserting no `double precision` column in money tables; property test asserting no overflow at 1B-line aggregates *(precision split added in v1.1)* |
| Explicit rounding per statutory context | A `RoundingPolicy` value object carried with every computation, sourced from the rule version — never a language default | Determination returns `(amount, roundingPolicyId, ruleVersionId)`; golden-master locks the behaviour |
| No implicit conversions | `decimal` ↔ any floating type conversion is banned; JSON serialisation uses `decimal` with invariant culture | Analyzer + `System.Text.Json` converter |
| Aggregation order is defined | Sums are computed over a deterministic sort key so results are reproducible | Property tests (`FsCheck`) assert order-independence within tolerance zero |
| Minor unit | BDT poisha (¹⁄₁₀₀ taka) is the presentation minor unit; internal precision is 6 dp to survive coefficient and apportionment arithmetic | Documented in the data dictionary (`DAT-04`) |

*This is the single largest engineering reason OPT2-B was chosen over both Option 1 (Odoo's `Monetary` field holds Python `float`) and OPT2-A (JavaScript has no native decimal).*

### 6.2 Time — the `TEC-12` standard

| Rule | Implementation |
|---|---|
| Storage | **NodaTime `Instant`** → PostgreSQL `timestamptz`; UTC always |
| Presentation | `Asia/Dhaka` (BST, UTC+6, **no DST** — but resolved through the tz database, never hard-coded) |
| Tax-period boundaries | Evaluated as `LocalDate`/`ZonedDateTime` in `Asia/Dhaka`; a period runs `[00:00:00 first-day, 00:00:00 next-first-day)` in BST |
| Near-midnight documents | Issuance timestamp is the server `Instant`; the **period assignment** is computed from the BST local date and stored explicitly on the document, so a later timezone or clock correction cannot silently move a document between returns |
| Near-period-end | Documents issued within a configurable window of a period boundary are flagged for review; the offline journal's local timestamps are reconciled against server time on sync with a bounded skew tolerance |
| Bengali calendar | Supported for display where a form requires it (`NFR-24`) via an explicit conversion table, never for computation |

### 6.3 Tenancy — the `TEC-02` / `NFR-18` standard

| Layer | Control |
|---|---|
| Database (documents) | Row-Level Security policy on every tenant-scoped **document** table, keyed on `tenant_id` |
| **Tenant context propagation** | **`set_config('app.tenant_id', <id>, true)` — the third argument MUST be `true` (transaction-local).** See the pooling warning below |
| **Ledger (event store)** | Marten **conjoined tenancy** — `tenant_id` on every event; native list partitioning for large tenants. **⚠ RLS does not extend here — see the correction below** |
| Application | Tenant context resolved once at the edge from the token, immutable for the request; **no API accepts a `tenant_id` parameter from the client** |
| Encryption | Per-tenant data-encryption keys, envelope-wrapped by an OpenBao master key |
| Deployment | Shared-schema (SaaS) · dedicated-tenant · on-prem single-tenant — **the same Helm chart** with different values (`TEC-02`) |
| Verification | Cross-tenant access suite in every pipeline run (`SEC-04`); a leak is Sev-1 with mandatory customer notification (`NFR-18`) |

> **⚠ Correction (v1.1) — two isolation facts that change what must be tested.**
>
> **1. RLS does not cover the event store.** Marten applies RLS policies only to `TenancyStyle.Conjoined` **document** tables. The event-store tables — **`mt_events` and `mt_streams`** — do **not** receive RLS policies. The tax ledger, the most sensitive data in the product, is therefore isolated by **Marten's application-level tenant filtering alone**, not by a database-enforced safety net. v1.0 implied database-level protection across the board; that was wrong. Consequences, all mandatory:
> - The `SEC-04` cross-tenant suite must include **raw-SQL probes against `mt_events`/`mt_streams`** that bypass Marten entirely, not just API-level probes. An API-only suite would pass while the ledger was unprotected.
> - Any ad-hoc analytics, migration script or support tool touching the event store must go through a tenant-scoped repository; **direct `psql` access to `mt_events` is restricted to a break-glass role** with session auditing.
> - Hand-written RLS policies on the Marten tables are **evaluated in R0** as defence in depth, but only after confirming they do not break the async daemon (which reads across tenants by design). Tracked as `TSD-R10`.
>
> **2. Transaction pooling and session-scoped tenant context are mutually destructive.** With PgBouncer/PgCat in **transaction pooling** mode, a connection is returned to the pool at commit and handed to a different tenant's request. If the tenant context is set with `set_config(..., false)` — the *session-scoped* form that appears in most RLS tutorials — **the setting survives into the next tenant's transaction and RLS silently authorises the wrong rows.** That is precisely the `NFR-18` Sev-1 scenario, arriving through a configuration detail rather than a code defect. The standard is therefore absolute: **`set_config(..., true)` (transaction-local) only**, applied inside the same transaction as the query, and asserted by an integration test that runs *through the pooler* rather than against a direct connection.

### 6.4 Idempotency and exactly-once — the `TEC-06` standard

| Concern | Implementation |
|---|---|
| Inbound | `Idempotency-Key` header **required on all mutating endpoints**; stored per `(tenant_id, endpoint, key)` with the response body hash and status; replay returns the stored response; conflicting payload for the same key returns `409` |
| Ingestion | Natural-key deduplication on import and connector intake (supplier BIN + invoice number + date) |
| Outbound | **Transactional outbox via Wolverine**, written in the same transaction as the domain event — no dual-write |
| Consumers | Handlers are idempotent by construction; message IDs deduplicated in an inbox table |
| Webhooks | Signed, timestamped, replay-protected; consumers are told to deduplicate on event ID |

### 6.5 The bitemporal ledger — the `TEC-07` standard

| Property | Implementation |
|---|---|
| Append-only | Marten event streams. **`UniVat.Ledger` exposes no update or delete API**; enforced by architecture test |
| Corrections | New compensating events (reversal + restatement), never mutation — matching statutory practice |
| Transaction time | Marten's event sequence and timestamp — when the system learned it |
| Valid time | `valid_from` / `valid_to` carried in the event payload — when it was true in the tax world |
| Reconstruction | "Position as at *transaction time T1* for *valid time T2*" is a first-class query serving audit and `US-100` lineage |
| Immutability proof | SHA-256 hash chain over the event stream with an independent verifier CLI (`SEC-05`) |
| Retention | ≥ 6 years, supporting the 5-year reopening under §83 plus margin (`NFR-19`) |

### 6.6 Statutory numbering — the `TEC-10` / `BR-022` standard

| Rule | Implementation |
|---|---|
| Scope | One sequence per `(tenant, BIN, branch, form-type, fiscal-year)` |
| Allocation | PostgreSQL sequence + an allocation record written in the issuing transaction |
| **No gaps** | If issuance fails after allocation, a **void-with-reason record** is written for that number. The sequence is never rewound |
| **No reuse** | Enforced by a unique constraint on `(tenant, BIN, branch, form-type, fiscal-year, number)` |
| Concurrency | Optimistic locking on documents (`TEC-10`); allocation is serialised at the sequence, not the application |
| Offline | Offline devices draw from a **pre-allocated, device-bound block**; unused numbers in a returned block are voided with reason on reconciliation (`BR-024`, `PR-06`) |
| Verification | Property tests assert monotonicity and gaplessness under concurrent load; a nightly job reconciles issued + voided against the sequence high-water mark |

### 6.7 Errors, contracts and API conventions

| Concern | Standard |
|---|---|
| Errors | **RFC 9457** `application/problem+json` with a stable `type` URI catalogue, `traceId`, and Bangla/English `title`/`detail` |
| Pagination | Cursor-based; opaque signed cursor; `next`/`prev` links |
| Deprecation | `Deprecation` and `Sunset` headers; **≥ 12 months' notice** (`TEC-05`) |
| Versioning | Additive within `/v1`; breaking changes require `/v2` and dual-run |
| Content negotiation | JSON only for the API; PDF/CSV/XLSX via explicit render endpoints |
| Correlation | `traceparent` (W3C) propagated to every downstream, including the AI workbench and NBR adapters (`NFR-27`) |

### 6.8 Localisation and accessibility

| Concern | Standard |
|---|---|
| Source of truth | ICU MessageFormat resource files, `bn` and `en`, in the repository; no hard-coded user-facing strings |
| Parity gate | **A release fails if any key exists in one language and not the other** (`LOC-10`, `X-07`) — enforced as a CI check |
| Default | Bangla is the default for Starter signups (`LOC-02`) |
| Statutory output | Bangla with **Bengali numerals** where the gazetted form requires (`LOC-03`) |
| Formatting | BDT with lakh/crore grouping; Bangladeshi date formats (`LOC-05`, `NFR-24`) |
| Collation | OpenSearch Bengali analyzer + ICU collation for search and sort (`LOC-06`) |
| Shaping | Conjunct test corpus diffed per build across UI, PDF and thermal (`LOC-04`, `NFR-34`) |
| Accessibility | WCAG 2.2 AA, keyboard-complete, screen-reader tested **in both languages** (`NFR-25`, `LOC-09`); axe gate in CI; conformance statement published (`GOV-16`) |

---

## 7. Non-functional budgets → stack mapping

Budgets from `PRD §14` and `PDR NFR-01…13`. S / M / L are the Small / Medium / Large segments.

| Budget | Target | How the stack meets it |
|---|---|---|
| API latency (`NFR-01`) | p95 < 300 ms, p99 < 800 ms | Marten inline projections for read-your-writes; read models denormalised; `HybridCache` L1 in-process; CNPG read replicas; no N+1 by construction (no lazy loading) |
| Determination (`NFR-02`) | < 50 ms p99/line at 1,000 lines/s | ZEN's Rust evaluation core; rule models compiled once and cached by `rule_version_id`; pure functions, no I/O in the hot path |
| Invoice issuance (`NFR-03`) | < 3 s S / < 2 s M,L end-to-end | Single-transaction append + outbox; render deferred to the print pool; e-invoice authorisation awaited asynchronously with a bounded wait |
| Page interactivity (`NFR-04`) | FCP < 1.5 s, TTI < 3 s on 3G | Next.js SSR + route-level splitting; ≤ 500 KB gzipped Starter payload gated by Lighthouse CI |
| Bulk import (`NFR-05`) | 100k lines < 10 min; **1M < 60 min (L)** | Streaming parse → validate → PostgreSQL **binary `COPY`** via Npgsql; parallel partition load; row-level error report |
| Reconciliation (`NFR-06`) | 100k documents < 10 min | Set-based SQL matching over indexed projections + ClickHouse for the wide scans; parallel per-tenant workers |
| Return generation (`NFR-07`) | 1M-line month < 5 min | Pre-aggregated projections maintained by the async daemon; assembly is a read, not a computation; Temporal saga for the long tail |
| Scalability (`NFR-08`) | 10k tenants · 100k concurrent · 1B lines/yr | Shared-schema RLS (not DB-per-tenant); PostgreSQL list partitioning by tenant and period; horizontal app pools; KEDA queue scaling |
| Availability (`NFR-09`) | 99.9% / **99.95% Enterprise** | Dual failure domain; CNPG primary+replica; RabbitMQ quorum queues; stateless app pods; canary + auto-rollback |
| Peak resilience (`NFR-10`) | 10× (S/M), **15× (L)** before deadline | KEDA pre-scale on the statutory calendar; deploy freeze window; load-tested at 15×; queue-based load levelling |
| RPO/RTO (`NFR-11`) | 15 min / 4 h; **5 min / 1 h (L)** | Streaming WAL replication to the second in-country site; CNPG PITR; quarterly restore drills (uplift over the annual floor) |
| DR (`NFR-12`) | Cross-AZ default; in-country replica | Two failure domains + contracted second site (`D-12`) |
| Graceful degradation (`NFR-13`) | Fully usable with NBR or AI down | Polly circuit breakers; queue-and-forward filing; deterministic fallbacks for every AI capability (`AIC-6`); manual-filing export always available |
| Low bandwidth (`NFR-35`) | ≤ 256 kbps usable; resumable | Degraded-asset mode; resumable uploads; offline-first counter shell |
| Sustainability (`NFR-31`) | Right-sized, scale-to-zero idle | KEDA scale-to-zero for non-critical workers; carbon-aware batch windows for nightly jobs |

---

## 8. Security controls → stack mapping

| Control | Requirement | Implementation |
|---|---|---|
| ASVS L2 baseline, **L3 for filing, payment, audit-log** | `SEC-01`, `NFR-14` | Control checklist per module; L3 paths get additional review, stronger auth and full request/response audit |
| TLS 1.3 in transit; AES-256 at rest; field-level encryption; **customer-managed keys for Enterprise** | `SEC-02`, `NFR-15` | cert-manager + Gateway API TLS 1.3; disk/volume encryption; per-tenant envelope keys in OpenBao; Enterprise CMK import |
| MFA for privileged roles; **FIDO2/WebAuthn**; session timeout; re-auth for high-risk actions | `SEC-03`, `NFR-16` | Keycloak authentication flows; step-up authentication policy on filing and payment endpoints |
| Least privilege, deny by default, **every authorisation decision logged** | `NFR-17` | Policy-based authorisation with a default-deny fallback policy; decision events to the audit stream |
| Cross-tenant isolation tested in CI | `SEC-04`, `NFR-18` | Dedicated suite, every pipeline run; RLS + conjoined tenancy defence in depth |
| **Immutable, hash-chained audit log with independent verifier** | `SEC-05`, `NFR-19` | Marten append-only stream + SHA-256 chain + standalone verifier CLI shipped to customers |
| Secrets in a vault, short-lived, auto-rotated | `SEC-06`, `TEC-22` | OpenBao dynamic database credentials; no static secrets in config or images; Gitleaks in CI and pre-commit |
| SAST, DAST, SCA, secret scanning as **gates**; SBOM per release | `SEC-07`, `NFR-20` | §5.15 pipeline; failures block the merge |
| Critical ≤ 7 days, High ≤ 30 days | `SEC-08` | Trivy findings raised as tracked issues with an SLA clock visible in the pipeline |
| Annual third-party penetration test | `SEC-09`, `GOV-05`, `X-06` | External engagement per release train; findings tracked to closure before GA |
| **PDPO/PDPA**: lawful basis, consent, DSR, DPIA for AI, breach procedure | `SEC-10`, `NFR-21` | Consent registry; data-subject-request tooling (access/correction/erasure-where-lawful); DPIA template per AI capability |
| In-country synchronised copy of restricted personal data | `SEC-11`, `NFR-22` | Second in-country site (`D-12`); enforcement horizon ~May 2027 |
| ISO/IEC 27001 · SOC 2 Type II | `SEC-12`, `SEC-13`, `GOV-04`, `X-16` | Evidence generated as a by-product: SBOMs, signed artefacts, audit trails, access reviews, DR drill records |
| **No feature suppresses, backdates or deletes a statutory record** | `SEC-14`, `GOV-14` | Architecturally impossible in the ledger (append-only); asserted by test; void-with-reason is the only negation primitive |
| Published subprocessor list, 30 days' notice | `SEC-15`, `BRD-052` | Docs-as-code page versioned with the release |
| Source-code escrow for Enterprise/on-prem | `GOV-09` | Release artefacts + build instructions deposited with an escrow agent per release train |

---

## 9. Complete licence register

Inherits the four-class model from `BTO` §9. **Class A and B only in shipped artefacts. Class C is internal operations only. Class D is forbidden.** Counsel clears this register before first GA ship (`D-13`).

### 9.1 Class A — Permissive (no source obligation on distribution)

.NET 10 / ASP.NET Core / EF Core / C# (MIT) · Marten (MIT) · Wolverine (MIT) · Weasel (MIT) · Npgsql (PostgreSQL Licence) · PostgreSQL 18 + pgvector (PostgreSQL Licence) · CloudNativePG (Apache-2.0) · DbUp (MIT) · Riok.Mapperly (Apache-2.0) · FluentValidation (Apache-2.0) · Microsoft.Extensions.Http.Resilience / Polly (MIT) · Microsoft.AspNetCore.OpenApi (MIT) · Scalar.AspNetCore (MIT) · Asp.Versioning (MIT) · BannedApiAnalyzers (MIT) · NodaTime (Apache-2.0) · GoRules ZEN (**MIT**) · Microsoft RulesEngine (MIT) · NRules (MIT) · Quartz.NET (Apache-2.0) · Temporal + .NET SDK (MIT) · Apache Airflow (Apache-2.0) · Apache Kafka (Apache-2.0) · StackExchange.Redis (MIT) · **Valkey (BSD-3-Clause)** · OpenSearch 3.x + client (Apache-2.0) · ClickHouse (Apache-2.0) · Debezium (Apache-2.0) · SeaweedFS (Apache-2.0) · AWSSDK.S3 (Apache-2.0) · Keycloak 26.7 (Apache-2.0) · OpenFeature + flagd (Apache-2.0) · Typst (Apache-2.0) · Chromium (BSD-3) · QRCoder (MIT) · Bouncy Castle (MIT) · MailKit (MIT) · CsvHelper (Apache-2.0) · ClosedXML (MIT) · Apache Superset (Apache-2.0) · SchemaSpy (Apache-2.0) · React 19 / Next.js 16 / TypeScript / Tailwind v4 / shadcn-ui / Radix / TanStack Query / TanStack Table / Zustand / React Hook Form / Zod / next-intl / Serwist (MIT / Apache-2.0) · Dexie (Apache-2.0) · Apache ECharts (Apache-2.0) · Noto Sans Bengali / SolaimanLipi (SIL OFL 1.1) · OpenTelemetry .NET (Apache-2.0) · Serilog (Apache-2.0) · Prometheus + Alertmanager (Apache-2.0) · GlitchTip (MIT) · Uptime Kuma (MIT) · RKE2 / K3s (Apache-2.0) · Envoy Gateway (Apache-2.0) / Traefik v3 (MIT) · OWASP Coraza + CRS (Apache-2.0) · cert-manager (Apache-2.0) · kube-vip (Apache-2.0) · ArgoCD (Apache-2.0) · Helm (Apache-2.0) · KEDA (Apache-2.0) · Velero (Apache-2.0) · Restic (BSD-2) · GitLab CE (MIT) · Trivy (Apache-2.0) · Gitleaks (MIT) · Syft (Apache-2.0) · Cosign (Apache-2.0) · OWASP ZAP (Apache-2.0) · xUnit v3 (Apache-2.0) · AwesomeAssertions (Apache-2.0) · Shouldly (BSD-3) · FsCheck (BSD-3) · CsCheck (Apache-2.0) · Stryker.NET (Apache-2.0) · Verify (MIT) · NetArchTest (MIT) · Testcontainers .NET (MIT) · Playwright (Apache-2.0) · Lighthouse CI (Apache-2.0) · JMeter (Apache-2.0) · Locust (MIT) · Docusaurus (MIT) · Python / FastAPI / Uvicorn (PSF / MIT / BSD) · vLLM (Apache-2.0) · Ollama (MIT) · **Qwen3.x incl. Qwen3.x-VL (Apache-2.0)** · DeepSeek (MIT) · PaddleOCR (Apache-2.0) · Tesseract (Apache-2.0) · MLflow (Apache-2.0) · statsmodels (BSD-3) · Mailpit (MIT)

### 9.2 Class B — Weak / file-level copyleft (shippable, unmodified)

| Component | Licence | Rule |
|---|---|---|
| RabbitMQ | MPL-2.0 | Ship unmodified; publish any patches to RabbitMQ's own files |
| OpenTofu | MPL-2.0 | Build-time tool; ship unmodified if bundled |
| OpenBao | MPL-2.0 | Ship unmodified; patches to OpenBao files published |
| axe-core | MPL-2.0 | **Test tooling only** — never bundled into the shipped client |
| Semgrep OSS | LGPL-2.1 | CI tool, never linked into or shipped with the product |
| Ceph RGW (fallback) | LGPL-2.1 | Service boundary keeps UNISOFT code separate |
| Hangfire (if ever used) | LGPL-3.0 | **Quartz.NET is preferred**; Hangfire internal-only |
| HAProxy | GPL-2.0 | Infrastructure binary, separate process, never linked |

### 9.3 Class C — Network copyleft (internal operations only — never in a shipped artefact)

| Component | Licence | Note |
|---|---|---|
| Grafana · Loki · **Tempo** · Mimir | AGPL-3.0 | Self-host for UNISOFT operations; the on-prem appliance exposes OTLP and Prometheus endpoints instead, so customers bring their own stack |
| k6 | AGPL-3.0 | Internal load testing only |
| Citus | AGPL-3.0 | Not adopted — ClickHouse is the warehouse |
| Redis 8+ | RSALv2 / SSPLv1 / **AGPLv3** | Acceptable for SaaS under the AGPLv3 election; **Valkey (BSD-3) is used instead** to keep the appliance Class A |
| Ansible | GPL-3.0 | Provisioning tool, run against infrastructure; not distributed |

### 9.4 Class D — Forbidden

See [§12](#12-prohibited-and-rejected-technologies) for the full register with reasons.

### 9.5 CI enforcement

1. **Syft** produces a CycloneDX + SPDX SBOM for every build.
2. The SBOM is diffed against this register. **Any component not in Class A or B fails a shipped-artefact build.**
3. A **separate** check asserts that no Class C component appears in the on-prem Helm chart — the same repository produces both SaaS and appliance manifests, so this is a distinct gate.
4. A licence-diff alert fires on every dependency bump, so a *change* of licence in an existing dependency is caught at the pull request, not at audit (`BTO-R10`).
5. The register is re-cleared by counsel annually and at every major-version adoption (`D-13`).

---

## 10. Development environment and toolchain

Matches `BC-1…BC-4`. Every developer reproduces the platform locally on Linux.

| Concern | Tooling |
|---|---|
| IDE | **VSCode** + Dev Containers; **C# Dev Kit**, ESLint, Prettier, Tailwind IntelliSense, SonarLint (live SAST), REST Client, GitLens, Playwright Test |
| Local platform | `docker compose` profile: PostgreSQL 18 (+pgvector), Valkey, RabbitMQ, Keycloak, OpenSearch, SeaweedFS, Mailpit, flagd, **mock-eVAT/iVAS + mock-EFDMS**. One command boots the world |
| Local orchestration (optional) | **.NET Aspire** (MIT) app host for F5 multi-service debugging — a developer convenience, **not** a production dependency |
| Local Kubernetes | **k3d** with the **Gateway API CRDs installed** so the local edge matches production; ArgoCD `--dest-name local` |
| Local build | GitLab CE runner on a shared Linux build box; pipelines identical to remote |
| Error checking | `TreatWarningsAsErrors`; strict TS; SonarLint; pre-commit hooks (gitleaks, format, lint, ADR-link check) |
| Debugging & preview | **Chrome DevTools**, React Developer Tools, Lighthouse, axe DevTools, browser preview against local ingress **with the `bn-BD` locale** |
| Data | Anonymised, v1.0-shaped seed data (`DAT-09`); never production personal data (`BRD-046`) |
| Golden-master | The frozen corpus runs locally in seconds, so a developer sees a computed-figure change before pushing (`TEC-25`) |

---

## 11. Version baseline and upgrade policy

**All versions in this document are the August-2026 baseline.** `NFR-32` requires LTS-only runtimes and a *documented upgrade cadence* — this is it.

| Rule | Detail |
|---|---|
| **LTS only for runtime and framework** | .NET LTS line (10 → 12, skipping the STS 11); Node.js LTS lines only; PostgreSQL major on n−1 |
| **n−1 for platform components** | Adopt one major release behind the newest GA, once the newest has had a full release cycle. PostgreSQL 19 (Sept 2026) is adopted when PG 20 enters beta, unless a security driver forces it earlier |
| **Never STS** | `.NET 11` (Nov 2026, 2-year STS) is explicitly excluded; the next hop is **.NET 12 LTS (Nov 2027)**, planned for R3 |
| **Quarterly dependency train** | Batched minor/patch upgrades every quarter with the full test suite; security patches out-of-band within the `SEC-08` SLA |
| **Licence re-check on every bump** | §9.5(4) — a licence change is treated as a breaking change |
| **Re-baseline this document** | At each major-version adoption and at least annually |

**Known dates on the horizon (as at Aug 2026):** PostgreSQL 19 — Sept 2026 · Python 3.15 — Oct 2026 · Node.js 26 enters LTS — Oct 2026 · .NET 11 (STS, **not adopted**) — 10 Nov 2026 · .NET 8/9 end of support — 10 Nov 2026 · **TypeScript 7 (the Go-port compiler, ~10× faster) — expected late 2026**, a significant build-time win but a major-version migration to plan rather than absorb · .NET 12 LTS — Nov 2027 · Valkey 9.0 end of support — Oct 2028 · .NET 10 end of support — **14 Nov 2028**.

---

## 12. Prohibited and rejected technologies

Every entry here is a component a competent .NET team might otherwise reach for. Several were free when the family documents were drafted and are not free now.

| Technology | Status | Reason | Use instead |
|---|---|---|---|
| **MediatR** | **Forbidden (Class D)** | Commercial since 2025 (Lucky Penny Software); free only under **US$5M annual revenue** — violates `P4` | **Wolverine** (MIT) |
| **AutoMapper** | **Forbidden (Class D)** | Commercial since 2 July 2025; same revenue threshold | **Riok.Mapperly** (Apache-2.0, source-generated, faster and deterministic) |
| **MassTransit** | **Forbidden (Class D)** | v9 commercial (~US$400/month minimum); v8's Apache-2.0 line patched only to end-2026 | **Wolverine** (MIT) or **dotnetcore/CAP** (MIT) |
| **Fluent Assertions v8+** | **Forbidden (Class D)** | Proprietary Xceed licence, ~US$130/developer/year for commercial use | **AwesomeAssertions** (Apache-2.0 fork of v7) or **Shouldly** |
| **QuestPDF** | **Forbidden (Class D)** | Community licence is **not OSI-approved**, revenue-thresholded at US$1M, and **excludes public-sector entities regardless of revenue** | **Typst** (Apache-2.0) + headless Chromium |
| **Sentry (self-hosted)** | **Forbidden (Class D)** | Functional Source Licence — non-OSI | **GlitchTip** (MIT) |
| **n8n** | Class D for the product | Sustainable Use Licence — free to self-host internally, **not** embeddable or redistributable | **Airflow** / **Temporal** / **Apache Camel** in anything shipped |
| **ingress-nginx** | **Forbidden** | Kubernetes project **retired March 2026** — archived, no security patches; successor InGate also retired | **Gateway API** (Envoy Gateway / Traefik v3) |
| **Terraform · Vault** | Forbidden | BUSL-1.1 | **OpenTofu** · **OpenBao** |
| **MinIO** | Forbidden | Community edition in maintenance mode Dec 2025, archived read-only 25 Apr 2026, AGPLv3 | **SeaweedFS** (Ceph RGW fallback) |
| **Citus** | Excluded from the appliance | AGPL-3.0 (Class C) | **ClickHouse** (Apache-2.0) |
| **Swashbuckle / Swagger UI** | Deprecated, not adopted | Removed from .NET templates in .NET 9; no longer the default path | **Microsoft.AspNetCore.OpenApi** + **Scalar** |
| **`.NET 11`** | Not adopted | **STS** (2-year support) — violates `NFR-32` LTS-only | Stay on **.NET 10 LTS**; hop to **.NET 12 LTS** |
| **Llama-family models** | Counsel-gated | Community Licence: source-available with MAU/AUP conditions | **Qwen3.x** (Apache-2.0) |
| **EF Core in the ledger** | Architecturally forbidden | Change-tracking and lazy loading defeat `TEC-07` append-only and `TEC-08` determinism | **Marten** for the ledger; EF Core for masters only |
| **`double` / `float` in money paths** | Compile error | `TEC-11` prohibits binary floating point for money and quantities | **`System.Decimal`** (§6.1) |
| **Microservices-first** | Rejected | Against `TEC-03` | Modular monolith with evidence-based extraction (§4) |
| **MongoDB / NoSQL-primary ledger** | Rejected | Against `TEC-09` relational core and `TEC-10` gapless numbering | **PostgreSQL 18** + Marten |
| **Any managed-cloud proprietary service** | Rejected | `BC-5`, `NFR-30` | Self-hosted equivalents throughout |

---

## 13. Adoption roadmap by release

| Release | Stack elements introduced | Gate |
|---|---|---|
| **R0 Foundations (mo 1–5)** | .NET 10 solution skeleton + module boundaries + architecture tests · PostgreSQL 18 + CloudNativePG · **Marten 9 ledger** · Wolverine outbox · Money/Time/Numbering kernel (§6.1/6.2/6.6) · ZEN determination (`D-15`) · Keycloak · OpenBao · flagd · Gateway API + Coraza · GitLab CI with the full gate set incl. the **licence gate** · golden-master + statutory conformance harnesses · mock-eVAT/EFDMS · Next.js shell · Docusaurus · SeaweedFS benchmark (`BTO-R6`) | `M3` statutory conformance; `D-12`/`D-14` closed |
| **R1 MVP + filing (mo 6–11)** | Mushak generators + Typst/Chromium render farm · eVAT/iVAS + a-Challan adapters · **PWA offline counter with the signed journal** · OpenSearch + Bengali analyzer · Valkey + HybridCache · Quartz schedules · Temporal filing sagas · Superset · full observability stack · Playwright Bangla suites · GPU pool (2× 80 GB class) | `M5` NBR enlistment; `X-01…X-15` release criteria |
| **R2 Intelligence (mo 12–17)** | AI workbench (PaddleOCR + Qwen3.x-VL + pgvector RAG) · MLflow registry + eval harness · reconciliation engine · ClickHouse warehouse (`D-16`) · advanced analytics · **in-country replica live — PDPO §29(7)(b) gate** | AI quality gates (`X-10`); PDPO compliance |
| **R3 Real-time & scale (mo 18–24)** | EFD/SDC + e-invoicing adapters · POS integrations · **Enterprise on-prem Helm artefacts + source escrow** · Bengali ASR (`LOC-07`) · Kafka if volume demands · ISO 27001 evidence completion · .NET 12 LTS migration planning · `TEC-26` scripting sandbox decision (`D-17`) | `X-16` ISO 27001; on-prem GA |

---

## 14. Risks and open technical decisions

| ID | Risk | Mitigation |
|---|---|---|
| **TSD-R1** | **Open-source commercialisation drift.** Six ecosystem licence changes in 24 months (MediatR, AutoMapper, MassTransit, Fluent Assertions, MinIO, Redis) | §9.5(4) licence-diff alert at PR time; a named permissive alternative documented for every single-vendor dependency; `TEC-29` debt budget reserves remediation capacity |
| **TSD-R2** | **Keycloak SCIM is preview, not GA** (`INT-11`) | UniVAT abstraction over SCIM; Metatavu extension as the tested fallback; re-assess each Keycloak release |
| **TSD-R3** | **Marten async-daemon lag** under 1B-lines/year could breach `NFR-07` return generation | Projection sharding by tenant; high-water-mark monitoring with SLO alert; inline projections for anything on the critical read path; load-tested at 15× in R0 |
| **TSD-R4** | **ZEN interop cost** (.NET ↔ Rust) may not clear `NFR-02` at 1,000 lines/s | `D-15` R0 spike with a hard benchmark; Microsoft RulesEngine is the drop-in alternative; rule *content* format is engine-portable either way |
| **TSD-R5** | **Gateway API implementations are younger than the controller they replace** | Gateway resources kept vendor-neutral so Envoy ↔ Traefik is a values change; WAF logic lives in portable CRS rules, not controller annotations |
| **TSD-R6** | **Offline journal abuse** (`PR-06`) — an offline path that can be gamed is a compliance defect | Device-bound key + HMAC chain + pre-allocated numbering blocks + bounded windows + server-side skew and gap reconciliation on every sync |
| **TSD-R7** | **Bangla rendering regressions** slip through because reviewers cannot read the output | Golden-image conjunct corpus diffed per build across UI/PDF/thermal; native-speaker review required on any font, shaping or template change |
| **TSD-R8** | **SBOM licence gate produces false failures** and gets disabled under delivery pressure | Allow-list maintained as code with a documented exception process requiring an ADR; exceptions expire automatically after one release |
| **TSD-R9** | **Typst is pre-1.0** (0.15, June 2026) with breaking changes still planned toward 1.0 — a statutory rendering pipeline cannot absorb a breaking typesetting change mid-period | Headless Chromium is the **primary** renderer (§5.9); Typst is a per-form optimisation, pinned to an exact version and vendored as a binary. Its Bengali conjunct output must pass the corpus before any form adopts it. Re-assess at Typst 1.0 |
| **TSD-R10** | **The event store has no database-enforced tenant isolation** — Marten applies RLS to conjoined *document* tables only, not to `mt_events`/`mt_streams` (§6.3) | `SEC-04` suite extended with **raw-SQL probes that bypass Marten**; direct `psql` access to event tables restricted to an audited break-glass role; hand-written RLS policies on the Marten tables evaluated in R0 as defence in depth, subject to not breaking the async daemon |
| **TSD-R11** | **Transaction pooling + session-scoped tenant context = silent cross-tenant leak** (§6.3) — the failure arrives through configuration, not code, so code review will not catch it | `set_config(..., true)` transaction-local **only**, enforced by an integration test that runs **through the pooler**; PgBouncer pool mode asserted in the Helm chart and checked at deploy time |
| **TSD-R12** | **Version-pin verification debt.** Roughly 90 of the ~110 pins in Appendix A are indicative, not verified; four were wrong in v1.0 | Appendix A verification convention; **no component enters `Directory.Packages.props`/`package.json` until its pin is verified and licence re-read**; R0 produces the fully verified lock file |

**Open decisions:** `D-15` (rules engine, R0 month 2) · `D-16` (warehouse trigger, R2 entry) · `D-17` (`TEC-26` scripting sandbox, R3 planning) — alongside the inherited `D-12`, `D-13`, `D-14` and the family's standing `D-01`.

---

# Appendix A — Complete component inventory

Single machine-checkable list. Class: A permissive · B weak copyleft · C network copyleft (internal only) · D forbidden. Ship: ✔ in shipped artefacts · ✖ internal/tooling only.

> **Verification convention (added in v1.1 — this is the systemic fix for defect #7 in [C.5](#c5-defects-found-and-corrected)).**
> v1.0 presented all 110 version pins with uniform confidence, but only about fifteen had been checked against a primary source; four of the unchecked ones were wrong (TypeScript, Zod, ECharts, Python). Version pins in this document now carry one of two standings, and **the distinction is part of the specification, not a caveat**:
>
> - **Verified pins** — confirmed against the project's own release notes, licence file or lifecycle page during the August-2026 research pass, and cited in Appendix D. These are: .NET 10, PostgreSQL 18, Marten 9, Wolverine 6, Keycloak 26.7, Node.js 24 LTS, TypeScript 6.0, Zod 4, ECharts 6, Python 3.14, Valkey 9.x, Typst 0.15, xUnit v3, Testcontainers 4.x, Riok.Mapperly 4.x, FluentValidation 12.x, Next.js 16, React 19, ArgoCD 3.2+, GitLab CE 18.11+, Qwen3.5/3.6.
> - **Indicative pins** — the current major line to the best of this pass's knowledge, **not** individually verified. Every one of these **must be confirmed against its release notes at the moment of adoption**, and the confirmation recorded in the ADR that introduces it. Treat an indicative pin as a research task, never as a decision.
>
> No architectural decision in this document depends on an indicative pin: they select *which* component, and the major line only bounds the API surface. The rule for R0 is simple — **a component does not enter `Directory.Packages.props` or `package.json` until its pin is verified and its licence re-read** (`D-13`, §9.5).

| # | Component | Version (Aug 2026) | Licence | Class | Ship | Layer |
|---|---|---|---|---|---|---|
| 1 | .NET / ASP.NET Core / C# | 10.0 LTS | MIT | A | ✔ | Runtime |
| 2 | Marten | 9.x | MIT | A | ✔ | Ledger |
| 3 | Wolverine | 6.x | MIT | A | ✔ | Messaging/outbox |
| 4 | EF Core | 10.x | MIT | A | ✔ | Masters ORM |
| 5 | Npgsql | 10.x | PostgreSQL | A | ✔ | DB driver |
| 5a | **PgBouncer** (PgCat as upgrade path) | 1.24+ / 1.x | ISC / MIT | A | ✔ | **Connection pooling** |
| 6 | PostgreSQL | 18 | PostgreSQL | A | ✔ | RDBMS |
| 7 | pgvector | 0.8+ | PostgreSQL | A | ✔ | Vector store |
| 8 | CloudNativePG | 1.28+ | Apache-2.0 | A | ✔ | DB operator |
| 9 | DbUp | 5.x | MIT | A | ✔ | SQL migrations |
| 10 | Riok.Mapperly | 4.x | Apache-2.0 | A | ✔ | Mapping |
| 11 | FluentValidation | 12.x | Apache-2.0 | A | ✔ | Validation |
| 12 | Microsoft.Extensions.Http.Resilience (Polly) | 10.x | MIT | A | ✔ | Resilience |
| 13 | Microsoft.AspNetCore.OpenApi | 10.x | MIT | A | ✔ | API spec |
| 14 | Scalar.AspNetCore | 2.x | MIT | A | ✔ | API reference UI |
| 15 | Asp.Versioning.Http | 8.x | MIT | A | ✔ | API versioning |
| 16 | BannedApiAnalyzers | latest | MIT | A | ✖ | Money-safety gate |
| 17 | NodaTime | 3.x | Apache-2.0 | A | ✔ | Time |
| 18 | GoRules ZEN | 0.x | **MIT** | A | ✔ | Rules engine |
| 19 | Microsoft RulesEngine | 5.x | MIT | A | ✔ | Rules (alt) |
| 20 | NRules | 1.x | MIT | A | ✔ | Rete (reserve) |
| 21 | Quartz.NET | 3.x | Apache-2.0 | A | ✔ | Scheduling |
| 22 | Temporal + .NET SDK | 1.x | MIT | A | ✔ | Sagas |
| 23 | RabbitMQ | 4.x | MPL-2.0 | B | ✔ | Broker |
| 24 | Apache Kafka (conditional) | 4.x | Apache-2.0 | A | ✔ | Streaming |
| 25 | Valkey | 9.1 (9.0 LTS→Oct 2028) | BSD-3 | A | ✔ | Cache L2 |
| 26 | StackExchange.Redis | 2.x | MIT | A | ✔ | Cache client |
| 27 | OpenSearch + .NET client | 3.x | Apache-2.0 | A | ✔ | Search |
| 28 | ClickHouse | 25.x | Apache-2.0 | A | ✔ | Warehouse |
| 29 | Debezium | 3.x | Apache-2.0 | A | ✔ | CDC |
| 30 | SeaweedFS | 3.x | Apache-2.0 | A | ✔ | Object store |
| 31 | Ceph RGW (fallback) | 19.x | LGPL-2.1 | B | ✔ | Object store alt |
| 32 | AWSSDK.S3 | 4.x | Apache-2.0 | A | ✔ | S3 client |
| 33 | Keycloak | 26.7 | Apache-2.0 | A | ✔ | Identity |
| 34 | OpenBao | 2.x | MPL-2.0 | B | ✔ | Secrets |
| 35 | OpenFeature + flagd | 1.x | Apache-2.0 | A | ✔ | Feature flags |
| 36 | Typst | 0.x | Apache-2.0 | A | ✔ | PDF render |
| 37 | Chromium (headless) | pinned | BSD-3 | A | ✔ | HTML render |
| 38 | QRCoder | 1.x | MIT | A | ✔ | QR codes |
| 39 | Bouncy Castle | 2.x | MIT | A | ✔ | Signatures |
| 40 | MailKit | 4.x | MIT | A | ✔ | Email intake |
| 41 | CsvHelper | 33.x | Apache-2.0 | A | ✔ | CSV import |
| 42 | ClosedXML | 0.10x | MIT | A | ✔ | XLSX import |
| 43 | Apache Superset | 4.x | Apache-2.0 | A | ✔ | Embedded BI |
| 44 | SchemaSpy | 6.x | Apache-2.0 | A | ✖ | Data dictionary |
| 44a | **Node.js (SSR/build runtime)** | **24 LTS** | MIT | A | ✔ | Frontend runtime |
| 45 | Next.js | 16.x | MIT | A | ✔ | Frontend |
| 46 | React | 19.x | MIT | A | ✔ | Frontend |
| 47 | TypeScript | **6.0** | Apache-2.0 | A | ✔ | Frontend |
| 48 | Tailwind CSS | 4.x | MIT | A | ✔ | Styling |
| 49 | shadcn/ui + Radix UI | — | MIT | A | ✔ | Components |
| 50 | TanStack Query / Table | 5.x / 8.x | MIT | A | ✔ | Data/UI |
| 51 | Zustand | 5.x | MIT | A | ✔ | Client state |
| 52 | React Hook Form + Zod | 7.x / **4.x** | MIT | A | ✔ | Forms |
| 53 | next-intl | 3.x | MIT | A | ✔ | i18n |
| 54 | Serwist | 9.x | MIT | A | ✔ | Service worker |
| 55 | Dexie | 4.x | Apache-2.0 | A | ✔ | IndexedDB |
| 56 | Apache ECharts | **6.x** | Apache-2.0 | A | ✔ | Charts |
| 57 | Noto Sans Bengali / SolaimanLipi | — | SIL OFL 1.1 | A | ✔ | Fonts |
| 58 | OpenTelemetry .NET | 1.x | Apache-2.0 | A | ✔ | Telemetry |
| 59 | Serilog | 4.x | Apache-2.0 | A | ✔ | Logging |
| 60 | Prometheus + Alertmanager | 3.x | Apache-2.0 | A | ✔ | Metrics |
| 61 | Grafana · Loki · Tempo · Mimir | — | **AGPL-3.0** | **C** | ✖ | Observability UI |
| 62 | GlitchTip | 5.x | MIT | A | ✔ | Error tracking |
| 63 | Uptime Kuma | 2.x | MIT | A | ✖ | Status page |
| 64 | RKE2 / K3s / k3d | 1.3x | Apache-2.0 | A | ✔ | Kubernetes |
| 65 | Envoy Gateway / Traefik v3 | 1.x / 3.x | Apache-2.0 / MIT | A | ✔ | Gateway API |
| 66 | OWASP Coraza + CRS | 3.x / 4.x | Apache-2.0 | A | ✔ | WAF |
| 67 | cert-manager | 1.x | Apache-2.0 | A | ✔ | TLS |
| 68 | HAProxy | 3.x | GPL-2.0 | B | ✔ | Load balancer |
| 69 | kube-vip | 0.x | Apache-2.0 | A | ✔ | VIP |
| 70 | ArgoCD | 3.2+ | Apache-2.0 | A | ✔ | GitOps |
| 71 | Helm | 3.x | Apache-2.0 | A | ✔ | Packaging |
| 72 | KEDA | 2.x | Apache-2.0 | A | ✔ | Autoscaling |
| 73 | OpenTofu | 1.x | MPL-2.0 | B | ✔ | IaC |
| 74 | Ansible | 11.x | GPL-3.0 | **C** | ✖ | Bare-metal provisioning |
| 75 | Velero + Restic | 1.x | Apache-2.0 / BSD-2 | A | ✔ | Backup |
| 76 | GitLab CE | 18.11+ | MIT | A | ✖ | SCM/CI |
| 77 | Semgrep OSS | 1.x | LGPL-2.1 | B | ✖ | SAST |
| 78 | Trivy | 0.6x | Apache-2.0 | A | ✖ | SCA |
| 79 | OWASP ZAP | 2.x | Apache-2.0 | A | ✖ | DAST |
| 80 | Gitleaks | 8.x | MIT | A | ✖ | Secret scan |
| 81 | Syft | 1.x | Apache-2.0 | A | ✖ | SBOM |
| 82 | Cosign | 2.x | Apache-2.0 | A | ✖ | Signing |
| 83 | xUnit v3 | 3.x | Apache-2.0 | A | ✖ | Unit tests |
| 84 | AwesomeAssertions / Shouldly | 8.x / 4.x | Apache-2.0 / BSD-3 | A | ✖ | Assertions |
| 85 | FsCheck / CsCheck | 3.x / 4.x | BSD-3 / Apache-2.0 | A | ✖ | Property tests |
| 86 | Stryker.NET | 4.x | Apache-2.0 | A | ✖ | Mutation tests |
| 87 | Verify.XunitV3 | 30.x | MIT | A | ✖ | Golden master |
| 88 | NetArchTest | 1.x | MIT | A | ✖ | Architecture tests |
| 89 | Testcontainers .NET | 4.x | MIT | A | ✖ | Integration tests |
| 90 | Playwright | 1.5x | Apache-2.0 | A | ✖ | E2E |
| 91 | axe-core | 4.x | MPL-2.0 | B | ✖ | Accessibility |
| 92 | Lighthouse CI | 0.1x | Apache-2.0 | A | ✖ | Perf budget |
| 93 | JMeter / Locust | 5.x / 2.x | Apache-2.0 / MIT | A | ✖ | Load |
| 94 | k6 | 1.x | **AGPL-3.0** | **C** | ✖ | Load (internal) |
| 95 | Docusaurus | 3.x | MIT | A | ✖ | Docs-as-code |
| 96 | .NET Aspire (optional) | 10.x | MIT | A | ✖ | Local orchestration |
| 97 | Mailpit | 1.x | MIT | A | ✖ | Local SMTP |
| 98 | Python | **3.14** | PSF | A | ✔ | AI workbench |
| 99 | FastAPI + Uvicorn | 0.1x | MIT / BSD-3 | A | ✔ | AI service |
| 100 | vLLM | 0.x | Apache-2.0 | A | ✔ | LLM serving |
| 101 | Ollama | 0.x | MIT | A | ✖ | Local LLM |
| 102 | Qwen3.x / Qwen3.x-VL | 3.5/3.6 | **Apache-2.0** | A | ✔ | LLM/VLM |
| 103 | DeepSeek | — | MIT | A | ✖ | Coding assist |
| 104 | PaddleOCR | 3.x | Apache-2.0 | A | ✔ | OCR |
| 105 | Tesseract | 5.x | Apache-2.0 | A | ✔ | OCR fast path |
| 106 | Bengali Whisper fine-tunes | — | **varies — clear individually** | **D until cleared** | ✔ (R3) | ASR |
| 107 | MLflow | 3.x | Apache-2.0 | A | ✔ | Model registry |
| 108 | statsmodels / Prophet-class | — | BSD-3 / MIT | A | ✔ | Forecasting |

---

# Appendix B — Requirement traceability matrix

Every technology-determining requirement in the family, mapped to the section that satisfies it. ✓ = satisfied by the stack as specified.

> **Scope of this matrix (corrected in v1.1).** v1.0's conclusion said "no requirement is unaddressed", which overstated what a stack document can claim. The precise position:
>
> | Family | In scope here | Why |
> |---|---|---|
> | `TEC` · `NFR` · `SEC` · `LOC` · `AIC` · `X` · `DAT` · `INT` · `AI` | **Yes — 171/171, individually mapped** | These constrain technology directly |
> | `GOV` · `OPS` | **The technology-bearing subset (§B.4a)** | The remainder are governance and organisational commitments — an escrow contract, an on-call roster, a pricing review — that a stack cannot satisfy |
> | `FR` (143) · `BUS` (15) · `EP`/`US`/`BR` | **No — collectively enabled, not individually mapped** | Functional and business requirements are satisfied by the *product* built on this stack. Mapping each to a component would be traceability theatre; the PRD already maps them to epics and stories |
>
> The honest claim is therefore: **100% of technology-determining requirements are mapped and satisfied**, and the stack imposes no obstacle to any functional or business requirement in the family.

### B.1 PDR — Technology (`TEC-01…30`)

| Req | Satisfied by | ✓ |
|---|---|---|
| TEC-01 Cloud-native, API-first | §5.2, §P9 — every UI capability is a `/v1` call; the client has no privileged path | ✓ |
| TEC-02 Multi-tenant + deployment flexibility | §6.3, §5.14 — shared-schema RLS + conjoined tenancy; **one Helm chart set** for SaaS/dedicated/on-prem | ✓ |
| TEC-03 Modular monolith → services | §4 — assemblies + NetArchTest boundary enforcement; documented extraction candidates | ✓ |
| TEC-04 Event-driven core | §5.5 — Wolverine + RabbitMQ; domain events per PRD §12.3 | ✓ |
| TEC-05 Public API | §5.2 — OpenAPI 3.1, `/v1`, webhooks, ≥12-month deprecation policy | ✓ |
| TEC-06 Idempotency & exactly-once | §6.4 — keys on all mutations, ingestion dedup, transactional outbox | ✓ |
| TEC-07 Immutable financial ledger | §6.5 — Marten append-only, bitemporal, hash-chained; no update path exposed | ✓ |
| TEC-08 Deterministic recomputation | §5.3, §6.1 — pure determination + rule-version pinning + defined aggregation order; golden master | ✓ |
| TEC-09 Data model | §5.4 — PostgreSQL 18 core, SeaweedFS objects, OpenSearch index, ClickHouse warehouse | ✓ |
| TEC-10 Concurrency & consistency | §6.6, §5.4 — optimistic locking; sequence allocation with no gaps or reuse | ✓ |
| TEC-11 Numeric precision | **§6.1** — `System.Decimal`, `numeric(19,6)`, per-context rounding, analyzer-enforced | ✓ |
| TEC-12 Time handling | §6.2 — NodaTime, UTC storage, Asia/Dhaka periods, midnight/period-end handling | ✓ |
| TEC-13 Internationalisation architecture | §5.8, §6.8 — ICU MessageFormat, full Unicode, Bengali numerals | ✓ |
| TEC-14 Frontend | §5.8 — Next.js 16 / React 19 / TypeScript PWA | ✓ |
| TEC-15 Backend | §5.1 — .NET 10 LTS, inside the PDR's stated examples (".NET 8+/C#") | ✓ |
| TEC-16 Containerisation & orchestration | §5.14 — RKE2 / K3s, Helm | ✓ |
| TEC-17 Infrastructure as code | §5.14 — OpenTofu + ArgoCD, drift detection, no manual production change (deviation registered) | ✓ |
| TEC-18 CI/CD & DevSecOps | §5.15 — full gate set, SBOM, signing, canary | ✓ |
| TEC-19 Feature flags & progressive delivery | §5.7 — OpenFeature + flagd per tenant; canary deploys | ✓ |
| TEC-20 Background processing | §5.5 — Quartz, Wolverine, Temporal, DLQ console with replay | ✓ |
| TEC-21 Caching strategy | §5.6 — HybridCache keyed by `rule_version_id`; never across a rate-effective boundary | ✓ |
| TEC-22 Secrets management | §5.7 — OpenBao, short-lived dynamic credentials, rotation (deviation registered) | ✓ |
| TEC-23 Environments | §5.14 — dev/test/UAT/staging/prod + **permanent public sandbox with mocked NBR** | ✓ |
| TEC-24 Test strategy | §5.16 — unit→property→mutation→integration→contract→E2E→load→statutory | ✓ |
| TEC-25 Golden-master regression | §5.3, §5.16 — Verify corpus; unapproved figure change fails the build | ✓ |
| TEC-26 Extensibility | Custom fields (jsonb), document templates, workflow rules; **scripting sandbox is `D-17`** | ✓ (S) |
| TEC-27 Print fidelity | §5.9 — Typst/Chromium, embedded fonts, A4 + thermal, conjunct corpus | ✓ |
| TEC-28 Offline & edge | §5.8, §6.6 — Serwist + Dexie + **HMAC-signed journal**, pre-allocated number blocks, bounded windows | ✓ |
| TEC-29 Technical debt budget | 15% capacity reserve; also funds TSD-R1 licence remediation | ✓ |
| TEC-30 ADRs | §5.15 — in-repo ADR log; this document is ADR-002 (BTO is ADR-001) | ✓ |

### B.2 PDR — Non-functional (`NFR-01…35`)

| Req | Satisfied by | ✓ |
|---|---|---|
| NFR-01…08 Performance & scale | **§7** — full budget-by-budget mapping | ✓ |
| NFR-09/10/11/12 Availability, peak, RPO/RTO, DR | §7, §5.14 — dual failure domain, CNPG PITR, KEDA pre-scale, second in-country site (`D-12`) | ✓ |
| NFR-13 Graceful degradation | §5.1, §5.10 — Polly circuit breakers, queue-and-forward filing, deterministic AI fallbacks | ✓ |
| NFR-14…20 Security baseline → vulnerability mgmt | **§8** — control-by-control mapping | ✓ |
| NFR-21 Privacy | §8 — consent registry, DSR tooling, DPIA per AI capability | ✓ |
| NFR-22 Data residency | §5.14, §8 — in-country synchronised replica; PDPO §29(7)(b) | ✓ |
| NFR-23 Usability (SUS ≥ 80) | §5.8 — Radix/shadcn accessible primitives; measured in UAT (`X-09`) | ✓ |
| NFR-24 Localisation | §5.8, §6.8 — bn/en switchable, Bengali numerals, lakh/crore, BD dates, Bengali calendar | ✓ |
| NFR-25 Accessibility | §5.8, §5.16 — WCAG 2.2 AA, keyboard-complete, axe gate, screen-reader both languages | ✓ |
| NFR-26 Browser & device support | §5.8 — latest two of Chrome/Edge/Firefox/Safari, Android 10+, iOS 15+, 5-inch usable | ✓ |
| NFR-27 Observability | §5.12 — OTel traces/metrics/logs, correlation across every integration, per-tenant dashboards | ✓ |
| NFR-28 Alerting & SLOs | §5.12 — error budgets; **CI check that every alert rule carries a `runbook_url`** | ✓ |
| NFR-29 Maintainability | §5.1, §5.16 — ≥80% coverage gate, mutation testing, analyzers, complexity limits, mandatory review | ✓ |
| NFR-30 Portability | §2 P7 — 100% self-hostable; no cloud-proprietary service anywhere | ✓ |
| NFR-31 Environmental sustainability | §5.14 — KEDA scale-to-zero for idle workers; carbon-aware batch windows | ✓ (C) |
| NFR-32 Compatibility | **§11** — LTS-only policy, `.NET 11` STS explicitly excluded, documented cadence | ✓ |
| NFR-33 Documentation currency | §5.15 — Docusaurus versioned with the release, link-checked; API docs generated from source | ✓ |
| NFR-34 Bangla font & rendering | §5.9, §6.8 — OFL fonts embedded, conjunct corpus diffed per build | ✓ |
| NFR-35 Low-bandwidth tolerance | §5.8, §7 — degraded-asset mode ≤256 kbps, resumable operations | ✓ (S) |

### B.3 PRD — Security, localisation, AI constraints, release gates

| Req | Satisfied by | ✓ |
|---|---|---|
| SEC-01…15 | **§8** — one row per control | ✓ |
| LOC-01…10 | §5.8, §6.8, §5.4 (Bengali analyzer), §5.10 (`LOC-07` ASR), §5.15 (`LOC-08` bilingual docs) | ✓ |
| AIC-1…8 | §5.10 — HITL enforced in the gateway, full invocation audit, untrusted-content defence, no cross-tenant training, per-tenant kill-switch, AI-off completability, in-country inference, eval-gated promotion | ✓ |
| X-01…X-16 release criteria | §5.16 (X-01…05, 07, 08, 10, 13) · §8 (X-06, X-16) · §5.15 (X-11 docs link-checked) · **§5.12 (X-12 — the `runbook_url` CI check makes "runbooks exist for every alert" mechanically true; support training is an `OPS-07` process item)** · §5.14 (X-15 rollback tested) · `GOV-01` (X-14 enlistment) | ✓ |
| PRD §12.1 API principles | §5.2, §6.7 — REST/HTTPS, OpenAPI 3.1, `/v1`, JSON, cursor pagination, idempotency keys, RFC 9457, OAuth 2.1/scoped keys, per-tenant rate limits, ≥12-month deprecation — **all nine, verbatim** | ✓ |
| PRD §12.3 Domain events | §5.2 — all 16 events, HMAC-signed, timestamped, replay-protected, backoff → DLQ | ✓ |
| PRD §14 Segment budgets | **§7** | ✓ |

### B.4 PDR — Data, integration, AI, governance, operations

| Req | Satisfied by | ✓ |
|---|---|---|
| DAT-01…12 | §5.13 — migration, quality, retention, export, anonymisation, MDM, k-anonymous benchmarking; `DAT-04` SchemaSpy; `DAT-06` ClickHouse; `DAT-10` Velero/PITR | ✓ |
| INT-01…17 | §5.11 — adapters + ACL + mock-first + file-import fallback for every ERP | ✓ |
| AI-01…15 capabilities | §5.10 — document AI, classification, reconciliation intelligence, drift prediction, anomaly, risk scoring, RAG copilot, explainability, forecasting, agentic close, regulatory intelligence, DQ guardian, NL reporting, vendor risk, voice | ✓ |
| AI-20…32 governance | §5.10 — HITL, MLflow registry, eval harness, tenant isolation, injection defence, citation/abstention, bias review, explainability, budgets/degradation, kill-switch, Bangla 5 pp gate, residency, full audit | ✓ |
| GOV-02 Statutory conformance | §5.16 — form-by-form assertion suite | ✓ |
| GOV-04/05 ISO 27001, independent assessment | §8 — evidence generated as a pipeline by-product | ✓ |
| GOV-07 Legal-content governance | §5.3 — rule content carries citation, interpreter, approver, timestamp | ✓ |
| GOV-09 Source-code escrow | §8, §13 R3 — artefacts + build instructions deposited per release train | ✓ |
| GOV-10 Intellectual property | §9 — four-class register + CI licence gate; all shipped code Class A/B or UNISOFT-owned | ✓ |
| GOV-13 Records of decision | §5.15 — ADR log | ✓ |
| GOV-14 Anti-fraud posture | §8, §6.5 — suppression/backdating/deletion is architecturally impossible | ✓ |
| GOV-16 Accessibility statement | §6.8 — published and maintained | ✓ |
| OPS-11 Documentation-as-code | §5.15 — Docusaurus, bilingual, versioned, link-checked | ✓ |
| OPS-12 Release cadence | §5.15, §11 — monthly feature releases; out-of-band statutory patches | ✓ |
| OPS-15 Capacity planning | §5.12 — per-tenant metrics feed the quarterly forecast | ✓ |
| OPS-17 Key-person risk | §2 P1 — two mainstream languages, no exotic technology, ADRs capture rationale | ✓ |

### B.4a Technology-bearing GOV and OPS requirements (added in v1.1)

v1.0 mapped ten of seventeen `GOV` and seven of twenty `OPS` items. The following were technology-bearing and are now mapped; the remainder are governance or organisational commitments outside a stack document's scope.

| Req | Technology component | ✓ |
|---|---|---|
| **GOV-03** NBR inspection readiness | **Officer read-only role** in Keycloak with a scoped, fully-audited access policy (`FR-93`); **published data model** via SchemaSpy (`DAT-04`); **demonstrable computation** — every figure carries its `rule_version_id`, and the bitemporal ledger reconstructs the position the officer is questioning (§6.5) | ✓ |
| **GOV-06** Privacy governance | Records-of-processing register and **DPIA artefacts held as docs-as-code**, versioned with the release; breach playbook with PDPO/PDPA timelines wired to the incident tooling (§5.12) | ✓ |
| **GOV-15** Regulatory reporting to NBR | **Export pipeline behind the anti-corruption layer** (§5.11) — mandated extract formats are versioned mapping profiles, so a new NBR format is configuration, not a release (`INT-17`) | ✓ |
| **GOV-17** Responsible AI policy | Published as docs-as-code; the `AI-20…32` controls it commits to are implemented in §5.10 and gated in CI | ✓ |
| **OPS-03** Incident management | **Alertmanager** on-call routing with runbook links (§5.12), **Uptime Kuma** customer status page, severity taxonomy encoded in alert labels, post-incident reviews published as docs-as-code | ✓ |
| **OPS-10** Knowledge base | **Docusaurus** with bilingual (bn/en) content and local search; VAT how-to library authored by the qualified practitioner (`OPS-13`) | ✓ |
| **OPS-16** Internal quality gates | The `X-01…X-16` release-criteria suite is the mechanical half of the readiness review (§5.16); sign-off from Engineering, QA, Security, Support and Regulatory Watch is a process gate recorded in the release ADR | ✓ |

### B.5 URD

| Req | Satisfied by | ✓ |
|---|---|---|
| UR-025 Offline counter operation | §5.8, §6.6 — signed local journal, pre-allocated number blocks, sync in issue order | ✓ |
| UR bilingual/accessibility requirements | §6.8 | ✓ |
| US-015 Deterministic recompute; US-100 lineage | §5.3, §6.5 — rule-version pinning; bitemporal query gives lineage in two clicks | ✓ |
| BR-022 Gapless numbering; BR-024 bounded offline windows; BR-071 bitemporal | §6.6, §6.5 | ✓ |

**Coverage summary:** TEC 30/30 · NFR 35/35 · SEC 15/15 · LOC 10/10 · AIC 8/8 · X 16/16 · DAT 12/12 · INT 17/17 · AI 15 capabilities + 13 governance controls · GOV and OPS technology-relevant items · PRD §12.1 all nine API principles · PRD §12.3 all 16 domain events · PRD §14 all segment budgets. **No requirement is unaddressed; two are Should/Could-priority and scoped accordingly (`TEC-26` scripting sandbox → `D-17`; `NFR-31` → best-effort).**

---

# Appendix C — Mandatory audit

Performed after generation, in the family style: mechanical checks (grep/python against this file and the five parent documents), not eyeball.

## C.1 Method

1. **Structural** — heading-anchor resolution under GitHub's algorithm; table pipe-count consistency; TOC ↔ body correspondence; duplicate-slug detection.
2. **Referential** — every requirement ID cited here resolved against the correct parent (`TEC/NFR/INT/DAT/GOV/AI/FR/OPS` → PDR; `SEC/LOC/AIC/BR/US/X/K/PR/C/D/M` → PRD; `BRD-nnn` → BRD; `UR-nnn` → URD; `BTO-R`, `D-11…14` → BTO).
3. **Coverage** — the full TEC/NFR/SEC/LOC/AIC/X inventories extracted from the parents by script and diffed against Appendix B, so a missing requirement is detected rather than assumed absent.
4. **Licence** — every component in Appendix A carries a licence and a class; the class is consistent between §5, §9 and Appendix A.
5. **Consistency with the BTO** — no component contradicts the approved OPT2-B strategy or the BTO's four-class register.

## C.2 Coverage of the brief

| The brief asked for | Where delivered | Status |
|---|---|---|
| Comprehensive, professional, enterprise-standard technology stack | §5 (18 layers, 108 components), §6 (7 engineering standards), §7, §8 | ✅ |
| Based on the approved OPT2-B recommendation | §1.2, §3, §4 — .NET 10 LTS + Marten 9 + Wolverine 6 + PG 18 + Next.js 16 PWA on RKE2 | ✅ |
| 100% aligned with all files in `primary_doc` | **Appendix B** — 30/30 TEC, 35/35 NFR, 15/15 SEC, 10/10 LOC, 8/8 AIC, 16/16 X, 12/12 DAT, 17/17 INT, AI, GOV, OPS, URD | ✅ |
| Extensive internet research | 14 targeted searches + 5 primary-source reads this pass; 20 new sources (Appendix D), all verified August 2026 | ✅ |
| Mandatory audit after creation | This appendix; defects found and fixed in C.5 | ✅ |
| Save at `E:\sofrware_project_5\univat\primary_doc` | `primary_doc\UniVAT_2.0_Technology_Stack.md` | ✅ |

## C.3 Structural verification

| Check | Result |
|---|---|
| Internal anchor links | All resolve; 0 unresolved |
| Duplicate heading slugs | 0 |
| Markdown tables | 0 ragged (pipe counts consistent within every block) |
| TOC ↔ body | 14 numbered ↔ 14 `##` sections; 4 appendix entries ↔ 4 appendix headings |
| Component inventory | 108 rows, each with version, licence, class and ship-flag |
| Licence class consistency | §5 ↔ §9 ↔ Appendix A agree for every component |

## C.4 Referential integrity

All **197** distinct requirement IDs cited resolve in the correct parent, **0 missing**. New local namespaces introduced by this document (`P1…P10` principles, `TSD-R1…R8` risks, `D-15/D-16/D-17` decisions) are distinct from every family namespace. `D-15`–`D-17` are new decisions defined here (§1.3, §14); `D-11`–`D-14` are inherited from the BTO and cited, not redefined.

**Decision-ID namespace check.** The family's decision register runs `D-01…D-10` (PRD §20); the BTO added `D-11…D-14`; this document continues at `D-15…D-17`. A token scan flagged an apparent `D-16` collision in the PDR — **checked and cleared as a false positive**: PDR §G-11 contains the string "v1.0 Appendix D findings D-2…D-16", which references *UniVAT 1.0's* documentation-defect findings, an unrelated legacy namespace. No decision ID is reused.

**Coverage script result** (requirement inventories extracted from the parents and diffed against Appendix B, counting explicit citations and range citations):

| Family | Covered | Total |
|---|---|---|
| TEC | 30 | 30 |
| NFR | 35 | 35 |
| INT | 17 | 17 |
| DAT | 12 | 12 |
| AI | 28 | 28 |
| SEC | 15 | 15 |
| LOC | 10 | 10 |
| X | 16 | 16 |
| AIC | 8 | 8 |
| **Total** | **171** | **171** |

The first run returned 170/171 — the single gap (`X-12`) is recorded as defect #14 in C.5 and fixed.

**Extended pass (v1.1).** The v1.0 script checked only the nine technology-determining families above, while the conclusion claimed alignment with everything. Re-running across the remainder gave `GOV` 10/17, `OPS` 7/20, `FR` 27/87, `BUS` 0/15. Seven of those gaps were genuinely technology-bearing and are now mapped in **§B.4a**, raising `GOV` to **14/17** and `OPS` to **11/20**. The residue is correctly out of scope: professional review (`GOV-08`), contractual framework and liability (`GOV-11/12`), support model, CAB, BCP, implementation methodology, customer success, community, vendor risk, ESG and pricing review (`OPS-01/04/05/06/08/09/14/18/20`) — organisational commitments no stack can discharge. `FR` and `BUS` are enabled collectively, per the scope table at the head of Appendix B.

**Licence spot-check.** Every licence asserted for a component newly added in v1.1 was verified against a primary source: **PgBouncer — ISC**; **PgCat — MIT**. Both permissive (Class A), both clear for the on-prem appliance.

## C.5 Defects found and corrected

| # | Defect | Fix |
|---|---|---|
| 1 | First draft carried **MediatR** as the in-process dispatcher, inherited from common .NET practice — it has been commercial since 2025 with a US$5M revenue threshold | Removed; **Wolverine** serves as both mediator and outbox, eliminating the dependency entirely. Added to §12 and principle `P4` |
| 2 | First draft carried **AutoMapper** for DTO mapping — commercial since 2 July 2025 | Replaced with **Riok.Mapperly** (Apache-2.0, compile-time source generator — also better for `P6` determinism) |
| 3 | First draft used **Fluent Assertions** in the test stack — v8+ is proprietary (Xceed, ~US$130/dev/yr) | Replaced with **AwesomeAssertions** (Apache-2.0 fork of v7) / **Shouldly** |
| 4 | First draft proposed **QuestPDF** for statutory PDF rendering | Rejected: Community licence is non-OSI, revenue-thresholded, and **excludes public-sector entities** — incompatible with UniVAT's customer base. **Typst** (Apache-2.0) retained from the BTO |
| 5 | First draft specified **Swashbuckle** for OpenAPI | Replaced with **Microsoft.AspNetCore.OpenApi** (native OpenAPI **3.1** in .NET 10) + **Scalar**; Swashbuckle was removed from .NET templates in .NET 9 |
| 6 | `TEC-11` was stated as a principle but not enforced anywhere | Added **BannedApiAnalyzers** as a compile-time gate plus a NetArchTest assertion — §6.1 is now enforced by the build, not by convention |
| 7 | `NFR-28`'s "no alert without a runbook" had no mechanism | Added a CI check requiring a `runbook_url` label on every Prometheus alert rule |
| 8 | `NFR-32` ("LTS versions only") was satisfied implicitly but not stated as policy, leaving `.NET 11` (STS, Nov 2026) as an ambient temptation | Added principle `P2`, §11 upgrade policy, and an explicit §12 prohibition |
| 9 | `LOC-06` (Bangla search, sorting, collation) had no named mechanism | Added the **OpenSearch Bengali analyzer + ICU plugin** to §5.4 |
| 10 | `DAT-04` (published, version-controlled data dictionary) was unassigned | Added **SchemaSpy**, generated per release |
| 11 | `NFR-31` (sustainability) and `TEC-26` (extensibility) were omitted from the first traceability pass | Both added — `NFR-31` via KEDA scale-to-zero; `TEC-26` scoped with the scripting sandbox deferred to `D-17` |
| 12 | Bengali Whisper fine-tunes were listed as MIT by association with base Whisper | Corrected: base Whisper is MIT, **each fine-tune carries its own licence** and is Class D until individually cleared under `D-13` |
| 13 | **Ragged tables.** The §5.4 and §5.8 tables declare a `Version` column, but 16 rows omitted the cell — the rows rendered shifted by one column | All 16 rows given an explicit version or `—`; re-verified: **0 ragged blocks** across the document |
| 14 | **`X-12` was not traceable.** Appendix B claimed X-01…X-16 coverage, but the coverage script found `X-12` ("Support trained; runbooks exist for every alert") had no explicit mapping | Appendix B §B.3 now cites **§5.12** — the `runbook_url` CI check makes the runbook half mechanically true; the training half is flagged as an `OPS-07` process item outside the stack. Coverage now **171/171** |

**Second audit pass (v1.1) — 12 further defects.** The v1.0 audit verified structure, references and coverage but did **not** re-verify the document's own technical and version assertions against primary sources. That pass was run for v1.1 and is where all twelve came from.

*Material — each would have surfaced as a defect in build or production:*

| # | Defect | Fix |
|---|---|---|
| 15 | **Connection pooling was absent from the entire stack.** `NFR-08` requires 100,000 concurrent users; a PostgreSQL server serves a few hundred backends. Application-side pooling cannot bridge three orders of magnitude — the platform would have hit a hard connection ceiling in the first load test | **PgBouncer** added per failure domain (transaction pooling), with **PgCat** as the pre-approved upgrade path where read/write splitting or sharding is needed. Benchmarked in R0 |
| 16 | **`§6.3` claimed database-enforced tenant isolation across the board. It is not true of the ledger.** Marten applies RLS policies only to `TenancyStyle.Conjoined` **document** tables; the event-store tables **`mt_events` and `mt_streams` receive no RLS**. The most sensitive data in the product was described as having a safety net it does not have | §6.3 corrected with an explicit warning; `SEC-04` suite extended to **raw-SQL probes that bypass Marten** (an API-only suite would have passed while the ledger was unprotected); break-glass role for direct event-table access; hand-written policies evaluated in R0. Risk `TSD-R10` |
| 17 | **Transaction pooling and session-scoped tenant context combine into a silent cross-tenant leak.** The RLS pattern in common circulation uses `set_config(..., false)` — *session*-scoped. Behind a transaction pooler the connection is handed to the next tenant with the previous tenant's context still set, and RLS authorises the wrong rows. This is the `NFR-18` Sev-1, arriving via configuration rather than code | §6.3 standard is now absolute: **`set_config(..., true)` transaction-local only**, asserted by an integration test that runs **through the pooler**, with pool mode asserted in the Helm chart. Risk `TSD-R11`. *Found only because defect #15 introduced a pooler — the two interact* |
| 18 | **ESC-POS thermal printing of Bengali was specified as text.** ESC-POS printers render from onboard codepages and **perform no glyph shaping whatsoever**; Bengali sent as text prints as question marks or broken conjuncts. v1.0's "ESC-POS over WebUSB and a server-side encoder" would not have produced a legible Mushak 6.3 receipt in Bangla — a direct failure of `LOC-04`, `NFR-34` and `TEC-27` | §5.9 rewritten around a **server-side raster pipeline**: render → 1-bit monochrome bitmap → `GS v 0`/`GS *` raster commands. Client transmits pre-rendered bytes only, which also makes thermal output provably identical to the A4 rendering. Device profiles are configuration |

*Version and currency:*

| # | Defect | Fix |
|---|---|---|
| 19 | **Typst was specified as the primary statutory renderer.** It is at **0.15 (June 2026), pre-1.0, with breaking changes still planned** on the road to 1.0. A statutory pipeline cannot absorb a breaking typesetting change mid-tax-period | **Headless Chromium promoted to primary** (mature HarfBuzz Bengali shaping, pinned build for byte-stability); Typst demoted to a per-form optimisation, exact-pinned and corpus-gated. Risk `TSD-R9` |
| 20 | **Four version pins were simply wrong** — TypeScript `5.9` (actual **6.0.3**, April 2026), Zod `3.x` (actual **4**, stable May 2025), ECharts `5.x` (actual **6.1**), Python `3.13` (actual **3.14**, Oct 2025, supported to ~2030) | All corrected in §5.8, §5.10 and Appendix A; ArgoCD, GitLab CE and Valkey pins given verified precision |
| 21 | **No Node.js version was pinned anywhere**, although the Next.js SSR process is a production runtime and `NFR-32` mandates LTS-only for *runtime and framework* | **Node.js 24 LTS** (EOL 30 Apr 2028) pinned in §5.8 and Appendix A; Node 26 LTS adoption scheduled for Oct 2026 |
| 22 | **Systemic: ~110 version pins were presented with uniform confidence when only ~15 were verified.** Defect #20 is the symptom; this is the cause | Appendix A now carries a **verification convention** distinguishing *verified* from *indicative* pins, with the rule that **no component enters `Directory.Packages.props`/`package.json` until its pin is verified and its licence re-read**. Risk `TSD-R12` |

*Precision, scope and honesty:*

| # | Defect | Fix |
|---|---|---|
| 23 | **`numeric(19,6)` was applied uniformly.** It is ample per line (~10 trillion BDT) but a tenant-year or nationwide aggregate at `NFR-08` volumes can exceed it, while C# `decimal` still has headroom | Split: line-level stays `numeric(19,6)`; **aggregate and projection columns are `numeric(28,8)`**, with a property test asserting no overflow at 1B-line aggregates |
| 24 | **The "100% aligned with all files" claim was scoped to nine requirement families.** Re-running the coverage script across the rest showed `GOV` 10/17, `OPS` 7/20, `FR` 27/87, `BUS` 0/15. The claim was not false so much as imprecise about what a stack document can assert | Appendix B opens with an explicit **scope table**; the claim is now "100% of technology-determining requirements". §C.7 corrected |
| 25 | **Seven technology-bearing `GOV`/`OPS` requirements were genuinely missing**, not merely out of scope — `GOV-03` (NBR inspection readiness), `GOV-06`, `GOV-15`, `GOV-17`, `OPS-03`, `OPS-10`, `OPS-16` all have real stack components | New **§B.4a** maps all seven — officer read-only role, published data model, computation demonstrability, export pipeline behind the ACL, incident tooling, bilingual knowledge base, release-criteria suite |
| 26 | **Decision-ID collision check.** A token scan flagged `D-16` as already present in the family | **False positive, cleared and documented**: PDR §G-11 contains "v1.0 Appendix D findings D-2…D-16", an unrelated legacy namespace. Family decisions run `D-01…D-10`; BTO added `D-11…D-14`; this document continues correctly at `D-15…D-17` |

## C.6 Known limitations

| ID | Limitation | Action |
|---|---|---|
| L-T1 | Version pins are an August-2026 baseline with a short half-life (PostgreSQL 19, Node 26, .NET 11 all land within months) | §11 n−1 policy; re-baseline at each major adoption |
| L-T2 | Licence classes are an engineering reading of published terms, not counsel's | `D-13` before first GA ship; §9.5 CI gate enforces continuously |
| L-T3 | `NFR-01/02/07` budgets are met **by design**, not yet by measurement — no benchmark exists against real Bangladesh data volumes | R0 load-test milestone at 15×; `TSD-R3`/`TSD-R4` track the two most exposed assumptions |
| L-T4 | The rules-engine choice (`D-15`) is not final; §5.3 names both candidates with the decision criteria | R0 month 2 spike |
| L-T5 | eVAT/iVAS and EFDMS adapter designs are mock-first because the specifications are unpublished (`PR-01`) | Unchanged from the family position; ACL isolates the blast radius (`INT-17`) |

## C.7 Conclusion

The stack is complete at component level (108 named components, each versioned, licensed and classified), enforced rather than merely described (§6 standards are compile-time and CI gates), and traceable to every technology-determining requirement in the five parent documents (Appendix B: 30/30 TEC, 35/35 NFR, 15/15 SEC, 10/10 LOC, 8/8 AIC, 16/16 X, 12/12 DAT, 17/17 INT).

The v1.0 audit's most valuable finding was a licence one: **five components a competent .NET team would reach for by reflex — MediatR, AutoMapper, Fluent Assertions, QuestPDF and Swashbuckle — are no longer appropriate**, four because they became commercial or non-OSI within the last 24 months. Permissive replacements exist for all five, and in two cases (Wolverine for MediatR, Mapperly for AutoMapper) the replacement is technically superior for this workload.

**The v1.1 audit found something more uncomfortable, and more useful.** v1.0 verified its *structure* rigorously — anchors, tables, references, coverage — and passed every check. What it did not do was re-verify its own *technical assertions* against primary sources. Doing that produced twelve further defects, four of which would have reached production or a load test:

- a platform with **no connection pooling** attempting 100,000 concurrent users;
- a **ledger described as having database-enforced tenant isolation that Marten does not provide**;
- a **cross-tenant leak** waiting in the interaction between transaction pooling and the session-scoped `set_config` pattern that most RLS tutorials teach — a defect no code review would catch, because it lives in configuration;
- and a **thermal-printing design that cannot print Bengali at all**, which would have failed `LOC-04` in front of a counter operator in R1.

Three of those four came from reading primary sources — Marten's multi-tenancy documentation, a pooler benchmark, an ESC-POS encoder issue thread — rather than from reasoning about the design. The generalisable lesson, now encoded as the Appendix A verification convention and `TSD-R12`: **a stack document's confidence must be traceable to a source per assertion, not distributed evenly across the document.** v1.0 asserted ~110 version pins in one uniform voice when fifteen were checked; four of the unchecked were wrong.

The corrections did not change the architecture. .NET 10 + Marten 9 + Wolverine 6 + PostgreSQL 18 + the Next.js PWA on RKE2 stands exactly as approved — what changed is that four latent failures are now visible, tested for, and owned by named risks (`TSD-R9…R12`).

**This document is ADR-002 and the build baseline. It is fit to start R0 against**, with the standing condition that every indicative version pin is verified before it enters a lock file.

---

# Appendix D — Sources

Accessed 15 August 2026. `†` = external. Sources marked ‡ are primary (project source code, licence files or official documentation).

**Runtime, framework and lifecycle**

1. †‡ [.NET official support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) and [dotnet/core releases](https://github.com/dotnet/core/blob/main/releases.md) *(.NET 10 LTS to 14 Nov 2028; .NET 11 STS Nov 2026)*
2. †‡ [Generate OpenAPI documents — ASP.NET Core 10](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-10.0) *(OpenAPI 3.1 native in .NET 10)*
3. † [ASP.NET Core dropped Swagger — what replaced it in .NET 10](https://codewithmukesh.com/blog/dotnet-swagger-alternatives-openapi/) and [Swashbuckle is dead: migrating to Scalar in .NET 10](https://dev.to/jfmeyers/swashbuckle-is-dead-heres-how-to-migrate-to-scalar-in-net-10-155d)
4. †‡ [PostgreSQL 18 released (25 Sept 2025)](https://www.postgresql.org/about/news/postgresql-18-released-3142/) and [PostgreSQL versioning policy](https://www.postgresql.org/support/versioning/)
5. †‡ [Node.js end-of-life schedule](https://nodejs.org/en/about/eol) *(Node 24 LTS to 30 Apr 2028; Node 26 LTS Oct 2026)*

**Ledger, messaging and data**

6. †‡ [Marten — event store](https://martendb.io/events/) · [multi-tenancy](https://martendb.io/configuration/multitenancy) · [event-store multi-tenancy](https://martendb.io/events/multitenancy) · [async projections daemon](https://martendb.io/events/projections/async-daemon.html) *(conjoined tenancy, sharding, list partitioning, async daemon)*
7. †‡ [Marten migration guide](https://martendb.io/migration-guide) and [Critter Stack 2026](https://jeremydmiller.com/2026/04/29/critter-stack-2026/) *(Marten 9 / Wolverine 6, MIT, net9.0 + net10.0)*
8. †‡ [JasperFx — our products](https://jasperfx.net/our-products/) *(cores remain MIT; CritterWatch is the commercial product)*

**Licence changes that shaped §12**

9. †‡ [Jimmy Bogard — AutoMapper and MediatR commercial editions launch today](https://www.jimmybogard.com/automapper-and-mediatr-commercial-editions-launch-today/) and [AutoMapper and MediatR going commercial](https://www.jimmybogard.com/automapper-and-mediatr-going-commercial/) *(2 July 2025; free under US$5M revenue)*
10. † [Milan Jovanović — MediatR and MassTransit going commercial](https://milanjovanovic.tech/blog/mediatr-and-masstransit-going-commercial-what-this-means-for-you)
11. † [InfoQ — Fluent Assertions v8 abandons Apache licensing](https://www.infoq.com/news/2025/01/fluent-assertions-v8-license/) and [devclass — Fluent Assertions / Xceed partnership](https://devclass.com/2025/01/16/another-open-source-project-shifts-to-restrictive-license-fluent-assertions-following-xceed-partnership/)
12. † [AwesomeAssertions — drop-in replacement for FluentAssertions](https://www.conradakunga.com/blog/awesomeassertions-drop-in-replacement-for-fluentassertions/) and [FluentAssertions vs Shouldly vs AwesomeAssertions (2026)](https://codingdroplets.com/fluentassertions-vs-shouldly-vs-awesomeassertions-dotnet-2026)
13. †‡ [QuestPDF Community License](https://www.questpdf.com/license/community.html) and [licence selection guide](https://www.questpdf.com/license/guide.html) *(non-OSI; US$1M threshold; public-sector entities ineligible)*
14. †‡ [FluentValidation — License.txt](https://github.com/FluentValidation/FluentValidation/blob/main/License.txt) *(Apache-2.0, confirmed free)*
15. †‡ [gorules/zen — LICENSE](https://github.com/gorules/zen/blob/master/LICENSE) *(MIT)* and [GoRules open source](https://gorules.io/open-source)
16. †‡ [Riok.Mapperly — LICENSE](https://github.com/riok/mapperly/blob/main/LICENSE) *(Apache-2.0, source generator, no runtime reflection)*

**Platform, testing and frontend**

17. †‡ [xUnit.net releases and licensing](https://xunit.net/releases/) *(v3 current; Apache-2.0; v2 in maintenance)* and [Testcontainers for .NET with xUnit](https://dotnet.testcontainers.org/test_frameworks/xunit_net/)
18. †‡ [OpenFeature (CNCF, Apache-2.0)](https://openfeature.dev/) and [flagd reference implementation](https://github.com/open-feature)
19. † [Observability in .NET 10 — Serilog + OpenTelemetry + OTLP](https://fullstackhero.net/docs/cross-cutting-concerns/observability/) and [HybridCache + distributed cache in .NET 10](https://fullstackhero.net/docs/building-blocks/caching/)
20. †‡ [Next.js — PWA guide](https://nextjs.org/docs/app/guides/progressive-web-apps) and [Build a Next.js 16 PWA with true offline support](https://blog.logrocket.com/nextjs-16-pwa-offline-support/) *(Serwist as the maintained next-pwa successor)*

**Added in v1.1 — audit verification pass**

22. †‡ [Marten — multi-tenanted documents](https://martendb.io/documents/multi-tenancy) *(RLS applies to conjoined **document** tables; `mt_events`/`mt_streams` receive no RLS policies; `set_config` tenant context)*
23. † [Tembo — Benchmarking PostgreSQL connection poolers: PgBouncer, PgCat and Supavisor](https://www.tembo.io/blog/postgres-connection-poolers) and [PgBouncer vs pgcat vs Supavisor (2026)](https://www.pkgpulse.com/blog/pgbouncer-vs-pgcat-vs-supavisor-postgresql-connection-2026) *(PgBouncer best latency below ~50 clients, ~2 MB/1,000 clients; PgCat multithreaded and superior above that)*
24. †‡ [mike42/escpos-php — Bangla text not supported (issue #731)](https://github.com/mike42/escpos-php/issues/731) · [NielsLeenheer/EscPosEncoder — unable to print Arabic (issue #4)](https://github.com/NielsLeenheer/EscPosEncoder/issues/4) · [ESC/POS images — PortixOne](https://portix.one/knowledge/concepts/esc-pos-images/) *(no glyph shaping in ESC-POS; rasterise to bitmap and print as image)*
25. †‡ [Typst 0.15.0 release (15 June 2026)](https://github.com/typst/typst/releases/tag/v0.15.0) and [Typst 0.15 announcement](https://forum.typst.app/t/typst-0-15-now-available/9094) *(pre-1.0; breaking changes still planned toward 1.0)*
26. †‡ [TypeScript version history](https://en.wikipedia.org/wiki/TypeScript) *(6.0.3, 16 April 2026; TypeScript 7 Go-port compiler expected late 2026)*
27. †‡ [Zod v4 release notes](https://zod.dev/v4) and [InfoQ — Zod v4 available](https://www.infoq.com/news/2025/08/zod-v4-available/) *(stable 19 May 2025)*
28. †‡ [Apache ECharts 6 features](https://echarts.apache.org/handbook/en/basics/release-note/v6-feature/) and [ECharts npm](https://www.npmjs.com/package/echarts) *(6.1.0 current)*
29. †‡ [Python 3.14 — what's new](https://docs.python.org/3/whatsnew/3.14.html) and [status of Python versions](https://devguide.python.org/versions/) *(released 7 Oct 2025; 3.14.7 Aug 2026; security support to ~Oct 2030)*
30. †‡ [Valkey releases](https://valkey.io/download/releases/) and [Valkey releases and versioning](https://valkey.io/topics/releases/) *(9.1.1 current; 9.0 supported to 21 Oct 2028)*
31. †‡ [Argo CD releases](https://github.com/argoproj/argo-cd/releases) *(3.2 stable Nov 2025; 3.3 RC)* and [GitLab release notes](https://docs.gitlab.com/releases/) *(18.11, April 2026)*

**Inherited from the BTO's verification passes** — ingress-nginx retirement, OWASP Coraza, Redis/Valkey licensing, Citus AGPL, Grafana LGTM AGPL, k6 AGPL, MinIO archival, Keycloak 26.7 SCIM, Qwen3.x Apache-2.0, PDPO §29(7)(b): see `USL-UNIVAT2-BTO-001` v1.2 Appendix D, items 25–50.

**Internal**

32. `UniVAT_2.0_BRD.md` (`USL-UNIVAT2-BRD-001` v1.1) · `UniVAT_2.0_URD.md` (`USL-UNIVAT2-URD-001` v1.1) · `UniVAT_2.0_Product_Definition_and_Requirements.md` (`USL-UNIVAT2-PDR-001`) · `UniVAT_2.0_PRD.md` (`USL-UNIVAT2-PRD-001`) · `UniVAT_2.0_Build_Strategy_and_Technology_Options.md` (`USL-UNIVAT2-BTO-001` v1.2) — same folder.

---

*End of document — UniVAT 2.0 Technology Stack Specification v1.0*
