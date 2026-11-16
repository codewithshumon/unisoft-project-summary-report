# UniVAT 2.0 — Build Strategy & Technology Options

**How UNISOFT will build, develop and implement UniVAT 2.0: three open-source-based strategy choices (Option 1) and three fully-custom strategy choices (Option 2), evaluated against every requirement in the PRD, BRD and URD — for a full-stack team on Linux + VSCode, deployed to UNISOFT's own cloud in its own data center**

---

## Document control

| Field | Value |
|---|---|
| Document title | UniVAT 2.0 — Build Strategy & Technology Options |
| Document ID | USL-UNIVAT2-BTO-001 |
| Version | 1.2 (Draft for decision — post-audit revision + OPT1-A+ variant evaluation) |
| Date | 15 August 2026 |
| Status | Draft — awaiting CTO decision (feeds D-03/D-04/D-06; adds D-11/D-12/D-13/D-14) |
| Owner | Engineering / Architecture, UNISOFT Systems Limited |
| Classification | Confidential — Internal & Named Partners |
| Parent documents | `USL-UNIVAT2-BRD-001` v1.1 · `USL-UNIVAT2-PDR-001` · `USL-UNIVAT2-PRD-001` · `USL-UNIVAT2-URD-001` v1.1 — same folder |
| Delivery context | Built by the UNISOFT software development department: full-stack developers on Linux desktops, VSCode IDE, local dev/DB/build servers, Chrome DevTools; deployed after full test to UNISOFT's own cloud servers in UNISOFT's own data center |
| Standards applied | ISO/IEC/IEEE 29148 discipline inherited from the family; weighted-decision scoring; licence-risk analysis (OSI-approved classes only for zero-cost claims) |

### Revision history

| Ver | Date | Author | Summary |
|---|---|---|---|
| 1.0 | 15 Aug 2026 | Engineering / Architecture | First baseline. Requirement drivers extracted from PDR/PRD/BRD/URD; evaluation method; Option 1 (Odoo CE / Frappe-ERPNext / Apache OFBiz) and Option 2 (TypeScript end-to-end / .NET + TS / Java + TS) fully specified with components, licences, scoring, risks; shared own-DC platform blueprint; dev toolchain; licence register; delivery mapping; recommendation; mandatory audit. |
| **1.1** | **15 Aug 2026** | **Engineering / Architecture** | **Comprehensive external audit and revision.** 23 defects corrected (full table in [C.5](#c5-defects-found-and-corrected)). Material changes: NGINX Ingress Controller replaced (project retired March 2026); MassTransit removed from the zero-licence set (v9 commercial); Redis 8 / Citus / Grafana Tempo / GoRules ZEN / k6 licence classifications corrected; PostgreSQL baseline 17 → 18; Marten 7 → 9; ModSecurity supplemented by Coraza; PDPO 2025 / PDPA 2026 residency clock added as a first-class driver (§2.3) — it was absent from v1.0; licence register restructured into four risk classes (§9); cost-model section added (§7.7, new decision D-14); OPT1-C security score corrected on CISA-KEV evidence (61.7 → 60.7). Recommendation (OPT2-B primary, OPT2-A co-finalist) **unchanged and strengthened**. |
| **1.2** | **15 Aug 2026** | **Engineering / Architecture** | **Evaluation of the maximum-reuse Odoo variant requested by the business: Odoo 19 CE + OCA free modules + free Odoo App Store modules + custom ([§4.1.1](#411-variant-opt1-a--odoo-19-ce--oca--free-app-store-modules-maximum-reuse)).** Result: the variant scores **53.6** — *lower* than bare Odoo CE — because the OCA catalogue is predominantly **AGPL-3.0** and the binding OCA–Odoo licence agreement forbids proprietary modules from depending on AGPL modules, which would force UNISOFT's Bangladesh VAT engine to be published under AGPL to every network user. Two further first-order findings from reading Odoo 19's source: **`fields.Monetary` holds Python `float` in memory** (binary floating point — a direct TEC-11 violation), and **`restrict_mode_hash_table` already provides SHA-256 hash-chained, gapless, inalterable entries** (a genuine partial TEC-07/TEC-10 win, credited). Consequent re-scores on new evidence: OPT1-A 60.0 → **57.4**. Recommendation (OPT2-B primary, OPT2-A co-finalist) **unchanged**; §4.1.1 adds the constructive Odoo reuse paths UNISOFT should take instead. |

### Position in the document family

| | BRD | URD | PDR | PRD | **This document (BTO)** |
|---|---|---|---|---|---|
| Question | Why invest? | What must users do? | What must the product be? | What do we build in what order? | **With what foundation and stack do we build, develop and implement it?** |
| Feeds | — | — | — | — | Decisions `D-03` (rewrite), `D-04` (deployment model), `D-06` (AI build-vs-hosted); adds `D-11` (team-skill spike), `D-12` (own-DC capacity/DR), `D-13` (OSS licence counsel review), `D-14` (own-DC TCO baseline) |

> **Licence verification notice.** All licence claims below were re-researched from primary project sources and market reporting in **August 2026** and are listed per component in [§9](#9-licence-and-compliance-register). Licences change — and between v1.0 and v1.1 of this document alone, the audit found **six** component licence or lifecycle facts that had drifted. **Before commitment, every selected component's licence file must be re-read in-repo and cleared by counsel (D-13)** — especially the copyleft (GPL/LGPL/AGPL/MPL) classes and AI-model licences. "Zero licence cost" here means: no per-seat, per-tenant, per-revenue or subscription fee for the software itself under the stated licence class, self-hosted on UNISOFT hardware. **Zero licence cost is not zero cost** — see [§7.7](#77-cost-model-what-zero-licence-does-and-does-not-mean).

---

## Table of contents

1. [Purpose and decision context](#1-purpose-and-decision-context)
2. [Constraints and requirement drivers](#2-constraints-and-requirement-drivers)
3. [Evaluation method](#3-evaluation-method)
4. [Option 1 — open-source base strategies](#4-option-1--open-source-base-strategies)
5. [Option 2 — fully custom development strategies](#5-option-2--fully-custom-development-strategies)
6. [Cross-option comparison and recommendation](#6-cross-option-comparison-and-recommendation)
7. [Shared platform blueprint (own data center)](#7-shared-platform-blueprint-own-data-center)
8. [Development environment and toolchain](#8-development-environment-and-toolchain)
9. [Licence and compliance register](#9-licence-and-compliance-register)
10. [Delivery mapping](#10-delivery-mapping)
11. [Risks and open decisions](#11-risks-and-open-decisions)
12. [Recommendation summary](#12-recommendation-summary)
- [Appendix A — Glossary](#appendix-a--glossary)
- [Appendix B — Requirement-coverage matrix (recommended stack)](#appendix-b--requirement-coverage-matrix-recommended-stack)
- [Appendix C — Mandatory audit](#appendix-c--mandatory-audit)
- [Appendix D — Sources](#appendix-d--sources)

---

## 1. Purpose and decision context

### 1.1 What is being decided

The family documents fix *what* to build (PDR: 278 requirements; PRD: 19 epics, 118 stories, 54 rules) and *why* (BRD) and *for whom* (URD: 111 user requirements). This document decides the **foundation and technology stack** to build, develop and implement it, in two strategy families requested by the business:

- **Option 1 — open-source base**: a suitable open-source product (zero licence cost) as the foundation, plus another open-source product where needed, plus custom development on top. Three best choices: [OPT1-A](#41-choice-opt1-a--odoo-19-community--custom-bangladesh-vat-modules), [OPT1-B](#42-choice-opt1-b--frappe-framework--erpnext--custom-univat-app), [OPT1-C](#43-choice-opt1-c--apache-ofbiz--custom-vat-domain-on-a-permissive-java-platform) — plus the maximum-reuse variant [OPT1-A+](#411-variant-opt1-a--odoo-19-ce--oca--free-app-store-modules-maximum-reuse) (Odoo CE + OCA + free App Store modules), evaluated at the business's request in v1.2.
- **Option 2 — fully custom from scratch**: everything domain-specific written by the team on a commodity open-source infrastructure layer (which itself is zero-licence). Three best choices: [OPT2-A](#51-choice-opt2-a--typescript-end-to-end), [OPT2-B](#52-choice-opt2-b--net-10--typescript-frontend-recommended-primary), [OPT2-C](#53-choice-opt2-c--java-25--spring-boot--typescript-frontend).

### 1.2 Delivery constraints stated by UNISOFT (binding for all choices)

| # | Constraint | Consequence |
|---|---|---|
| BC-1 | Built by the UNISOFT software development department, staffed by **full-stack developers** | One-language or two-language stacks preferred; deep specialisation silos (separate frontend/backend/DBA/ops teams) not assumed |
| BC-2 | **Linux desktops, VSCode IDE** with required extensions, error-checking and monitoring tools | Every chosen stack must have first-class Linux + VSCode support (all six do; specifics in [§8](#8-development-environment-and-toolchain)) |
| BC-3 | Local development server, local database server, local build server | Docker-Compose-based local parity environment; GitLab CE runner on a shared build box ([§8](#8-development-environment-and-toolchain)) |
| BC-4 | Chrome DevTools, browser preview | Web-first frontend; Playwright + Lighthouse + axe in the loop |
| BC-5 | After implementation and testing, deploy to **UNISOFT's own cloud servers in UNISOFT's own data center** | Self-hosted everything; no managed-cloud proprietary services in the critical path (also mandated by `NFR-30`); `NFR-22` in-country residency is satisfied **by construction**; a single-site data center cannot alone meet `NFR-09/11/12` — see [§7.6](#76-disaster-recovery-gap-to-close-d-12) and decision `D-12` |
| BC-6 | Zero licence cost is the objective of Option 1 and the default posture of Option 2's commodity layer | Licence register and copyleft analysis in [§9](#9-licence-and-compliance-register); **cost model in [§7.7](#77-cost-model-what-zero-licence-does-and-does-not-mean)** — zero licence cost ≠ zero total cost |

### 1.3 The one thing research settled up front

**There is no mature open-source VAT-determination or Bangladesh-VAT compliance engine.** Searches across GitHub and the ecosystem (re-run August 2026) surface only rate databases (e.g. `kdeldycke/vat-rates` for EU), offline calculators (e.g. `node-sales-tax`), a small crop of **US-federal** income-tax engines that appeared during 2025–26 (`filedcom/opentax`, `Invaro/opentax-engine`) which model a different tax entirely, and general-purpose rules engines. Nothing addresses Bangladesh VAT, the Mushak form surface, or NBR filing.

**The statutory core — date-effective Bangladesh rules-as-data, the 34 end-to-end Mushak form generators (of 79 catalogued forms, PDR §Appendix A), the bitemporal append-only tax ledger, the NBR eVAT/iVAS connector — is custom development in every option, Option 1 included.** Option 1 differs from Option 2 only in how much *surrounding* scaffolding (masters, inventory, manufacturing, UI shell, tenancy) is adopted versus written. This finding drives the scoring: the base products are evaluated as scaffolding, not as VAT products.

---

## 2. Constraints and requirement drivers

### 2.1 Stack-determining requirements from the family

Condensed from `PDR-001` §12–§17, `PRD-001` §12–§15, and the BRD/URD; full coverage per requirement in [Appendix B](#appendix-b--requirement-coverage-matrix-recommended-stack).

| Driver | Requirement IDs | What it forces on the stack |
|---|---|---|
| API-first, UI-as-client | TEC-01, TEC-05, INT-10 | OpenAPI 3.1 spec-first REST plus webhooks, versioned `/v1` with a published **≥ 12-month deprecation policy**; GraphQL only for read-heavy composition where justified; every screen a client of the same API |
| Multi-tenant SaaS + on-prem artefacts | TEC-02, NFR-18, BRD-048 | Shared-schema tenancy with row-level isolation, or a provably operable tenancy model; the **same artefacts** must run at a customer site for Enterprise on-prem |
| Modular monolith first | TEC-03 | Enforced module boundaries in one deployable; no premature microservices |
| Event-driven core, outbox | TEC-04 | Durable bus + transactional outbox pattern |
| **Bitemporal, append-only ledger** | TEC-07, BR-071, BRD-005 | Event-sourced or equivalent design; no UPDATE/DELETE on tax postings; valid-time + transaction-time |
| Deterministic recomputation | TEC-08, US-015 | Same inputs + rule version ⇒ byte-identical outputs; golden-master harness (TEC-25) |
| Polyglot persistence | TEC-09 | PostgreSQL core + document store + object storage + columnar warehouse + search index |
| Gapless statutory numbering | TEC-10, BR-022 | DB-level sequence allocation with void-with-reason, no reuse |
| **Decimal-only money** | TEC-11 | No binary floating point in money paths — native decimal type or strict decimal library + integer-minor-unit policy; store at higher precision than displayed |
| Bangla + Bengali numerals, complex-script shaping | TEC-13, LOC-01…10, NFR-24/34 | Full Unicode, ICU message format, correct **conjunct (যুক্তাক্ষর) shaping** in UI/PDF/thermal (TEC-27). Bangla is a left-to-right complex script — the hard problem is glyph composition and font embedding, **not** bidirectional layout |
| TypeScript SPA/SSR frontend, PWA offline | TEC-14, TEC-28, UR-025, BR-024 | React-or-equivalent with service worker, IndexedDB, signed local journal |
| Backend runtime | TEC-15 | PDR text: *"A managed-memory, strongly typed stack (e.g. **.NET 8+/C#, Java 21+, or Go**) — chosen for team depth and long-term support. Migration off ASP.NET WebForms is non-negotiable."* |
| Kubernetes, IaC, DevSecOps | TEC-16…18, SEC-07 | Containers + K8s + declarative infra; SAST/DAST/SCA/secret-scan, SBOM, signed artefacts in CI |
| Feature flags per tenant | TEC-19, AIC-5 | Runtime toggles incl. regulatory switch-overs |
| Background jobs, DLQ, replay | TEC-20 | Reliable queue/scheduler with operator tooling |
| AI, self-hosted possible | AI-01…32, AIC-7, BRD-034/035 | OCR/VLM + classification + RAG + forecasting behind gates; inference on UNISOFT GPUs (AI-31 residency) with graceful degradation (AI-28) |
| Integrations | INT-01…17 | Anti-corruption layer; NBR eVAT/iVAS, e-invoice/EFDMS mocked-first (`PR-01`), a-Challan, ERP connectors, SAML/OIDC + SCIM (INT-11) |
| Scale & budgets | NFR-01…08, §14 PRD | p95 < 300 ms / p99 < 800 ms; 5M documents/month/tenant; 1B lines/year; 10k tenants; 100k concurrent users |
| Availability & DR | NFR-09…13 | 99.9%/99.95%; RPO ≤ 15 min; RTO ≤ 4 h (tested annually per NFR-11); cross-AZ default; in-country replica |
| Security baseline | SEC-01…15, NFR-14…20 | ASVS L2/L3, MFA, tenant-isolation tests in CI, immutable audit log (hash-chained) |
| **Data-protection residency clock** | NFR-22, BRD-012, C-06, BR-074 | PDPO 2025 / PDPA 2026 — see [§2.3](#23-the-legal-clock-pdpo-2025--pdpa-2026) |
| Portability | NFR-30 | No unremovable single-cloud proprietary dependency — self-hosting satisfies by construction |
| IP cleanliness | GOV-10, BRD-065 | Clean-room ownership; third-party licence register verified in CI — copyleft strategy must be explicit |

### 2.2 Team-fit driver

`PRD-001` §20.3 plans a department peaking at 53 engineers in R1 (30 → 53 → 65 → 79 across R0→R3) with full-stack developers as the building block (`BC-1`). Stack choice must minimise the number of languages a single developer must hold: at most **one backend language + TypeScript everywhere on the client**, plus Python only inside the AI workbench (self-contained service).

### 2.3 The legal clock: PDPO 2025 / PDPA 2026

*(New in v1.1 — this driver was missing from v1.0 despite being first-class in the BRD and PRD: `BRD-012`, `C-06`, `BR-074`, `NFR-22`.)*

Bangladesh's **Personal Data Protection Ordinance 2025** was gazetted on **6 November 2025**. A **March 2026 amendment** narrowed the original blanket cloud-localisation mandate: under the amended §29(7)(b), the obligation to keep **at least one synchronised real-time copy inside Bangladesh** now attaches specifically to **restricted personal data and Critical Information Infrastructure data** — which is exactly UniVAT's data class. The compliance window runs **18 months from gazette**, putting enforcement at approximately **May 2027**.

Three consequences for this document:

1. **It converts NFR-22 from an architectural preference into a dated legal obligation.** May 2027 falls inside the **R2** window (months 12–17) — the in-country synchronised copy must therefore be operating *before* R2 closes, not deferred to R3.
2. **It reinforces, and is reinforced by, D-12.** The second in-country site required for DR ([§7.6](#76-disaster-recovery-gap-to-close-d-12)) is the same asset that discharges §29(7)(b). One infrastructure decision satisfies both a reliability requirement and a statutory one — which is the strongest available argument for funding it early.
3. **It constrains the AI decision (D-06/AI-31).** Inference over restricted personal data must occur in Bangladesh or under an approved transfer mechanism. Self-hosted GPU inference in the own DC satisfies this by construction; any hosted-API fallback is limited to non-restricted content under AIC-7.

**Naming note.** NBR has rebranded the **Integrated VAT Administration System (iVAS)** as the **eVAT system**; functions and services are unchanged. The family documents use "iVAS" throughout. This document uses **"eVAT (formerly iVAS)"** on first mention in each section and keeps the `iVAS` token in connector/requirement names so traceability to `INT-*` and `PR-01` is preserved. A naming-alignment pass across the family is recommended but is not a stack decision.

---

## 3. Evaluation method

Twelve weighted criteria derived from §2 (weights sum to 100). Each choice is scored 0–10 per criterion by evidence from the family documents and the August-2026 research; weighted total = Σ(score × weight ÷ 10).

| # | Criterion | Weight | Derived from |
|---|---|---|---|
| C1 | Statutory-core correctness: decimal, bitemporal immutable ledger, determinism, gapless numbering | 15 | TEC-07/08/10/11; BR-022/071 |
| C2 | Architecture fit: API-first, tenancy, modular monolith, events/outbox, idempotency, flags | 12 | TEC-01…06, 19, 21 |
| C3 | Scale & performance headroom | 10 | NFR-01…08; PRD §14 |
| C4 | Offline PWA, Bangla parity, print fidelity | 8 | TEC-13/14/27/28; LOC-01…10 |
| C5 | Self-hosted AI enablement (incl. Bangla within 5 pp, GPU in own DC) | 8 | AI-01…32; PRD §13.1 |
| C6 | Integration surface & anti-corruption fit | 7 | INT-01…17 |
| C7 | Security & certification path (incl. the base product's own vulnerability history) | 5 | SEC-01…15; NFR-14…20 |
| C8 | Licence risk: zero-cost **and** clean for on-prem redistribution | 12 | GOV-10; TEC-02; BRD-065 |
| C9 | Team & toolchain fit (full-stack, Linux, VSCode) | 8 | BC-1…BC-4 |
| C10 | Delivery speed to R1 enlistment window | 7 | M5/M6 (month 11) |
| C11 | Own-DC operability & DR | 5 | NFR-09…13; TEC-16…20 |
| C12 | Long-term maintainability & talent pool | 3 | OPS-17; TEC-29 |

**Scoring hygiene (v1.1).** Criterion scores are expert judgement, but each must be *anchored to cited evidence*. Where the v1.1 audit found a score contradicted by hard evidence, the score was changed and the change is recorded in [C.5](#c5-defects-found-and-corrected). One such change was made (OPT1-C, C7).

---

## 4. Option 1 — open-source base strategies

### 4.0 Approach and licensing ground rules

Option 1 adopts an open-source product as the application foundation, a second open-source product where it fills a gap, and writes the Bangladesh VAT domain as custom modules. Three ground rules frame every choice:

1. **SaaS vs distribution.** UNISOFT will operate the SaaS from its own data center — network use of GPL code by customers is **not distribution**, so GPL source-sharing is not triggered for the SaaS. (Note the exception: **AGPL** *does* treat network interaction as triggering source availability — which is why the AGPL components in §9.3 are internal-only and never embedded in a shipped artefact.) But `PDR-001` requires an **Enterprise on-premise edition** (TEC-02) — shipping artefacts to a customer *is* distribution: GPLv3 derivatives must then be offered in source form to that customer; LGPLv3 permits proprietary modules kept as separate works; Apache-2.0/MIT impose no such obligation.
2. **The VAT core is custom anyway** (§1.3). The base product's job is masters, inventory/manufacturing scaffolding, UI shell, tenancy and CRUD velocity.
3. **Shared commodity layer.** All Option-1 choices sit on the same zero-licence infrastructure layer as Option 2 ([§7](#7-shared-platform-blueprint-own-data-center)): PostgreSQL 18, Keycloak, SeaweedFS, OpenSearch, Temporal, Superset, the Grafana LGTM stack, GitLab CE, RKE2, ArgoCD, OpenTofu, OpenBao, vLLM/Ollama + Qwen3.x/PaddleOCR.

### 4.1 Choice OPT1-A — Odoo 19 Community + custom Bangladesh VAT modules

**Composition**

| Layer | Component (licence) | Role |
|---|---|---|
| Base product | **Odoo 19 Community** (LGPL-3.0) | Masters, inventory, manufacturing, purchasing shell; multi-company; Bangla translations; OCA module ecosystem |
| Companion OSS | **Keycloak** (Apache-2.0) for SSO where needed; **Apache Superset** (Apache-2.0) for analytics; **Apache Camel** (Apache-2.0) or **Temporal** (MIT) for integration glue¹; **PaddleOCR + Qwen3.x via vLLM** for document AI | Identity, BI, integration, AI |
| Custom development | Full VAT determination engine (rules-as-data on a rules engine); 34 Mushak form generators with Bangla print pipeline; bitemporal append-only VAT subledger (Odoo's `account.move` is mutable and not bitemporal); eVAT/iVAS, a-Challan and EFD connectors; PWA offline counter; statutory numbering service; reconciliation engine | The actual product |

¹ **n8n** may be used *internally* for ops glue but uses the **Sustainable Use License** (fair-code), not OSI-open: zero-cost self-hosting is permitted, embedding/redistribution is not. It therefore cannot appear in the Enterprise on-prem appliance. Apache Camel, Airflow or Temporal are the redistributable substitutes.

**Key facts from research (Aug 2026).** Odoo 19 CE is LGPL-3.0 — proprietary custom modules are permitted if kept separable. **The full Accounting app is Enterprise-only**; Community ships "Invoicing" (invoices, vendor bills, payment recording) without bank synchronisation, automatic reconciliation, standard financial reports, or any AI features. Odoo's tenancy model is database-per-tenant for SaaS; the PRD's 10,000-tenant ambition (NFR-08) makes per-tenant DB operations the dominant cost. **Version-currency caveat:** Odoo 19 (Sept 2025) is the current stable release only until **Odoo 20**, expected at Odoo Experience on **24–26 September 2026** with GA 2–4 weeks later — i.e. roughly six weeks after this document's date. Any Option-1A plan must budget for a major-version migration in R0/R1, not after it.

**Assessment.** Fastest CRUD start; weakest statutory fit. The accounting model means the compliance ledger must be a parallel custom subsystem — at which point the base contributes masters/stock/mfg scaffolding while adding upgrade-coupling and performance ceilings (shared-nothing per-tenant DBs; Python ORM at 5M lines/month/tenant is unproven in this design). LGPL is workable for on-prem redistribution of the base + proprietary modules, subject to counsel review (D-13). Two corrections from the v1.2 source-level review are folded into §4.1.1 below and into the scores in §4.4.

#### 4.1.1 Variant OPT1-A+ — Odoo 19 CE + OCA + free App Store modules (maximum reuse)

*(Added in v1.2 at the business's request: "Odoo 19 CE (zero licence cost) + OCA free modules (zero licence cost) + free third-party modules from the Odoo App Store (zero licence cost) + custom development." This is a materially different proposition from §4.1's bare-CE case and was evaluated on its own terms, including a source-level review of Odoo 19 itself.)*

**The premise is sound and the ecosystem is real.** The OCA maintains 250+ repositories and 1,000+ production-grade modules, all free and OSI-licensed; the Odoo App Store carries thousands more, many free. On paper this closes most of the Community-Edition accounting gap identified in §4.1 — `account_financial_report` and `mis_builder` restore the financial reports Odoo reserves for Enterprise, `account_move_template` / `account_move_name_sequence` / `account_journal_lock_date` harden the journal, the `connector` and `queue_job` frameworks give integration and background-job infrastructure, and `base_tier_validation` gives approval workflows. Three findings, all verified against primary sources, nevertheless move the score **down** rather than up.

**Finding 1 — the AGPL dependency rule is the decisive one (C8: 7 → 3).**

The OCA's licence policy recommends AGPLv3, and the OCA board's standing decision is to keep existing addons on AGPL. The repository default across the accounting catalogue is AGPL-3.0 — spot-checked at module level: `OCA/account-financial-tools/19.0/account_move_name_sequence/__manifest__.py` declares `"license": "AGPL-3"`. The formal licence agreement between the OCA and Odoo SA — the instrument that makes LGPL core, AGPL community modules and proprietary Enterprise modules coexist — states the operative rule plainly:

> **OCA AGPLv3 contributions remain AGPLv3 and cannot be used as a dependency by any proprietary module.**

A proprietary module may depend on the LGPL Odoo core. It may **not** depend on an AGPL OCA module. So the moment a UniVAT module lists an AGPL OCA module in its manifest `depends`, that UniVAT module must itself be AGPL-3.0 — and AGPL §13's network clause means UNISOFT must offer its complete corresponding source to **every user of the hosted SaaS**. That is: the Bangladesh VAT determination engine, the rules-as-data content, the Mushak generators and the eVAT connector, published to the five NBR-enlisted competitors named in the PDR's market analysis. The Enterprise on-prem edition (TEC-02) would ship source as well.

This collides head-on with `GOV-10` ("clean-room ownership of all product IP") and with the commercial premise of the whole programme. There is an escape hatch — restrict the build to **LGPL-3-licensed OCA and App Store modules only** — but it is self-defeating: the LGPL subset is small and skewed to infrastructure (the job queue, some `server-tools`), while the accounting catalogue that constitutes this strategy's entire value proposition is the AGPL part. **The more of the free catalogue you use, the more of your own product you must give away.** That variant is scored separately below at 56.2 and is the only legally comfortable form of the strategy — but it is barely distinguishable from bare Odoo CE, which is the point.

**Finding 2 — Odoo's money is binary floating point (C1: 5 → 4).**

`TEC-11` is written as an absolute: *"Decimal arithmetic only (no binary floating point) for money and quantities."* Reading Odoo 19's ORM source directly (`odoo/orm/fields_numeric.py`, branch `19.0`):

- `Monetary._column_type = ('numeric', 'numeric')` — **storage** is correct, PostgreSQL `numeric`.
- But the in-memory type is native Python `float`. The class is typed `Field[float]` and conversion is `float(value or 0.0)`. **`decimal.Decimal` does not appear in the implementation.**
- `Float` is worse still: it maps to `float8`/`double precision` unless an explicit `digits` precision is set — the source comment states *"FLOAT8 type is still the default when there is no precision because it is faster for most operations (sums, etc.)"*.

So every tax computation, rounding and aggregation in Odoo happens in IEEE-754 binary floating point and is rounded back via `float_round` on the way to storage. For ordinary invoicing this is tolerable and thousands of businesses rely on it. For UniVAT it is not, on two counts: `TEC-11` forbids it outright, and `TEC-08` demands *byte-identical* recomputation from the same inputs and rule version — a guarantee that float summation, being order-sensitive, cannot give across 1-billion-line-per-year aggregates. Satisfying TEC-11 inside Odoo means bypassing the ORM for every money path, which is precisely the "parallel custom subsystem" that empties the base of its value. **This is architectural, not a setting.**

**Finding 3 — Odoo is stronger than v1.1 credited on immutability and offline (C4: 7 → 8, and partial credit on C1).**

In fairness, two v1.1 statements were too harsh and are corrected here:

- **Inalterability already exists.** With `restrict_mode_hash_table` enabled on a journal, Odoo chains every posted entry with **SHA-256** into an `inalterable_hash`, allocates a gapless `secure_sequence_number` ("Inalterability No Gap Sequence #"), refuses edits to secured entries, and cannot be switched off once used. Odoo even ships a *Data Inalterability Check Report*. That is a real, shipped answer to much of `TEC-07`'s append-only requirement and `TEC-10`/`BR-022`'s gapless numbering, and it is more than v1.1's "mutable `account.move`" implied. What it is **not** is bitemporal: there is no valid-time/transaction-time pair, so `TEC-07`'s "any historical position is reconstructible" and `BR-071` are still unmet, and rule-version pinning for `TEC-08` is absent entirely.
- **Offline POS is a genuine head start.** Odoo 18/19's Point of Sale ships as a PWA with IndexedDB-backed offline operation and automatic sync on reconnect — real progress against `TEC-28`/`UR-025`/`BR-024`. The caveats matter, though: the session must be opened online, offline data reflects no backend changes, and **unsynced orders can be lost if the browser cache is cleared** — which fails `TEC-28`'s "signed local journal that cannot be silently altered". It is a head start, not a solution.

**Supporting findings (smaller, but they all point the same way).**

| Area | Evidence | Effect |
|---|---|---|
| OCA porting lag | Odoo 19 shipped Sept 2025. Eleven months later `OCA/account-financial-tools` carries **21 modules on `19.0` against 41–47 on `18.0`** (~45% ported). Build on 19 and much of the catalogue is missing; build on 18 and you start a 24-month programme on the previous major | C10 unchanged (start is still fastest) but the advantage is thinner than it looks; dependencies port on someone else's schedule |
| App Store vetting | The Odoo App Store **does not pre-screen submissions for security**; the standing advice is to treat third-party modules as attack vectors and to commission independent penetration testing before production use. Free App Store modules are typically LGPL-3 (OPL-1 is the paid/proprietary licence) | C7 6 → 5. Every added free module enlarges an unvetted attack surface, against `SEC-01…15` and the R3 ISO 27001 path |
| Upgrade economics | Odoo ships a major version annually (Odoo 20 due 24–26 Sept 2026). Industry guidance is to upgrade every 12–18 months; **"the custom module count is the single biggest cost driver"**, and skipping versions costs disproportionately more. UniVAT would carry the largest custom-module footprint imaginable — the entire statutory domain | C12 7 → 5 |
| API shape | Odoo 19 adds a **JSON-2** interface and partial OpenAPI-conformant REST, but REST coverage is *"not yet comprehensive enough for most production integrations"* for accounting, and **XML-RPC/JSON-RPC are scheduled for removal in Odoo 22 (autumn 2028)** — a forced migration inside UniVAT's operating life | C2 unchanged at 5, but `TEC-01`/`TEC-05` spec-first API-first remains bent |
| Tenancy at 10k | Odoo tenancy is database-per-tenant. Per-tenant databases produce "infrastructure sprawl", "fragile orchestration", and *"transactional databases simply aren't designed to support tens of thousands of databases within a single instance"* — catalogue bloat and DDL bottlenecks. `NFR-08` requires 10,000 tenants | C11 6 → 5 |
| Single-tenant volume | **In Odoo's favour:** one Odoo client has processed over a billion accounting bookings, and large projects sustain 100k–300k transactions/day. The per-tenant volume in `NFR-08` is demonstrably achievable | C3 held at 5 rather than lowered — the constraint is the *tenant count*, not the per-tenant volume |
| Bangladesh localisation | `l10n_bd` exists in Odoo core (BAS-aligned chart of accounts, 15% standard VAT, supplementary duty, zero-rated/exempt mappings) — a genuine starting point. But neither Odoo nor ERPNext ships deep Bangladesh compliance; Mushak generation and NBR filing are partner-built every time. **No Odoo-derived product appears among the NBR-enlisted VAT vendors** (SmartVAT, Uddogi, Mediasoft, Prime VAT, DISYSTEM are all purpose-built) | Confirms §1.3: the statutory core is custom regardless |

**Scores.** Full variant **53.6**; LGPL-only variant **56.2**; both below bare Odoo CE as re-scored (**57.4**). The counter-intuitive result is the finding: *adding the free-module catalogue makes this option worse, not better*, because the catalogue's price is paid in licence obligations and maintenance surface rather than in money.

**What UNISOFT should do with Odoo instead.** Rejecting Odoo as the *foundation* is not rejecting Odoo. Three reuse paths are genuinely valuable and carry none of the above costs:

1. **Reference model.** Odoo's and ERPNext's accounting, stock and manufacturing schemas are mature, freely readable designs. Reading them to inform UniVAT's own masters is fast, legal and unencumbered — LGPL/AGPL obligations attach to *code*, not to learning from it. (Keep it clean-room-documented for `GOV-10`.)
2. **Migration importers (`DAT-02`).** Prospects running Odoo are prospects with structured data. An importer that reads an Odoo database is a sales asset, and it is a *separate tool* — its licence exposure does not touch the product.
3. **"UniVAT Connector for Odoo" as a distribution channel.** A published Odoo module that pushes a customer's Odoo transactions into UniVAT's API puts UNISOFT on the App Store in front of every Odoo-using Bangladeshi business, with the statutory engine staying proprietary on UNISOFT's side of the API. This is Odoo as a **channel**, not a foundation — the position that captures the ecosystem's value without inheriting its constraints. It fits `INT-01…17`'s ERP-connector scope and should be raised as a candidate for the R2/R3 connector backlog.

### 4.2 Choice OPT1-B — Frappe Framework (+ ERPNext) + custom UniVAT app

**Composition**

| Layer | Component (licence) | Role |
|---|---|---|
| Base platform | **Frappe Framework v16** (**MIT**) | Doc-type rapid application framework: auto-generated REST APIs, RBAC, background jobs, scheduler, multi-site (= multi-tenant), Bangla i18n, web/PWA shell |
| Optional base app | **ERPNext v16** (**GPL-3.0**) | Accounting/stock/manufacturing masters — **SaaS use safe; on-prem distribution of a derivative requires source-sharing**. Two sub-variants: **B1 (recommended)** build on Frappe Framework only (clean MIT IP); **B2** adopt ERPNext modules and accept GPL for on-prem |
| Companion OSS | Superset, Keycloak, SeaweedFS, OpenSearch, vLLM/PaddleOCR as in §4.0 | Same commodity layer |
| Custom development | Same VAT core as OPT1-A; plus (in B1) accounting-relevant masters the framework alone doesn't provide | The product |

**Key facts from research (Aug 2026).** Frappe Framework is MIT — the clean-IP path; ERPNext is GPLv3, so any derivative distributed on-prem must be offered as source to that customer. Version 16 of Frappe Framework, ERPNext and Frappe HR went beta in November 2025 and reached final release on **12 January 2026**, with substantial performance and stability work.

**Database correction (v1.1).** v1.0 stated that "ERPNext's PostgreSQL support is experimental/partial". The accurate 2026 position is sharper: **Frappe Framework** supports MariaDB, PostgreSQL and SQLite (v16 improved cross-database query handling for permission filters, joins and DB-specific functions), but **ERPNext itself does not support PostgreSQL** — it remains MariaDB-bound. This makes the B1/B2 split a *database* decision as much as a licence one: **B2 (ERPNext) is incompatible with TEC-09's PostgreSQL relational core** and would force either MariaDB for the ERP masters (a second RDBMS to operate, back up and secure) or a fork. B1 (Frappe Framework on PostgreSQL) remains viable but must be validated by a spike before commitment.

**Assessment.** The highest-velocity option for screens and CRUD (doc-types generate UI + API together), genuinely strong Bangla/multilingual posture, and B1 keeps IP clean. But: the ledger remains custom (mutable doctypes by default), per-site tenancy operations mirror Odoo's cost at scale, TEC-01/02's API-first multi-tenant shape is bent (Frappe's API is generated *after* the doctype, not spec-first), and B2 now carries a hard TEC-09 conflict.

### 4.3 Choice OPT1-C — Apache OFBiz + custom VAT domain on a permissive Java platform

**Composition**

| Layer | Component (licence) | Role |
|---|---|---|
| Base platform | **Apache OFBiz** (**Apache-2.0**, ASF-governed) | Java ERP *framework* (not turnkey): entity engine, service engine, accounting/inventory/manufacturing applications to harvest patterns from |
| Companion OSS | The full custom-stack layer of Option 2 (§5): Spring-modulith-style domain services may be embedded, React frontend, Keycloak, Temporal, etc. | Modern shell around the framework core |
| Custom development | The VAT core (same as above) **plus a full React/TypeScript frontend** — OFBiz's own UI stack is dated and would not meet URD/LOC UX requirements | The product |

**Key facts from research (Aug 2026).** The current release is **24.09.07 (June 2026)**. Two qualifiers materially affect the v1.0 assessment:

- **Feature freeze.** The 24.09 series has been **feature-frozen since September 2024** and receives bug fixes only. "Actively maintained" is true; "actively developed as a platform" is not. Adopting it means adopting a frozen framework surface for a 24-month build.
- **Vulnerability history.** OFBiz sustained a chain of unauthenticated remote-code-execution flaws through 2024 — **CVE-2024-32113, CVE-2024-36104, CVE-2024-38856 and CVE-2024-45195** — in which each fix was bypassed by the next disclosure. **CVE-2024-45195 was added to the CISA Known Exploited Vulnerabilities catalogue on 4 February 2025**, and CVE-2024-32113 was observed being exploited to deploy Mirai. For a product whose security requirement set is ASVS L2/L3 with an ISO 27001 path (SEC-01…15, R3), inheriting that attack surface is a direct C7 cost. **v1.1 lowers OPT1-C's C7 score from 6 to 4 on this evidence** (see [C.5](#c5-defects-found-and-corrected) #21).

iDempiere (GPL-2.0) is the alternative with the same "harvest a Java ERP core" idea but a copyleft licence — OFBiz's Apache-2.0 remains the cleanest base licence in this document.

**Assessment.** Best licence, weakest velocity, and now a documented security-history penalty: the framework's learning curve, dated conventions, frozen feature surface and thin modern-API/frontend story mean UNISOFT pays double — maintain OFBiz-era integration *and* build a modern shell. Its value here is mostly as a **reference implementation** of mature accounting/manufacturing data models.

### 4.4 Option 1 comparative scoring

| Criterion (weight) | OPT1-A Odoo CE | **OPT1-A+ CE + OCA + App Store** | OPT1-B Frappe/ERPNext | OPT1-C OFBiz |
|---|---|---|---|---|
| C1 Statutory core (15) | **4** ⟵ *v1.2: float money* | **4** | 5 | 6 |
| C2 Architecture fit (12) | 5 | 5 | 6 | 6 |
| C3 Scale headroom (10) | 5 | 5 | 5 | 7 |
| C4 Offline/Bangla/print (8) | 7 | **8** *(LGPL-only: 7)* | 8 | 4 |
| C5 Self-hosted AI (8) | 6 | 6 | 6 | 6 |
| C6 Integration (7) | 6 | **7** *(LGPL-only: 6)* | 6 | 6 |
| C7 Security path (5) | 6 | **5** ⟵ *unvetted App Store* | 6 | **4** ⟵ *v1.1: CISA-KEV evidence* |
| C8 Licence risk (12) | 7 | **3** ⟵ *AGPL dependency rule* *(LGPL-only: 7)* | 6 (B1: 8; B2: 5) | 10 |
| C9 Team/toolchain fit (8) | 6 | 6 | 6 | 5 |
| C10 Speed to R1 (7) | 8 | 8 *(LGPL-only: 7)* | 8 | 4 |
| C11 Own-DC ops (5) | **5** ⟵ *v1.2: DB-per-tenant* | **5** | 6 | 6 |
| C12 Maintainability (3) | **5** ⟵ *v1.2: annual majors* | **5** | 7 | 5 |
| **Weighted total (100)** | **57.4** *(was 60.0)* | **53.6** *(LGPL-only: 56.2)* | **60.8** (B1 63.2; B2 59.6) | **60.7** |

*Notes.* **(a)** B2 is scored explicitly (C8 = 5) because the ERPNext/PostgreSQL conflict in §4.2 makes it a genuinely worse variant, not a footnote. **(b)** OPT1-A's C1/C11/C12 were re-scored in v1.2 on evidence found while evaluating OPT1-A+ — the binary-float money model, the database-per-tenant ceiling and the annual major-version cadence are properties of Odoo itself, so they apply to the bare-CE case too. **(c)** OPT1-A+ scores *below* bare OPT1-A: the free-module catalogue buys velocity (C4, C6) at the cost of licence exposure (C8) and attack surface (C7). Full derivation and sources in [§4.1.1](#411-variant-opt1-a--odoo-19-ce--oca--free-app-store-modules-maximum-reuse).

### 4.5 Option 1 verdict

The whole family lands between **53.6 and 63.2**, roughly 25–35 points below every Option 2 choice: viable for an internal tool or a non-flagship product, **not recommended for the flagship SaaS** — because the parts that make UniVAT *UniVAT* (bitemporal ledger, rules-as-data determination, eVAT/iVAS filing, offline PWA, AI gates) are custom in every case, while the adopted base simultaneously adds licence-analysis overhead (AGPL/GPL/LGPL), tenancy-operations drag at 10k tenants, architecture friction with TEC-01/02/07/09/11, and — in OPT1-A and OPT1-B — a major-version upgrade train UNISOFT does not control.

**The maximum-reuse variant does not rescue the option; it deepens the problem.** OPT1-A+ (§4.1.1) is the strongest possible form of the "zero licence cost" argument — every free module the ecosystem offers, stacked — and it scores **lowest of all six Option 1 forms**. The reason is worth stating plainly for the decision record: *in the Odoo ecosystem the free accounting catalogue is AGPL, and AGPL modules may not be depended upon by proprietary ones.* The saving is real but it is paid for in the one currency UNISOFT cannot spend — the source code of the Bangladesh VAT engine, published to its competitors. Underneath that, Odoo's `Monetary` field is binary floating point in memory, which `TEC-11` prohibits outright.

Strategic reuse instead: ERPNext/Odoo stay valuable as **reference models and migration-source importers** (DAT-02); a published **"UniVAT Connector for Odoo"** turns the ecosystem into a distribution channel rather than a dependency (§4.1.1); and the OPT1-B1 Frappe variant is a legitimate accelerator for internal admin consoles.

---

## 5. Option 2 — fully custom development strategies

### 5.0 Approach: one commodity layer, three domain-language choices

"Fully custom" means: **all domain code is UNISOFT's** (clean IP, GOV-10), on a zero-licence commodity layer that is identical across the three choices (and reused by §7). Version pins are the **August 2026** baseline; §11 BTO-R12 governs how they are refreshed.

| Layer | Component (licence, class) |
|---|---|
| Database core | **PostgreSQL 18** (PostgreSQL Licence, permissive) + pgvector (PostgreSQL Licence). *v1.1: baseline moved 17 → 18 (GA 25 Sept 2025) for the asynchronous-I/O subsystem, `uuidv7()` and expanded index usage. PostgreSQL 19 is due September 2026 — adopt on the n−1 policy, not at GA.* |
| Object storage | **SeaweedFS** (Apache-2.0) — the most production-mature self-hosted S3 option in 2026, strong on billions of small objects; **Ceph RGW** (LGPL-2.1) as the erasure-coded, multi-failure-domain alternative at petabyte scale. Chosen after **MinIO's community edition went to maintenance mode (Dec 2025) and was archived read-only (25 Apr 2026)** under AGPLv3. Garage (AGPL-3.0) rejected for the appliance on licence grounds |
| Search | OpenSearch (Apache-2.0, Linux Foundation governance) |
| Cache | **Redis 8+ under the AGPLv3 election** (tri-licensed RSALv2 / SSPLv1 / **AGPLv3** since May 2025 — AGPLv3 is the OSI-approved option and the one UNISOFT elects) — **or Valkey (BSD-3-Clause, Linux Foundation)** where a permissive-only posture is required, e.g. inside the Enterprise on-prem appliance. *v1.1 correction: v1.0 recorded only the RSALv2/SSPL pair.* |
| Queues & streaming | RabbitMQ (MPL-2.0) primary; Kafka (Apache-2.0) only if event volume demands |
| Workflow & orchestration | Temporal (MIT) for long-running sagas (return assembly, filing retries); **Apache Airflow** (Apache-2.0) for ops-side glue. n8n (fair-code) internal-only, never in the appliance |
| Identity | **Keycloak 26.7** (Apache-2.0, released 9 July 2026): SAML/OIDC; **SCIM API now a preview feature** (experimental in 26.6, April 2026) with full CRUD/PATCH, filtering, pagination and Enterprise User extension — Metatavu SCIM extension retained as the abstracted fallback (INT-11). 26.7 also previews **multi-cluster HA without external caches**, directly relevant to §7.6 |
| BI & warehouse | Apache Superset (Apache-2.0) embedded; columnar via **ClickHouse (Apache-2.0)** when scale demands (DAT-06). *v1.1: Citus is **AGPL-3.0**, not Apache-2.0 — it remains usable self-hosted but is no longer the default warehouse recommendation and moves to §9.3* |
| AI runtime | vLLM (Apache-2.0) + Ollama (MIT) on UNISOFT GPUs; models: **Qwen3.x family (Apache-2.0)** primary — currently Qwen3.5 (Feb 2026) / Qwen3.6 (Apr 2026), both Apache-2.0. Llama 4's Community License is source-available with MAU/AUP conditions, so it is a counsel-gated fallback, not the default; DeepSeek (MIT) as coding/analysis assist |
| Document AI | PaddleOCR (Apache-2.0) + the Bengali-specific **bbOCR** pipeline patterns; VLM extraction via Qwen3.x-VL (Apache-2.0, 256K context, multilingual OCR); Tesseract (Apache-2.0) for the clean-scan fast path |
| ASR (R3, LOC-07) | Fine-tuned Bengali Whisper ecosystem (Bangla-WhisperDiar; HuggingFace `BanglaASR` / `whisper-bangla`; offline `BanglaSpeech2Text`). **Base Whisper is MIT; each fine-tune carries its own licence and must be cleared individually (D-13)** |
| Observability | Prometheus (Apache-2.0) + OpenTelemetry (Apache-2.0) + **Grafana, Loki, Tempo and Mimir (all AGPL-3.0 — self-host only, never embedded)**; error tracking **GlitchTip** (MIT) — Sentry's FSL licence noted as non-OSI. *v1.1: Tempo was mis-filed as Apache-2.0 in v1.0* |
| Edge & ingress | **Gateway API** implementation — **Envoy Gateway** (Apache-2.0, CNCF) or **Traefik v3** (MIT) — plus **OWASP Coraza** (Apache-2.0, Go, SecLang/CRS-compatible) as the WAF engine. *v1.1: the Kubernetes **ingress-nginx** controller is retired as of **March 2026** (repository archived; no further releases or security patches) and its intended successor **InGate was also retired**. ModSecurity's commercial support ended July 2024; it is now an OWASP community project, with Coraza the recommended cloud-native successor.* ModSecurity + CRS remains acceptable for non-Kubernetes edge appliances |
| Platform | RKE2 (Apache-2.0) production Kubernetes; K3s (Apache-2.0) for edge/sandbox; ArgoCD (Apache-2.0) GitOps; **OpenTofu** (MPL-2.0) — Terraform-compatible, chosen because Terraform moved to BUSL (documented deviation from TEC-17's "Terraform/Bicep" wording); **OpenBao** (MPL-2.0) for secrets — Vault moved to BUSL (deviation from TEC-22 wording) |
| CI/CD & supply chain | GitLab CE (MIT, self-hosted) + runners; Trivy (Apache-2.0), Semgrep OSS (LGPL-2.1 — tool, not shipped), Gitleaks (MIT), Syft (Apache-2.0, SBOM), Cosign (Apache-2.0, signing) |
| Test | Playwright (Apache-2.0), JMeter (Apache-2.0), Locust (MIT); **k6 is AGPL-3.0** — permitted as an internal, non-distributed load-testing tool only (*v1.1: v0.1's "where licensed terms are confirmed" placeholder now resolved*) |
| Frontend (all three choices) | TypeScript + **React 19 + Next.js 16** (MIT), PWA service worker, IndexedDB (Dexie, Apache-2.0) signed local journal, thermal-print pipeline (WebUSB/ESC-POS + PDF via Typst or headless Chromium), **Noto Sans Bengali / SolaimanLipi fonts under SIL OFL 1.1** (embedding permitted; licence text must ship with the appliance) |

The three choices differ only in the **backend domain language and framework**.

### 5.1 Choice OPT2-A — TypeScript end-to-end

| Aspect | Selection |
|---|---|
| Runtime | Node.js 24 LTS (Active LTS; EOL 30 April 2028). Node 26 enters LTS in October 2026 — plan the n+2 hop in R2 |
| Backend framework | **NestJS 11** modular monolith — modules mirror PDR domains (determination, documents, ledger, filing, reconciliation, AI gateway); Fastify adapter; SWC compiler |
| API | Spec-first OpenAPI 3.1 (tsoa/nestia codegen) — TEC-01/05 |
| Data access | Drizzle ORM (MIT) or Prisma (Apache-2.0); raw SQL for ledger paths |
| Money & quantities | **`decimal.js` + integer-minor-unit (poisha) `bigint` storage policy** + property-based tests — TEC-11 met by discipline, not by the type system |
| Events/outbox | BullMQ (MIT) on Redis/Valkey for jobs; RabbitMQ + a hand-rolled transactional-outbox module for domain events (TEC-04) |
| Rules engine | **GoRules ZEN** (**MIT**; Rust core, first-class Node bindings) for rules-as-data (FR-10); content authored as versioned JSON decision models. *v1.1: ZEN is MIT, not Apache-2.0 as stated in v1.0 — still fully permissive* |
| Ledger design | Custom append-only event/entry tables in PostgreSQL with bitemporal columns (valid_from/valid_to, tx_from/tx_to) — no ORM update path exposed |
| AI gateway | Node service calling vLLM/Python AI workbench; document intake via PaddleOCR service |
| Team/toolchain | **The single-language stack** — one developer moves across PWA, API, workers; matches BC-1…BC-4 exactly |
| Deviations to flag | (a) TEC-15 lists ".NET 8+/C#, Java 21+, or Go" as *examples*; TypeScript/Node qualifies as "managed, strongly-typed" only via TS typing at compile time — **requires the D-11 decision record**, and the decimal-discipline risk (C1) is the price. (b) No mature framework-native event store: the bitemporal ledger is hand-built, where OPT2-B gets it from a library |

### 5.2 Choice OPT2-B — .NET 10 + TypeScript frontend (recommended primary)

| Aspect | Selection |
|---|---|
| Runtime | **.NET 10 (LTS, released 11 Nov 2025, supported to 14 Nov 2028)** on Linux; C#; VSCode + C# Dev Kit. *.NET 11 (Nov 2026) is **STS** with only 2 years of support — stay on the LTS train; next hop is .NET 12 LTS (Nov 2027)* |
| Backend framework | ASP.NET Core **modular monolith** — one `UniVat.App` host, domain modules as separate assemblies with enforced boundaries (TEC-03); minimal-API endpoints generated spec-first (OpenAPI 3.1) |
| Data & ledger | **Marten 9 (MIT) — PostgreSQL event store + projections**: the append-only, bitemporal tax ledger as a first-class event stream (TEC-07 sweet spot); EF Core (MIT) for relational masters; optimistic concurrency (TEC-10). *v1.1: v1.0 specified **Marten 7**, which predates .NET 10 support — an internal contradiction with the .NET 10 runtime pin. Marten 9 is the Critter Stack 2026 release (JasperFx 2.0 / Weasel 9 / Marten 9 / Wolverine 6), targeting net9.0 and net10.0* |
| Money | **Native `System.Decimal`** end-to-end (TEC-11 sweet spot); `numeric` in PostgreSQL |
| Events/outbox | **Wolverine 6 (MIT)** — durable transactional inbox/outbox with native Marten integration, or **dotnetcore/CAP (MIT)** as the framework-independent alternative, over RabbitMQ (TEC-04/06). *v1.1: **MassTransit is removed from the zero-licence set** — v9 moved to a commercial licence (minimum ~US$400/month per organisation) and v8's Apache-2.0 line is only guaranteed security patches through the end of 2026. Retaining it would have breached BC-6* |
| Rules engine | **Microsoft RulesEngine** (MIT) — JSON rules-as-data; **NRules** (MIT) for complex Rete needs; GoRules ZEN (MIT) via its C# bindings if a shared engine across OPT2-A/B is preferred (FR-10, GOV-07) |
| Background | Quartz.NET (Apache-2.0) preferred for shipped artefacts; Hangfire (LGPL-3.0) internal-only; Temporal for sagas |
| Frontend | The shared TypeScript/React 19/Next.js 16 PWA (§5.0) — two languages total: C# + TS |
| Determinism & tests | Deterministic recompute service + golden-master harness (TEC-08/25); statutory conformance suite in xUnit; NodaTime (Apache-2.0) for Asia/Dhaka period boundaries |
| Team/toolchain | First-class Linux + VSCode; a C#-fluent full-stack team is the norm in BD enterprise shops — **conditional on D-11 confirming team depth** |
| Fit notes | Native decimals + Marten's event-sourcing hit the two hardest requirements (C1) head-on; MIT/Apache licences throughout the domain layer keep Enterprise on-prem redistribution trivial (C8). **Watch item:** JasperFx operates an open-core model — Marten and Wolverine cores remain MIT, but the CritterWatch monitoring product is commercial. Nothing required by this architecture is behind that paywall; the open-core pattern is nonetheless the exact shape that produced the MassTransit surprise, so it is tracked under BTO-R10 |

### 5.3 Choice OPT2-C — Java 25 + Spring Boot + TypeScript frontend

| Aspect | Selection |
|---|---|
| Runtime | **Java 25 LTS** (Sept 2025; OpenJDK builds — Temurin, GPL-2.0 w/ Classpath Exception) |
| Backend framework | **Spring Boot 4.1** (released **10 June 2026**, on Spring Framework 7.0.x; Apache-2.0) + **Spring Modulith** for enforced module boundaries (TEC-03) |
| Data | jOOQ Open Source Edition (Apache-2.0 — permissive for open-source databases including PostgreSQL; commercial editions only for Oracle/SQL Server) for spec-tight SQL + Spring Data JPA; custom append-only ledger tables with bitemporal columns; ShedLock/Quartz for scheduling |
| Money | **`BigDecimal`** native (TEC-11 sweet spot) |
| Events/outbox | RabbitMQ/Kafka + outbox via Spring transactions + Debezium (Apache-2.0) CDC option |
| Rules engine | **Drools** (Apache-2.0) — powerful but heavyweight; GoRules ZEN (MIT) via sidecar as the lighter alternative |
| Frontend | Shared TypeScript/React 19/Next.js 16 PWA (§5.0) |
| Team/toolchain | Excellent Linux/VSCode support (Extension Pack for Java); strongest local enterprise talent pool alongside .NET |
| Fit notes | Highest raw throughput headroom for the 5M-docs/month Large segment. **Maintenance caveat (v1.1):** Spring Boot ships minors roughly every 6–8 months with ~12–13 months of OSS support each — Boot 4.1's free support ends **31 July 2027**, and Boot 4.0's in December 2026. Over a 24-month build plus operations, that is a recurring, non-optional upgrade tax (or a paid commercial-support subscription, which would breach BC-6). JVM GC tuning and Spring's breadth are the other operational prices |

### 5.4 Considered and rejected (all options)

| Candidate | Why rejected |
|---|---|
| Go backend | Listed in TEC-15, but: no native decimal (library discipline like TS), thinner ERP-domain ecosystem, smaller BD hiring pool vs Java/.NET — scored below all three keepers |
| PHP / Laravel, Python / Django for the core | Team-signal ("full-stack") aside, both fail C1 discipline expectations (dynamic typing around money) and C3 headroom at the top segment; Python stays **inside the AI workbench only** |
| Ruby / Rails | Ecosystem and talent pool in Bangladesh too thin for a 24-month programme |
| Microservices-first | Explicitly against TEC-03; modular monolith in all choices, services extracted later where scale demands |
| MongoDB / NoSQL-primary ledger | Against TEC-09's relational-core for transactional integrity and gapless numbering |
| MinIO as object store | Community edition in maintenance mode Dec 2025, repository archived read-only 25 Apr 2026, AGPLv3 — replaced by SeaweedFS/Ceph in all choices |
| **MassTransit (v1.1)** | v9 is commercially licensed (~US$400/month minimum); v8's Apache-2.0 line gets security patches only through end-2026. Fails BC-6. Replaced by Wolverine 6 (MIT) / CAP (MIT) |
| **ingress-nginx (v1.1)** | Kubernetes project retired March 2026 (archived, unpatched); successor InGate also retired. Replaced by a Gateway API implementation |
| **Citus as the default warehouse (v1.1)** | AGPL-3.0, not Apache-2.0 as recorded in v1.0. Usable self-hosted but disqualified from the on-prem appliance; ClickHouse (Apache-2.0) becomes the default |
| Llama-family as default LLM | Community License is source-available with MAU/AUP conditions — Qwen3.x (Apache-2.0) is the clean default; Llama only with counsel sign-off |
| Sentry (self-hosted) | Functional Source Licence is non-OSI — GlitchTip (MIT) chosen instead |

### 5.5 Option 2 comparative scoring

| Criterion (weight) | OPT2-A TypeScript | OPT2-B .NET + TS | OPT2-C Java + TS |
|---|---|---|---|
| C1 Statutory core (15) | 7 | **10** | 9 |
| C2 Architecture fit (12) | 9 | 9 | 9 |
| C3 Scale headroom (10) | 7 | 9 | 9 |
| C4 Offline/Bangla/print (8) | 9 | 9 | 9 |
| C5 Self-hosted AI (8) | 9 | 8 | 8 |
| C6 Integration (7) | 8 | 8 | 8 |
| C7 Security path (5) | 7 | 8 | 8 |
| C8 Licence risk (12) | 10 | 10 | 10 |
| C9 Team/toolchain fit (8) | **10** | 7 | 6 |
| C10 Speed to R1 (7) | 8 | 8 | 7 |
| C11 Own-DC ops (5) | 9 | 9 | 9 |
| C12 Maintainability (3) | 8 | 8 | 8 |
| **Weighted total (100)** | **84.3** | **88.1** | **85.1** |

*Scores are unchanged from v1.0: the v1.1 corrections replaced components with same-class equivalents (MassTransit → Wolverine, both MIT; Marten 7 → 9, both MIT; ingress-nginx → Gateway API, both Apache-2.0) and so did not move any criterion. The Spring support-cadence finding (§5.3) argues for reducing OPT2-C's C12 by one point; it was left unchanged because a 3-weight change of 1 point moves the total by 0.3 — inside judgement error (L-B1) and immaterial to the ranking.*

---

## 6. Cross-option comparison and recommendation

### 6.1 Full matrix

| | OPT1-A | **OPT1-A+** | OPT1-B | OPT1-C | OPT2-A | **OPT2-B** | OPT2-C |
|---|---|---|---|---|---|---|---|
| Weighted score | 57.4 | **53.6** (LGPL-only 56.2) | 60.8 (B1 63.2 / B2 59.6) | 60.7 | 84.3 | **88.1** | 85.1 |
| Licence posture | LGPL base | **LGPL base + AGPL modules ⇒ product source published** | MIT base (GPL if ERPNext adopted) | Apache-2.0 base | All permissive | **All permissive; ledger + messaging libs MIT** | OpenJDK GPL-CPE + Apache-2.0 |
| TEC-11 decimal money | **Fails — ORM holds `float`** | **Fails — ORM holds `float`** | Custom (Python) | `BigDecimal` available | Discipline (`decimal.js` + bigint) | **Native `System.Decimal`** | Native `BigDecimal` |
| Statutory-core risk | High (custom anyway + base friction) | High | High | Medium | Medium (decimal discipline) | **Low (native decimal + event-sourced ledger)** | Low |
| Time-to-R1 | Fast start, slow finish (paradox of the custom core) | Fastest start, slowest finish | Same | Slow | Baseline | Baseline | Baseline − |
| 10k-tenant SaaS ops | DB-per-tenant drag | DB-per-tenant drag | Site-per-tenant drag | Framework-managed | Shared-schema RLS | **Shared-schema RLS** | Shared-schema RLS |
| Enterprise on-prem edition | LGPL separability work | **Ships product source (AGPL)** | Clean if B1 | Clean | Clean | **Clean** | Clean |
| TEC-09 PostgreSQL core | Native | Native | **B2 conflicts (MariaDB-bound)** | Native | Native | **Native** | Native |
| Upgrade train UNISOFT controls | No (Odoo 20 due Sept 2026) | **No — plus OCA porting lag (~45% at 11 months)** | No (v16 → v17 cadence) | Frozen branch | Yes | **Yes** | Partly (Spring ~12-mo minors) |

### 6.2 Recommendation

1. **Primary: Option 2, Choice OPT2-B — .NET 10 LTS modular monolith + Marten 9 event-sourced PostgreSQL 18 ledger + Wolverine 6 outbox + native decimals + the shared TypeScript/React 19/Next.js 16 PWA + the §5.0 commodity layer — on RKE2 in UNISOFT's own data center.** It directly answers the two hardest engineering requirements in the family (TEC-07 bitemporal append-only ledger via Marten; TEC-11 native decimal) with the cleanest licence posture, and it stays inside TEC-15's stated runtime examples.
2. **Fallback / co-finalist: OPT2-A (TypeScript end-to-end)** — within 3.8 points, and wins on team fit (C9). If the D-11 assessment shows the department is JavaScript-first with little C# depth, **choose OPT2-A without hesitation**; the decimal-discipline risk is manageable with bigint-poisha storage, decimal.js at edges, property tests, and golden-master gates. Note the second, quieter cost: OPT2-A hand-builds the bitemporal ledger that OPT2-B gets from Marten — budget it explicitly in R0.
3. **Option 1 is not recommended for the flagship** (53.6–63.2): the statutory core is custom in every scenario, and an adopted ERP base adds licence analysis, tenancy-ops drag, architecture friction, and an upgrade train UNISOFT does not control. **The maximum-reuse Odoo variant (OPT1-A+ — CE + OCA + free App Store modules) was evaluated at the business's request and scores lowest of all**, because the free accounting catalogue is AGPL-3.0 and AGPL modules may not be depended upon by proprietary ones — the zero-licence saving would be paid for with the published source of UNISOFT's VAT engine (§4.1.1). **Reuse instead**: Odoo/ERPNext as reference models and DAT-02 migration importers; a published **"UniVAT Connector for Odoo"** to make the ecosystem a distribution channel; Frappe (MIT) optionally for internal admin tooling.
4. **Decide by spike, not by committee** (closes D-11 in ≤ 2 weeks): two-pair teams build the *same* vertical slice on OPT2-B and OPT2-A — determination of one 15% line + one SD line, Mushak 6.3 issuance, append-only ledger posting with bitemporal query, one golden-master test, one PWA screen in Bangla. Score on velocity, defect feel, and developer preference.
5. **The recommendation is reversible up to the end of R0, and expensively so afterwards.** The shared commodity layer (§5.0), the API contracts, the rules-as-data content, the golden-master corpus, the Helm/GitOps assets and the entire frontend are stack-independent — a language switch after R0 would rewrite the domain services and the ledger, not the platform. This is why D-11 is a month-1 decision and not a month-6 one.

### 6.3 Decision-gate updates this document feeds

| Gate | This document's input |
|---|---|
| `D-03` rewrite vs strangler-fig | Confirms **rewrite** (all options are new builds; strangler-fig of ASP.NET WebForms scored below every keeper in C1/C2, and TEC-15 makes migration off WebForms non-negotiable) |
| `D-04` cloud vs on-prem | Defines **own-cloud on own metal** (BC-5): Kubernetes-first, portable by NFR-30; Enterprise on-prem = same Helm charts |
| `D-06` AI build vs hosted | Own-DC residency (AI-31 + PDPO §29(7)(b)) argues **self-hosted inference** (vLLM + Qwen3.x); hosted APIs allowed only for non-restricted workloads under AIC-7 |
| **New `D-11`** | Team-skill spike → pick OPT2-B vs OPT2-A (owner: CTO; by end of month 1) |
| **New `D-12`** | Own-DC readiness: second failure domain + GPU nodes + in-country backup site (owner: CTO/COO; before R0 exit) — **now also the PDPO §29(7)(b) compliance asset, deadline ~May 2027** |
| **New `D-13`** | OSS licence counsel review of the final register incl. copyleft, AI-model and font licences (owner: CISO/Legal; before first GA ship) |
| **New `D-14`** | **Own-DC TCO baseline** — quantify §7.7's cost drivers (hardware, GPU, DR site, power/cooling, bandwidth, platform-engineering headcount) and feed the family's D-02 bottom-up estimate (owner: CTO/CFO; before R0 exit) |

---

## 7. Shared platform blueprint (own data center)

Applies to all Option 2 choices (and the commodity layer of Option 1).

### 7.1 Topology (textual)

```
UNISOFT OWN DATACENTER (two failure domains / rooms, D-12)
┌────────────────────────────────────────────────────────────────────┐
│ EDGE: dual LB (HAProxy/kube-vip) → WAF (OWASP Coraza, Apache-2.0)  │
│       → Gateway API (Envoy Gateway or Traefik v3)                  │
│         [ingress-nginx retired Mar 2026 — do not deploy]           │
├────────────────────────────────────────────────────────────────────┤
│ RKE2 CONTROL (3 servers, spread across failure domains)            │
│ WORKER POOLS:  app pool │ print/render pool │ integration pool     │
│                ai-gpu pool (vLLM, PaddleOCR) │ build pool (GitLab) │
├────────────────────────────────────────────────────────────────────┤
│ DATA: CloudNativePG PostgreSQL 18 clusters (primary+replica per    │
│       failure domain; pgvector enabled) · Redis/Valkey sentinel    │
│       RabbitMQ quorum queues · SeaweedFS (or Ceph RGW at scale)    │
│       OpenSearch · ClickHouse (warehouse, when DAT-06 lands)       │
├────────────────────────────────────────────────────────────────────┤
│ PLATFORM SERVICES: Keycloak 26.7 · Temporal · Airflow/n8n (ops)    │
│       Superset · OpenBao (secrets) · ArgoCD (GitOps)               │
│       feature-flag service · cert-manager                          │
├────────────────────────────────────────────────────────────────────┤
│ OBSERVABILITY: Prometheus + OTel + Grafana/Loki/Tempo/Mimir (AGPL, │
│       internal only) · GlitchTip (errors) · status page            │
├────────────────────────────────────────────────────────────────────┤
│ INTEGRATION ZONE (anti-corruption layer, INT-17):                  │
│   mock-eVAT/iVAS · mock-EFDMS containers (spec-first, PR-01)       │
│   a-Challan · MFS/bKash adapters · ERP connector pool              │
│   EFD/SDC vendor SDKs (NOT open source — device procurement item)  │
└────────────────────────────────────────────────────────────────────┘
        ▲ Velero + Restic backups → second in-country site
          (D-12 · also discharges PDPO §29(7)(b), deadline ~May 2027)
```

**Edge-layer note (v1.1).** The v1.0 topology specified the Kubernetes **ingress-nginx** controller. That project entered best-effort maintenance in November 2025 and was **retired in March 2026** — the repository is archived and receives no further releases, bug fixes or security patches — and the intended successor project **InGate never matured and was also retired**. The Kubernetes project's own guidance is to migrate to the **Gateway API**. Because UniVAT has not yet been built, this is a greenfield choice rather than a migration: adopt Gateway API from R0 with Envoy Gateway (Apache-2.0, CNCF) or Traefik v3 (MIT). Separately, **ModSecurity**'s commercial support ended in July 2024 and the engine is now an OWASP community project; **OWASP Coraza** (Go, Apache-2.0, fully SecLang- and CRS-compatible) is the recommended engine for new cloud-native deployments, so existing CRS rule investment carries over unchanged.

### 7.2 Tenancy

Shared-schema multi-tenancy with PostgreSQL **Row-Level Security** keyed on `tenant_id`, plus per-tenant encryption keys for sensitive identifiers (SEC-02); dedicated-tenant and on-prem deployments are the same charts with single-tenant values (TEC-02). Cross-tenant isolation is CI-tested continuously (SEC-04, NFR-18).

### 7.3 The statutory core (identical in all custom choices)

Rules-as-data on the chosen rules engine (ZEN / RulesEngine / Drools): versioned, date-effective JSON decision models with citation, interpreter and approver metadata (GOV-07); determination results persist their rule-version ID (FR-10); golden-master corpus diffed on every build (TEC-25); statutory numbering via Postgres sequence allocation with void-with-reason records (TEC-10, BR-022). All statutory content remains subject to the family's standing **D-01** verification gate regardless of stack.

### 7.4 AI workbench (self-hosted, residency-clean)

Python 3.12 service (the only Python in the stack): PaddleOCR (+ bbOCR-derived Bengali pipeline) → Qwen3.x-VL extraction drafts (human-in-the-loop, AI-20) → pgvector RAG over the versioned legal corpus (AI-07) with citation enforcement (AI-25); forecasting (AI-04/09) in statsmodels/prophet-class OSS; eval harness with held-out golden sets per capability (AI-22), the ≥ 97% invoice-extraction field-accuracy bar and the Bangla-within-5-pp gate (AI-30); every invocation logged (AI-32); kill-switch wired to the feature-flag service (AIC-5, TEC-19); degradation to deterministic paths when GPU services are down (AI-28). Residency is satisfied by construction (AI-31, PDPO §29(7)(b)).

### 7.5 Print & offline

Print service renders gazetted A4 forms (Typst/headless-Chromium PDF) and ESC-POS thermal receipts in Bangla with embedded fonts (Noto Sans Bengali / SolaimanLipi, SIL OFL 1.1) and a conjunct-shaping test corpus (TEC-27, NFR-34, LOC-04). PWA offline: service worker + IndexedDB (Dexie) with a **signed local journal** — HMAC over the device key — synced in issue order with server acknowledgement (TEC-28, UR-025); bounded windows enforced per segment (BR-024).

### 7.6 Disaster-recovery gap to close (D-12)

A single own-DC site cannot meet NFR-09 (99.9% / 99.95%), NFR-11 (RPO ≤ 15 min / RTO ≤ 4 h) or NFR-12 (cross-AZ default). **Minimum remedy**: two failure domains (separate rooms/racks/switches/power) inside the DC for all stateful clusters, plus a contracted second in-country site (co-location or a partner DC) receiving streamed Postgres replicas and Velero backups.

Two things changed in v1.1:

- **The second site is no longer optional infrastructure hygiene — it is a statutory deliverable.** It is the same asset that discharges PDPO §29(7)(b)'s synchronised in-country copy for restricted personal data, with an enforcement horizon of ~**May 2027** (§2.3). It must therefore be operating before R2 closes.
- **Keycloak 26.7 previews multi-cluster HA without external caches**, which removes one of the harder pieces of dual-site identity operations. Track it to GA before depending on it.

NFR-11 requires the failover to be **tested annually**. This document recommends **quarterly Velero restore drills** as an uplift over that floor, on the grounds that an untested DR plan is the most common single cause of missed RTO — but the recommendation is UNISOFT's, not NFR-11's. This remains the single largest delivery risk of the own-DC strategy (tracked as BTO-R1).

### 7.7 Cost model: what "zero licence" does and does not mean

*(New in v1.1. v1.0 asserted a zero-licence-cost posture without ever stating what the strategy does cost — a material gap in a build-strategy document. No BDT figures are given here because none have been estimated; quantification is decision **D-14**, feeding the family's D-02 bottom-up estimate.)*

BC-6's objective is **zero licence cost**, and all six choices meet it. That is a genuine and substantial saving versus a commercial ERP or tax-engine base — but it is not the same as low total cost of ownership, and the difference falls almost entirely on the own-DC strategy rather than on the language choice.

| Cost driver | Nature | Which decision moves it |
|---|---|---|
| Compute, storage and network hardware for two failure domains | Capex, R0 | BC-5 / D-12 — identical across all six choices |
| **GPU nodes** for self-hosted inference (BTO-R8) | Capex + power, R0–R1 | D-06 — self-hosting trades an opex API bill for a capex GPU bill; residency (AI-31, PDPO) largely forces it |
| **Second in-country site** (co-location or partner DC) | Recurring opex from R1/R2 | D-12 — now statutory, not discretionary (§2.3) |
| Power, cooling, rack space, bandwidth, DDoS/transit | Recurring opex | BC-5 — the cost a managed cloud would have bundled into its unit price |
| **Platform-engineering headcount** to run Kubernetes, Postgres, Keycloak, storage and observability | Recurring opex, largest single line | BC-5 — self-hosting substitutes salaried SRE effort for a managed-service fee. PRD §20.3's 30→79 plan must contain it explicitly |
| Upgrade tax on the chosen stack | Recurring engineering capacity | D-11 — negligible for .NET LTS (3-year cadence); moderate for Node (LTS hops); **highest for Spring Boot (~12-month OSS support per minor)** and for Option 1's ERP major versions, which UNISOFT does not control |
| Commercial support subscriptions (optional) | Opex, only if elected | Redis, Grafana, GitLab, Keycloak and PostgreSQL all have paid-support markets. **Electing any of them re-introduces a recurring fee and would qualify BC-6 — it must be an explicit decision, not a procurement reflex** |
| Licence-drift remediation | Unplanned engineering | BTO-R10 — the MassTransit, MinIO, Terraform, Vault and Redis episodes are all from the last 24 months; assume the next one is coming and reserve capacity |

**The honest framing for the CTO:** Option 2 removes licence fees and vendor lock-in; it does not remove cost, it *converts* cost — from a predictable per-seat software fee into capex plus salaried platform engineering. That conversion is almost certainly correct for UniVAT given BC-5, NFR-22, NFR-30 and PDPO residency. But it should be approved with a number attached (D-14), not on the strength of the phrase "zero licence cost".

---

## 8. Development environment and toolchain

Matches the stated UNISOFT working model (BC-1…BC-4). Every developer reproduces the platform locally on Linux.

| Concern | Tooling (all zero-licence) |
|---|---|
| IDE | **VSCode** + Dev Containers; per-stack: C# Dev Kit (OPT2-B), Extension Pack for Java + Gradle (OPT2-C), ESLint/Prettier/vitest (all), SonarLint (live SAST), REST Client, GitLens |
| Local platform | `docker compose` profile: PostgreSQL 18 (+pgvector), Redis/Valkey, RabbitMQ, Keycloak, OpenSearch, SeaweedFS, Mailpit, **mock-eVAT/iVAS + mock-EFDMS** containers; one command boots the world |
| Local build | GitLab CE runner on a shared Linux build box; `gitlab-runner` executors per developer namespace; CI pipelines identical to remote (build → SAST/SCA/secret-scan → SBOM → sign → deploy to dev cluster) |
| Local Kubernetes | k3d/K3s cluster for Helm-chart parity **with the Gateway API CRDs installed** (parity with the production edge, §7.1); ArgoCD `--dest-name local` |
| Error checking & monitoring | ESLint/strict TS; SonarLint; pre-commit hooks (gitleaks, lint, format); local Prometheus+Grafana compose profile with RED/USE dashboards; OTel auto-instrumentation in dev |
| Debugging & preview | **Chrome DevTools**, React Developer Tools, Lighthouse CI (perf budget ≤ 500 KB gzipped Starter payload on a 2 GB Android device, PRD §10.3), axe DevTools (WCAG 2.2 AA), browser preview against local ingress with Bangla locale (bn-BD) |
| Test automation | Playwright E2E incl. Bangla snapshot tests; unit/property tests (vitest / xUnit / JUnit 5); JMeter & Locust load profiles (k6 permitted internally, AGPL — never shipped); statutory-conformance and golden-master suites runnable locally (TEC-24/25) |
| Data & migrations | Versioned SQL migrations (Flyway Community, Apache-2.0, or EF Core migrations); seeders with anonymised v1.0-shaped data (DAT-09) |
| Dependency hygiene | Renovate/Dependabot on a scheduled cadence + **licence allow-list enforced in CI** (§9.6) so a transitive licence change fails the build rather than surfacing at audit time (BTO-R5, BTO-R10) |

---

## 9. Licence and compliance register

Zero-cost posture with risk classes; **counsel validates in-repo licence files before commit (D-13)**; the SBOM pipeline (Syft) gates every release against this register (GOV-10).

**Restructured in v1.1.** v1.0 used a two-way split whose first section was titled *"Permissive core (no obligation on distribution)"* but contained MPL-2.0 (weak copyleft, file-level obligations), LGPL-2.1 (Semgrep) and GPL-2.0 (HAProxy) items with inline caveats — which defeats the purpose of a register that CI is supposed to enforce mechanically. The four classes below are ordered by obligation, and the rule that matters is stated once per class.

### 9.1 Class A — Permissive (no source obligation on distribution)

**Rule:** usable anywhere, including the Enterprise on-prem appliance. Attribution notices must ship.

PostgreSQL 18 & pgvector (PostgreSQL Licence) · SeaweedFS (Apache-2.0) · OpenSearch (Apache-2.0) · Kafka (Apache-2.0, optional) · Temporal (MIT) · Airflow (Apache-2.0) · Keycloak (Apache-2.0) · Superset (Apache-2.0) · ClickHouse (Apache-2.0) · **Valkey (BSD-3-Clause)** · vLLM (Apache-2.0) · Ollama (MIT) · **Qwen3.x incl. Qwen3.x-VL (Apache-2.0)** · DeepSeek (MIT) · PaddleOCR (Apache-2.0) · Tesseract (Apache-2.0) · OpenAI Whisper base models (MIT) · Prometheus (Apache-2.0) · OpenTelemetry (Apache-2.0) · GlitchTip (MIT) · RKE2 / K3s (Apache-2.0) · ArgoCD (Apache-2.0) · cert-manager (Apache-2.0) · **Envoy Gateway (Apache-2.0)** / **Traefik v3 (MIT)** · **OWASP Coraza (Apache-2.0)** · CloudNativePG (Apache-2.0) · Velero (Apache-2.0) · Restic (BSD-2-Clause) · GitLab CE (MIT) · Trivy (Apache-2.0) · Gitleaks (MIT) · Syft (Apache-2.0) · Cosign (Apache-2.0) · Playwright (Apache-2.0) · JMeter (Apache-2.0) · Locust (MIT) · Flyway Community (Apache-2.0) · React / Next.js (MIT) · Dexie (Apache-2.0) · NestJS (MIT) · Fastify (MIT) · Drizzle (MIT) · Prisma (Apache-2.0) · BullMQ (MIT) · **Marten (MIT)** · **Wolverine (MIT)** · EF Core (MIT) · CAP (MIT) · Quartz.NET (Apache-2.0) · NodaTime (Apache-2.0) · Microsoft RulesEngine (MIT) · NRules (MIT) · **GoRules ZEN (MIT)** · jOOQ Open Source Edition (Apache-2.0) · Spring Boot / Spring Modulith (Apache-2.0) · Drools (Apache-2.0) · Debezium (Apache-2.0) · ShedLock (Apache-2.0) · Typst (Apache-2.0) · Mailpit (MIT) · Noto Sans Bengali / SolaimanLipi (SIL OFL 1.1 — embedding permitted; OFL text must ship, fonts must not be sold standalone)

### 9.2 Class B — Weak / file-level copyleft (shippable, with conditions)

**Rule:** may ship in the appliance. Modifications *to these components' own files* must be published; UNISOFT code that merely uses them stays proprietary. Keep them as unmodified upstream artefacts wherever possible — then the obligation is nil.

| Component | Licence | Rule for UNISOFT |
|---|---|---|
| RabbitMQ | MPL-2.0 | Ship unmodified; publish any patches to RabbitMQ's own files |
| OpenTofu | MPL-2.0 | Build-time tool; ship unmodified if bundled |
| OpenBao | MPL-2.0 | Ship unmodified; patches to OpenBao files published |
| Ceph RGW (if adopted) | LGPL-2.1 | Dynamic linking / service boundary keeps UNISOFT code separate |
| Odoo 19 CE (OPT1-A) | LGPL-3.0 | Proprietary modules must remain separable works; on-prem distribution of a modified base requires publishing modifications to the LGPL parts |
| Hangfire (if used) | LGPL-3.0 | Internal use fine; **prefer Quartz.NET (Apache-2.0) in shipped appliances** |
| Semgrep OSS | LGPL-2.1 | CI tool, never linked into or shipped with the product |
| OpenJDK / Temurin (OPT2-C) | GPL-2.0 w/ Classpath Exception | The Classpath Exception is what makes this shippable — application code is not a derivative work |
| HAProxy | GPL-2.0 | Infrastructure binary, run as a separate process; never linked into product code |

### 9.3 Class C — Strong / network copyleft (internal use only — never in a shipped artefact)

**Rule:** self-host inside UNISOFT's own data center for UNISOFT's own operations. **Do not include in the Enterprise on-prem appliance, do not offer as a service to third parties, and do not modify without counsel.** AGPL's network clause means offering a *modified* version over a network triggers source availability to those users.

| Component | Licence | Note |
|---|---|---|
| Grafana, Loki, **Tempo**, Mimir | AGPL-3.0 | *v1.1: Tempo was mis-filed as Apache-2.0 in v1.0's permissive list, contradicting the same document's own AGPL row for Grafana/Loki/Mimir* |
| **Citus** | AGPL-3.0 | *v1.1: mis-filed as Apache-2.0 in v1.0. Usable self-hosted; ClickHouse (Apache-2.0) is now the default warehouse* |
| **k6** | AGPL-3.0 | *v1.1: v1.0 left this as "where licensed terms are confirmed". Confirmed AGPL-3.0 — internal load testing only* |
| Redis 8+ | RSALv2 / SSPLv1 / **AGPLv3** (tri-licensed since May 2025) | **UNISOFT elects AGPLv3** (the OSI-approved option). Unmodified self-hosted use carries no source obligation. For the on-prem appliance, prefer **Valkey (BSD-3)** to avoid the question entirely. *v1.1: v1.0 recorded only RSALv2/SSPLv1* |
| **OCA (Odoo Community Association) modules** | Predominantly **AGPL-3.0**; a minority LGPL-3 | **Decisive for OPT1-A+ ([§4.1.1](#411-variant-opt1-a--odoo-19-ce--oca--free-app-store-modules-maximum-reuse)).** Per the binding OCA–Odoo licence agreement, *AGPLv3 OCA contributions cannot be used as a dependency by any proprietary module*. Any UniVAT module listing an AGPL OCA module in its manifest `depends` becomes AGPL itself, obliging UNISOFT to offer its VAT-engine source to every SaaS user under AGPL §13. **Not usable by proprietary product code.** The LGPL-3 subset is usable (Class B) but small and infrastructure-skewed |
| Garage (rejected) | AGPL-3.0 | Listed for completeness — not adopted |
| MinIO (rejected) | AGPLv3, archived Apr 2026 | Not adopted |

### 9.4 Class D — Non-OSI, source-available or commercial (excluded from the product)

**Rule:** these are **not** zero-licence-cost open source. None may enter the shipped product or the critical path without an explicit, funded decision.

| Component | Licence | Position |
|---|---|---|
| **MassTransit v9** | Commercial (~US$400/month minimum per organisation) | **Excluded (v1.1).** v8 remains Apache-2.0 with security patches only through end-2026. Replaced by Wolverine 6 (MIT) or CAP (MIT) |
| n8n | Sustainable Use Licence (fair-code) | Free to self-host for UNISOFT's internal ops; **not** embeddable or redistributable — substitute Camel/Airflow/Temporal if the on-prem appliance needs workflow glue |
| Odoo App Store free modules | Typically LGPL-3 (OPL-1 is the paid/proprietary licence) | Licence class is usually workable (Class B), but the store performs **no security pre-screening** — independent review/pentest required per module before any production use, so no App Store module may enter the product without a named reviewer |
| Llama-family models | Llama Community License | Source-available; MAU threshold + AUP conditions — **not** the default; Qwen3.x (Apache-2.0) is |
| Sentry (self-hosted) | Functional Source Licence | Non-OSI — GlitchTip (MIT) adopted instead |
| Terraform, Vault | BUSL-1.1 | Superseded by OpenTofu / OpenBao (§9.5) |
| EFD / SDC device SDKs | Proprietary vendor terms | Device procurement item, not an open-source decision (PR-01, BTO-R7) |
| Bengali Whisper fine-tunes | Varies per model | Base Whisper is MIT; **each fine-tune's licence must be cleared individually** before use (D-13) |

### 9.5 Documented deviations from PDR wording (register in the ADR log, GOV-13)

| PDR text | Deviation & rationale |
|---|---|
| TEC-17 "Terraform/Bicep" | → **OpenTofu** (MPL-2.0): Terraform moved to BUSL-1.1; OpenTofu is the Linux-Foundation drop-in |
| TEC-22 "secrets management / central vault" | → **OpenBao** (MPL-2.0): Vault moved to BUSL-1.1 |
| TEC-15 backend examples ".NET 8+/C#, Java 21+, Go" | OPT2-B (.NET 10) and OPT2-C (Java 25) stay inside the list; **OPT2-A (TypeScript) is a documented extension** — decide via D-11 |
| TEC-09 "relational core (PostgreSQL)" | Upheld; recorded here because it is the specific clause that disqualifies OPT1-B2 (ERPNext is MariaDB-bound) |
| AI-31 inference residency | Self-hosted GPU inference in the own DC satisfies residency by construction; hosted LLM APIs only for non-restricted content (AIC-7) |
| *(implicit)* NGINX Ingress Controller | → **Gateway API** (Envoy Gateway / Traefik v3): the ingress-nginx project was retired in March 2026 |

### 9.6 CI enforcement

The register above is the machine-readable allow-list. The pipeline must **fail the build**, not warn, when Syft's SBOM surfaces a component whose licence is not in Class A or Class B, or when a Class B component appears in a *modified* form in an appliance image. Class C components are permitted only in the SaaS deployment manifests, never in the on-prem chart — this is a distinct CI check, because the same Helm repository produces both. Reviewed at every release and re-cleared by counsel annually (D-13).

---

## 10. Delivery mapping

All choices target the same PRD milestones; differences are risk, not sequence.

| Phase (PRD §17.1/§20.2) | Option 1 impact | Option 2 (recommended) impact |
|---|---|---|
| R0 Foundations (mo 1–5): platform, tenancy, ledger, determination, CI/CD | Ledger/determination fight the base's grain — expect R0 overrun or a parallel custom subsystem | OPT2-B: Marten 9 ledger + RulesEngine determination in R0; M3 statutory conformance on schedule. **D-11 closes month 1; D-12 and D-14 close before R0 exit** |
| M2 rule verification (mo 4) | Identical (custom rules content either way; D-01 gates) | Identical |
| R1 MVP + eVAT/iVAS filing (mo 6–11); M5 enlistment | Odoo/Frappe accelerate CRUD screens; **filing/ledger/PVA risk concentrates late** — enlistment demo risk. OPT1-A additionally faces the Odoo 20 upgrade landing mid-R1 | Vertical slices from month 6; mock-eVAT from day one (INT-17); PWA shell from month 6 |
| M4 internal dogfood (mo 9) | Base upgrade coupling may freeze customisation before dogfood | No external upgrade train |
| R2 Intelligence (mo 12–17): reconciliation, copilot, analytics | Same AI workbench bolted on; base's data model hampers lineage (TEC-07/FR-92) | Event-sourced ledger gives lineage natively (US-100 lineage in two clicks). **PDPO §29(7)(b) in-country synchronised copy must be live in this window (~May 2027) — it is a gate, not a stretch goal** |
| R3 Real-time & scale (mo 18–24): EFD/POS, on-prem, ISO 27001 | **On-prem edition collides with GPL bases** (ERPNext variant) or needs LGPL separability work; OFBiz's CVE history complicates the ISO 27001 evidence pack | Same Helm artefacts to customer sites; licence-clean; §9.6 CI check keeps Class C out of the appliance |

Effort shape: Option 1 trades a smaller R0 for a riskier R1 tail and a harder R3; Option 2 is linear. Headcount plan (PRD §20.3: 30→79) is unchanged by stack choice; D-11 only re-balances backend vs frontend ratios. **D-14's TCO baseline does not change the sequence but may change the funding profile of R0**, since the GPU and second-site items are front-loaded.

---

## 11. Risks and open decisions

| ID | Risk | Mitigation |
|---|---|---|
| BTO-R1 | Single-site own-DC cannot meet NFR-09/11/12 | D-12: second failure domain + in-country backup site before R0 exit; quarterly Velero restore drills (uplift over NFR-11's annual floor) |
| BTO-R2 | Team depth mismatch (C# vs TS) | D-11 two-week spike decides OPT2-B vs OPT2-A; training plan (OPS-07) covers the gap; note the ledger-build cost differential in §6.2(2) |
| BTO-R3 | Keycloak SCIM maturity (INT-11) | SCIM API is **preview** as of 26.7 (July 2026), not GA; Metatavu extension behind an abstraction; re-assess at each Keycloak release |
| BTO-R4 | AI quality gates (≥ 97% extraction, Bangla ≤ 5 pp) on self-hosted models | Human-in-the-loop from day one (AI-20); golden-set eval in CI (AI-22); hosted fallback only for non-restricted content (AIC-7) |
| BTO-R5 | Copyleft contamination via transitive dependencies | SBOM gate (Syft) + licence allow-list in CI that **fails** the build (§9.6); D-13 counsel review |
| BTO-R6 | Object-store durability at scale (SeaweedFS) | Benchmark in R0 at UniVAT document volumes; Ceph RGW (erasure coding, rack-level failure domains) as the pre-approved fallback |
| BTO-R7 | eVAT/iVAS and EFDMS specs unpublished (PR-01) | Mock-first anti-corruption zone; manual-filing fallback first-class (unchanged from family) |
| BTO-R8 | GPU capacity/procurement in own DC (D-06/D-12/D-14) | Start with 2× 80-GB-class GPUs for R1 volumes; scale-out plan with vLLM; capex in the D-14 baseline |
| **BTO-R9** | **Edge-layer churn.** ingress-nginx retired and InGate abandoned within a single year; Gateway API implementations are younger than the controller they replace | Adopt Gateway API at R0 (greenfield, no migration debt); keep the Gateway resources vendor-neutral so Envoy Gateway ↔ Traefik is a values-file change; WAF logic in CRS rules (portable between Coraza and ModSecurity), not in controller-specific annotations |
| **BTO-R10** | **Open-source commercialisation drift.** MinIO, Terraform, Vault, Redis, MassTransit and MediatR all changed licence or maintenance posture within ~24 months. Assume at least one more Class A dependency changes terms during the build | Prefer foundation-governed projects (CNCF/ASF/LF) over single-vendor ones for anything load-bearing; keep a named permissive alternative for every single-vendor dependency (documented in §5.0); CI licence-diff alert on every dependency bump; reserve TEC-29's debt budget for remediation |
| **BTO-R11** | **PDPO §29(7)(b) deadline (~May 2027) lands inside R2** and depends on the same asset as D-12 | Treat the in-country synchronised copy as an R2 exit gate with the CISO as owner; evidence pack folded into the ISO 27001 R3 workstream |
| **BTO-R12** | **Version-pin staleness.** Every version in §5.0 is an August-2026 pin; PostgreSQL 19, Odoo 20, .NET 11 and Node 26 all land within months of R0 | Adopt an explicit **n−1 policy** for platform components (upgrade one release behind the newest GA, on LTS lines where they exist); re-baseline §5.0 at each release boundary; never adopt an STS runtime (e.g. .NET 11) for the production line |

Open decisions: **D-11** (spike → OPT2-B vs OPT2-A, CTO, month 1) · **D-12** (own-DC second failure domain + GPU + in-country backup site, CTO/COO, before R0 exit) · **D-13** (licence counsel review incl. AI-model and font licences, CISO/Legal, before first GA ship) · **D-14** (own-DC TCO baseline, CTO/CFO, before R0 exit) — alongside the family's standing **D-01** (rule verification), which gates all statutory content regardless of stack, and **D-02** (bottom-up estimate), which D-14 feeds.

---

## 12. Recommendation summary

**Build UniVAT 2.0 fully custom (Option 2) on the zero-licence commodity layer, with .NET 10 LTS + Marten 9 + Wolverine 6 + PostgreSQL 18 + the TypeScript/React 19/Next.js 16 PWA (OPT2-B) as the primary stack — confirmed or swapped to the all-TypeScript stack (OPT2-A) by a two-week D-11 spike — deployed on RKE2 behind Gateway API and OWASP Coraza in UNISOFT's own data center, with a second failure domain and an in-country replica site that serves both DR (D-12) and the PDPO §29(7)(b) residency deadline of ~May 2027, all licences cleared by counsel against the four-class register in §9 (D-13), and the own-DC total cost quantified before R0 exit (D-14). Option 1's open-source ERP bases (Odoo CE, Frappe/ERPNext, OFBiz) are not recommended for the flagship because the Bangladesh statutory core is custom in every scenario. The maximum-reuse form of the Odoo case — CE plus the OCA catalogue plus free App Store modules — was evaluated on request and scores lowest of all (53.6), because the free accounting catalogue is AGPL-3.0 and AGPL modules cannot be depended upon by proprietary ones: the licence saving would be paid for with the published source of UNISOFT's VAT engine, and Odoo's `Monetary` field is binary floating point in memory, which TEC-11 prohibits. Odoo remains valuable as reference material, as a DAT-02 migration importer, and — as a new commercial opportunity — as a distribution channel via a published "UniVAT Connector for Odoo".**

---

# Appendix A — Glossary

| Term | Meaning |
|---|---|
| AGPL / GPL / LGPL / MPL | Copyleft licence families; obligations differ on modification, linking and network use — §4.0, §9 |
| Anti-corruption layer | Versioned mapping zone isolating external systems (NBR, ERP) from the domain model (INT-17) |
| Bitemporal | Both valid-time and transaction-time recorded, so any historical position is reconstructible (TEC-07) |
| CNPG | CloudNativePG — Kubernetes operator for PostgreSQL clusters |
| Coraza | OWASP's Go-based WAF engine, SecLang/CRS-compatible; the cloud-native successor to ModSecurity |
| Critter Stack | The JasperFx family (Marten, Wolverine, Weasel); cores are MIT, the CritterWatch monitoring product is commercial |
| D-11 … D-14 | New decisions introduced by this document (team spike; own-DC readiness; licence review; TCO baseline) |
| Event sourcing | Persisting state as an append-only event stream; basis of the tax ledger in OPT2-B (Marten) |
| eVAT / iVAS | NBR's Integrated VAT Administration System, rebranded "eVAT"; functions unchanged (§2.3) |
| Fair-code (SUL) | n8n's licence class: free self-host, no embedding/redistribution |
| Gateway API | The Kubernetes successor to Ingress; the ingress-nginx controller was retired in March 2026 |
| Golden-master regression | Frozen scenario corpus; any computed-figure change must be explicitly approved (TEC-25) |
| Marten | .NET library turning PostgreSQL into an event store with projections (MIT) |
| Modular monolith | Single deployable with enforced internal module boundaries (TEC-03) |
| OCA | Odoo Community Association — 250+ repositories, 1,000+ free modules, predominantly **AGPL-3.0**; AGPL modules may not be depended upon by proprietary modules (§4.1.1, §9.3) |
| OPL-1 | Odoo Proprietary License v1.0 — the licence of paid App Store apps; free apps are typically LGPL-3 |
| `restrict_mode_hash_table` | Odoo journal setting enabling SHA-256 hash-chained, gapless, inalterable posted entries |
| Mushak | The gazetted Bangladesh VAT form series; 79 catalogued, 34 generated end-to-end in scope |
| OpenBao / OpenTofu | MPL-2.0 community forks succeeding HashiCorp Vault/Terraform after their BUSL moves |
| PDPO / PDPA | Personal Data Protection Ordinance 2025 / Act 2026 — residency clock, enforcement ~May 2027 (§2.3) |
| Poisha / BDT | Minor unit (¹⁄₁₀₀ taka) and currency; money stored as integer poisha or native decimal |
| RLS | PostgreSQL Row-Level Security — the shared-schema tenant-isolation mechanism |
| RKE2 / K3s | Production-grade / lightweight Kubernetes distributions (Apache-2.0) |
| SBOM | Software Bill of Materials — generated by Syft, gated in CI (SEC-07) |
| Valkey | BSD-3-Clause Redis fork under Linux Foundation governance; the permissive-only cache option |
| vLLM | High-throughput self-hosted LLM inference server (Apache-2.0) |
| Wolverine | MIT .NET messaging/mediator library with a durable transactional outbox; replaces MassTransit here |
| ZEN engine | GoRules' open-source business-rules engine (**MIT**) used for rules-as-data in OPT2-A |

---

# Appendix B — Requirement-coverage matrix (recommended stack)

OPT2-B (primary) against the stack-determining requirements; ✓ = directly satisfied by the stack as designed, ◐ = satisfied with the named measure, ✖ = not met (none).

| Req | Requirement (abridged) | How OPT2-B satisfies it |
|---|---|---|
| TEC-01/05 | API-first, OpenAPI 3.1, versioned, webhooks, ≥ 12-month deprecation policy | Spec-first ASP.NET minimal APIs; OpenAPI contract tests in CI; deprecation policy published with `/v1` |
| TEC-02 | Multi-tenant + on-prem same artefacts | Shared-schema RLS; Helm values per deployment model; §9.6 CI check keeps Class C licences out of the appliance |
| TEC-03 | Modular monolith | Domain assemblies with architecture tests (NetArchTest) in CI |
| TEC-04/06 | Event core, outbox, idempotency | **Wolverine 6** durable outbox over RabbitMQ (CAP as alternative); idempotency-key middleware |
| TEC-07 | Bitemporal append-only ledger | **Marten 9** event store + bitemporal projections; no update path |
| TEC-08/25 | Deterministic recompute; golden master | Pure determination functions + rule-version pinning; corpus diff gate |
| TEC-09 | Polyglot persistence | **PostgreSQL 18** (+pgvector) / SeaweedFS objects / OpenSearch / ClickHouse warehouse |
| TEC-10 | Gapless numbering | Postgres sequence allocation service; void-with-reason records |
| TEC-11 | Decimal-only money | `System.Decimal` + Postgres `numeric`; analyser forbids `double`/`float` in money paths |
| TEC-12 | UTC/BST time | NodaTime (Apache-2.0); tax-period boundaries evaluated in Asia/Dhaka |
| TEC-13/27 | Bangla, Bengali numerals, print fidelity | ICU; embedded OFL Bangla fonts; Typst/Chromium render farm + conjunct test corpus (NFR-34) |
| TEC-14/28 | TS SPA/PWA; signed offline journal | Next.js 16 PWA; Dexie + HMAC journal; bounded windows (BR-024) |
| TEC-15 | Managed strongly-typed backend | .NET 10 LTS — within PDR's stated list; migration off WebForms is total |
| TEC-16/17 | K8s, declarative infra | RKE2 + ArgoCD + OpenTofu (deviation registered §9.5) |
| TEC-18 | DevSecOps pipeline | GitLab CE: build → Semgrep/Trivy/Gitleaks → Syft SBOM → **licence allow-list gate** → Cosign sign → canary |
| TEC-19 | Feature flags per tenant | Flag service wired to AI kill-switch (AIC-5) and regulatory switch-overs |
| TEC-20 | Background jobs, DLQ, replay | Quartz.NET + RabbitMQ DLQ + Temporal sagas |
| TEC-21 | Cache invalidation on rule change | Redis/Valkey cache keyed by rule-version ID |
| TEC-22 | Secrets vault | OpenBao (deviation registered §9.5) |
| TEC-23 | Environments incl. public sandbox | Namespace-per-env + mock-NBR sandbox chart |
| TEC-24 | Test strategy incl. statutory conformance | xUnit + Playwright + JMeter + statutory suite in CI |
| TEC-26 | Extensibility | Tenant custom fields (EAV-jsonb), template engine, workflow rules; scripting sandbox deferred |
| TEC-29/30 | Debt budget; ADRs | 15% capacity reserve (also funds BTO-R10 remediation); ADR repo — this document is ADR-001 |
| NFR-01…08 | Latency/scale budgets | p95 < 300 ms / p99 < 800 ms; load-tested at 15× deadline surge (PRD §14 Large); CNPG read replicas; worker pools |
| NFR-09/11/12 | Availability/DR | Dual failure domain + in-country replica site (D-12); annual live-failover test (NFR-11) plus quarterly restore drills |
| NFR-14…20 | Security baseline | ASVS L2/L3; MFA via Keycloak; isolation tests in CI; audit-log hash chain in the Marten stream |
| **NFR-22 / BRD-012 / C-06 / BR-074** | **In-country residency (PDPO §29(7)(b))** | **Own DC by construction; in-country synchronised replica live before R2 exit, ~May 2027 deadline (§2.3, BTO-R11)** |
| NFR-30 | Portability | 100% self-hostable OSS; no cloud-proprietary services |
| NFR-34 | Bangla font & conjunct rendering | OFL-licensed embedded fonts; shaping validated against the test corpus in UI, PDF and thermal |
| AI-01…32 | AI capability gates | §7.4 workbench: PaddleOCR/bbOCR + Qwen3.x-VL + pgvector RAG + eval gates + kill-switch |
| AI-31 | Inference residency | Self-hosted GPU inference in the own DC; hosted APIs only for non-restricted content (AIC-7) |
| INT-01…17 | Integrations | Integration zone with mock-eVAT/EFDMS; connectors as adapters; ERP file-import fallbacks |
| INT-11 | SAML/OIDC + SCIM | Keycloak 26.7 SCIM (preview) behind an abstraction; Metatavu extension fallback (BTO-R3) |
| LOC-01…10 | Bangla parity | bn-BD first-class in the i18n pipeline; parity gate in CI (X-07) |
| GOV-10 | Clean IP | All shipped code permissive-licensed (Class A/B) or UNISOFT-owned; SBOM + allow-list gate (§9.6) |

---

# Appendix C — Mandatory audit

**Mandatory audit, performed after generation**, in the family style: every check executed mechanically (grep/awk/python scripts against this file and the four parent documents), not by eyeball. **Version 1.1 re-ran the v1.0 checks and added an external-fact verification pass that v1.0 did not perform.**

## C.1 Method

**Structural and referential (v1.0 and re-run for v1.1):** internal anchor resolution (GitHub-style anchors generated from every heading); table pipe-count consistency; TOC↔body correspondence; **independent recomputation of every weighted score** in §4.4, §5.5 and §6.1 from the criterion scores and weights (python); referential integrity of every cited parent ID against the correct parent; weights-sum check.

**External verification (new in v1.1):** every third-party component named in §5.0, §7.1, §8 and §9 was re-checked in August 2026 against primary project sources (project licence files, release announcements, EOL/lifecycle pages) and current reporting for three properties — **licence class, maintenance status, and current version**. Every date, version number and licence string in the document was treated as a claim requiring a source. 21 additional sources were added to Appendix D as a result.

## C.2 Coverage of the brief

| The brief asked for | Where delivered | Status |
|---|---|---|
| Read and analyse all files in `primary_doc` | All four family documents consumed; requirement drivers extracted in §2 with per-ID mapping | ✅ |
| **Option 1: OSS + OSS + custom — 3 best choices** | §4: OPT1-A (Odoo 19 CE), OPT1-B (Frappe/ERPNext, with B1/B2 sub-variants now separately scored), OPT1-C (Apache OFBiz) — each with composition, licences, gaps→custom scope, scoring, verdict | ✅ |
| **Option 2: fully custom — 3 best choices** | §5: OPT2-A (TypeScript end-to-end), OPT2-B (.NET 10 + TS), OPT2-C (Java 25 + Spring Boot + TS) — plus a considered-and-rejected table | ✅ |
| Extensive internet research | v1.0: 10 searches, 24 sources. v1.1: a further 16 targeted searches, 21 new sources, covering every licence and lifecycle claim. **v1.2: a further 12 searches plus 5 primary-source code/manifest reads, 16 new sources, for the OPT1-A+ variant** | ✅ |
| **Evaluate Odoo 19 CE + OCA + free App Store modules (v1.2 request)** | [§4.1.1](#411-variant-opt1-a--odoo-19-ce--oca--free-app-store-modules-maximum-reuse) with composition, three primary findings, six supporting findings, scores, and three constructive Odoo reuse paths; method and evidence in [C.7](#c7-variant-evaluation-added-in-v12-opt1-a) | ✅ |
| Stack must fulfil PRD, BRD and URD comprehensively | Appendix B maps the recommended stack requirement-by-requirement; §2 derives every criterion from family IDs; **§2.3 adds the PDPO driver that v1.0 omitted** | ✅ |
| UNISOFT team / Linux / VSCode / local servers / own-DC deployment | §1.2 binding constraints BC-1…BC-6; §7 own-DC blueprint incl. the DR gap (D-12) and **the cost model (§7.7)**; §8 dev toolchain | ✅ |
| Save at `E:\sofrware_project_5\univat\primary_doc` | `primary_doc\UniVAT_2.0_Build_Strategy_and_Technology_Options.md` | ✅ |
| Comprehensive audit after creation, then revise | This appendix; 23 defects found and fixed in C.5 | ✅ |

## C.3 Structural verification

| Check | Result |
|---|---|
| Internal anchor links | 28 unique; **0 unresolved** (each recomputed under GitHub's heading-anchor algorithm) |
| Markdown tables | 0 ragged (pipe counts consistent in every block) |
| TOC ↔ body | 12 numbered ↔ 12 `##` sections; 4 appendix entries ↔ 4 appendix headings |
| Scoring weights | Sum = 100 (machine-verified) |
| Weighted totals | All ten figures (OPT1-A, OPT1-A+ full, OPT1-A+ LGPL-only, OPT1-B/B1/B2, OPT1-C, OPT2-A/B/C) recomputed independently in python — **all match** |
| Choice IDs | OPT1-A/B/C, OPT2-A/B/C each defined once in a heading, linked from §1.1, scored in their option table, and present in the §6.1 matrix |
| Document size | 1.1 baseline; v1.0 was 683 lines (v1.0's Appendix C reported "560 lines", a stale pre-appendix figure — corrected) |

## C.4 Referential integrity against the family

All **89** distinct parent IDs cited in this document (v1.0 cited 87; v1.1 adds `BRD-012`, `C-06`, `BR-074`, `NFR-34`, `AI-09`, `D-02`) were resolved mechanically against the four parents. **0 missing.** Routing confirmed: `TEC/NFR/INT/DAT/GOV/AI/FR/OPS` → PDR; `LOC/SEC/AIC/BR/US/X/K/PR/C/D/M` → PRD; `BRD-nnn` → BRD v1.1; `UR-nnn` → URD v1.1. Range citations were checked at both bounds (TEC-01…30, AI-01…32, NFR-01…08 / 09…13 / 14…20, INT-01…17, LOC-01…10, SEC-01…15 — all bounds exist). Family counts quoted in §1.1 were verified against the parents themselves: PDR 278 requirements ✓, PRD 19 epics / 118 stories / 54 rules ✓, URD 111 user requirements ✓, "34 Mushak forms generated end-to-end of 79 catalogued" ✓ (PDR §2, G-04).

`D-11`, `D-12`, `D-13` and `D-14` are new decisions introduced and defined by this document (§6.3, §11). Local ID namespaces (`BC-`, `C1…C12` criteria, `BTO-R`, `OPT1/2-x`) are distinct from every family namespace (PRD §6.1 uses `C-01…C-20` with a dash; the criteria here are dash-less and context-labelled).

## C.5 Defects found and corrected

**In v1.0, before first publication (2):**

| # | Defect | Fix |
|---|---|---|
| 1 | The §1.1 link to OPT2-B used a double hyphen where the heading generates a single one | Link corrected; re-verified resolving |
| 2 | §4.4 stated the OPT1-B **B1 sub-variant** score as 62.4; recomputation gives 63.2 | Corrected to 63.2; conclusion unaffected |

**In v1.1, by the external-verification pass (21).** Grouped by severity.

*Critical — a v1.0 instruction would have produced a broken, unsupported or fee-bearing build:*

| # | Defect | Fix |
|---|---|---|
| 3 | **§7.1 specified the NGINX Ingress Controller.** The Kubernetes ingress-nginx project was **retired in March 2026** — archived, no releases, no security patches — and its successor InGate was also retired | Replaced with a **Gateway API** implementation (Envoy Gateway / Traefik v3) across §5.0, §7.1, §8; rejected-table entry added; deviation registered §9.5; new risk BTO-R9 |
| 4 | **§5.2 and §9.1 listed MassTransit as an Apache-2.0 option.** MassTransit **v9 is commercially licensed** (~US$400/month minimum); v8's Apache-2.0 line has guaranteed security patches only to end-2026. Adopting it would have breached BC-6 | Removed from the zero-licence set; replaced by **Wolverine 6 (MIT)** with **CAP (MIT)** as the alternative; moved to Class D (§9.4); new risk BTO-R10 |
| 5 | **§5.2 paired .NET 10 with Marten 7** — a version that predates .NET 10 support. The primary recommendation was internally inconsistent | Updated to **Marten 9** (Critter Stack 2026: JasperFx 2.0 / Weasel 9 / Marten 9 / Wolverine 6, targeting net9.0 + net10.0) |
| 6 | **The PDPO 2025 / PDPA 2026 residency clock was entirely absent**, despite being first-class in the BRD (`BRD-012`, `C-06`), PRD (`BR-074`) and PDR (`NFR-22`) — and its ~May 2027 enforcement date falls inside R2 | New **§2.3**; driver row in §2.1; gate in §10; Appendix B row; new risk BTO-R11; folded into D-12's rationale |

*Licence-register errors — each would have failed a counsel review or a CI licence gate:*

| # | Defect | Fix |
|---|---|---|
| 7 | **Citus recorded as Apache-2.0.** It is **AGPL-3.0** | Moved to Class C (§9.3); **ClickHouse (Apache-2.0)** made the default warehouse |
| 8 | **Grafana Tempo recorded as Apache-2.0** in §9.1 while §9.2 correctly listed Grafana/Loki/Mimir as AGPL — a self-contradiction | Tempo moved to Class C with the rest of the LGTM stack |
| 9 | **Redis 8 recorded as RSALv2/SSPLv1 dual-licensed.** Since May 2025 it is **tri-licensed including AGPLv3**, the OSI-approved option | §5.0 and §9.3 updated; UNISOFT elects AGPLv3; **Valkey (BSD-3)** added as the permissive-only alternative for the appliance |
| 10 | **GoRules ZEN recorded as Apache-2.0.** It is **MIT** | Corrected in §5.1, §5.3, §9.1 and the glossary |
| 11 | **k6 left unresolved** ("where licensed terms are confirmed") | Confirmed **AGPL-3.0**; Class C, internal-only |
| 12 | **§9.1 was titled "Permissive core (no obligation on distribution)" but contained MPL-2.0, LGPL-2.1 and GPL-2.0 components** with inline caveats — unusable as a machine-enforceable allow-list | §9 restructured into four obligation-ordered classes (A permissive / B weak copyleft / C network copyleft / D non-OSI & commercial) with one rule per class, plus **§9.6 CI enforcement** |
| 13 | Blanket claim "Whisper + Bengali fine-tunes (MIT)" | Base Whisper is MIT; fine-tunes carry individual licences — moved to Class D pending per-model clearance under D-13 |
| 14 | Register omitted several shipped components (NodaTime, Dexie, Fastify, NRules, Flyway, Debezium, ShedLock, Mailpit, Restic, cert-manager) and **all Bangla font licences**, despite §7.5 embedding fonts in every printed artefact | All added to Class A; fonts recorded under SIL OFL 1.1 with the notice obligation stated |

*Factual and currency corrections:*

| # | Defect | Fix |
|---|---|---|
| 15 | **PostgreSQL 17** used as the baseline. PG 18 went GA on 25 Sept 2025; PG 19 is due Sept 2026 | Baseline moved to **PostgreSQL 18**, with an explicit n−1 adoption policy (BTO-R12) |
| 16 | MinIO's status given as "maintenance mode (April 2026)" | Corrected: **maintenance mode Dec 2025; repository archived read-only 25 April 2026** |
| 17 | **ERPNext PostgreSQL support described as "experimental/partial."** In fact Frappe Framework supports Postgres but **ERPNext does not** — a hard conflict with TEC-09 | §4.2 rewritten; OPT1-B2 scored explicitly at 59.6 and flagged in §6.1 and §9.5 |
| 18 | OFBiz described only as "actively maintained, active in 2026" | Qualified: current release 24.09.07 (June 2026) but the **24.09 branch has been feature-frozen since Sept 2024** (bug fixes only) |
| 19 | Odoo 19 presented as a stable pin | **Odoo 20 is due 24–26 Sept 2026** — roughly six weeks after this document; a major-version migration inside R0/R1 must be budgeted for OPT1-A |
| 20 | Qwen3 / Keycloak / Spring Boot / Next.js version facts stale or imprecise | Qwen3.x current line noted (3.5 Feb 2026, 3.6 Apr 2026, both Apache-2.0); **Keycloak 26.7** (9 July 2026) with SCIM at **preview** (experimental in 26.6) and multi-cluster HA preview; **Spring Boot 4.1 released 10 June 2026, OSS support to 31 July 2027** — recurring upgrade tax noted in §5.3 and §7.7; **Next.js 16** pinned; **ModSecurity** EOL/OWASP-transfer and **Coraza** successor added |

*Internal-consistency and completeness defects:*

| # | Defect | Fix |
|---|---|---|
| 21 | **OPT1-C scored 6 on C7 (security path)** with no reference to OFBiz's 2024 chain of unauthenticated RCEs (CVE-2024-32113 / -36104 / -38856 / -45195), one of which (**-45195**) entered the **CISA KEV catalogue on 4 Feb 2025** | Score lowered to **4** with the evidence cited in §4.3; OPT1-C total **61.7 → 60.7**; §6.1 and C.3 updated. Ranking unaffected |
| 22 | §2.1 described the Bangla driver as **"RTL-ready"**. Bangla is a **left-to-right** complex script; no family requirement mandates bidirectional layout — the term appears nowhere in the PDR/PRD/BRD/URD | Reworded to complex-script conjunct shaping (TEC-13, NFR-34); the real requirement is glyph composition and font embedding, not bidi |
| 23 | **No cost or TCO content anywhere**, in a document whose stated objective is "zero licence cost" — and "zero licence cost" was never distinguished from "zero cost" | New **§7.7** cost-driver model, new decision **D-14** (own-DC TCO baseline, before R0 exit), feeding the family's D-02; BC-6 cross-referenced. No BDT figures invented — the section is a driver framework, and quantification is explicitly D-14's job |

*Minor fidelity items corrected in passing:* the TEC-15 quotation now reproduces the PDR's actual wording (".NET 8+/C#, Java 21+, or Go" plus the non-negotiable WebForms migration); TEC-05's GraphQL clause and ≥ 12-month deprecation policy restored to the §2.1 driver row; NFR-01's p99 < 800 ms restored; the quarterly restore-drill recommendation is no longer attributed to NFR-11 (which specifies annual testing) but stated as a UNISOFT uplift; "iVAS" annotated as rebranded "eVAT" (§2.3); §1.3 refined to acknowledge the US-federal open tax engines that appeared in 2025–26 while confirming the Bangladesh finding stands.

## C.6 Known limitations

| ID | Limitation | Action |
|---|---|---|
| L-B1 | Criterion scores are expert judgement on researched evidence, not measurements; the ranking is robust (Option 1 ≈ 60–63 vs Option 2 ≈ 84–88) but point differences within an option are within judgement error | Decide OPT2-B vs OPT2-A by the D-11 spike, not by the 3.8-point gap |
| L-B2 | Licences verified from project documentation and August-2026 reporting, not from counsel reading each in-repo licence file. **v1.1 demonstrated the cost of skipping this: six licence/lifecycle facts had drifted in a single document generation** | D-13 before first GA ship; §9.6 CI allow-list enforces continuously; BTO-R10 tracks drift |
| L-B3 | No hands-on benchmark of SeaweedFS/Ceph at UniVAT document volumes | R0 storage benchmark (BTO-R6) |
| L-B4 | EFD/SDC device SDKs and eVAT/iVAS contracts are vendor items outside open source (PR-01); all options assume mock-first | Unchanged from the family position |
| L-B5 | Effort/velocity comparisons (§10) are qualitative, and §7.7's cost model names drivers without quantifying them | D-14 produces the own-DC TCO baseline; feeds the D-02 bottom-up estimate with the chosen stack |
| **L-B6** | **Every version number in §5.0 is an August-2026 pin with a short half-life** — PostgreSQL 19, Odoo 20, .NET 11 and Node 26 all land within months | BTO-R12's n−1 policy; re-baseline §5.0 at each release boundary rather than treating the pins as durable |
| **L-B7** | The NBR eVAT rebrand is reflected in this document only; the rest of the family still says "iVAS" | Naming-alignment pass across the family — editorial, not a stack decision |

## C.7 Variant evaluation added in v1.2 (OPT1-A+)

The business asked whether the recommendation would change if UniVAT were built on **Odoo 19 CE + OCA free modules + free Odoo App Store modules + custom development** — the maximum-reuse form of the zero-licence-cost argument. It was evaluated as a first-class variant, not dismissed: 12 targeted searches, primary-source reads of the OCA–Odoo licence agreement, an OCA module manifest, the Odoo 19 ORM numeric-field source, the Odoo 19 inalterability documentation, and branch-level counts across two OCA repositories.

| Check | Method | Result |
|---|---|---|
| OCA licence class | `OCA/account-financial-tools@19.0/account_move_name_sequence/__manifest__.py` read directly | `"license": "AGPL-3"` — confirms the repository default at module level |
| AGPL dependency rule | OCA–Odoo joint licence statement read directly | *"OCA AGPLv3 contributions … cannot be used as a dependency by any proprietary module"* — the decisive constraint |
| Odoo money representation | `odoo/orm/fields_numeric.py@19.0` read directly | `Monetary._column_type = ('numeric','numeric')` but in-memory type is `Field[float]` via `float(value or 0.0)`; **no `decimal.Decimal`**. `Float` defaults to `float8`/double precision absent explicit digits. Contradicts TEC-11 |
| Odoo immutability | Odoo 19 *Data inalterability check report* documentation | `restrict_mode_hash_table` gives SHA-256 hash chaining, `inalterable_hash`, gapless `secure_sequence_number`, irreversible once posted — **credited**; still not bitemporal (TEC-07/BR-071) |
| OCA v19 porting lag | Branch listings for `OCA/account-financial-tools` | 21 modules on `19.0` vs 41–47 on `18.0` (~45%), 11 months after Odoo 19 GA |
| Score recomputation | python, same weights | OPT1-A+ **53.6**; LGPL-only subset **56.2**; OPT1-A re-scored **57.4**. All independently verified |

**Two v1.1 statements were corrected in UNISOFT's disfavour and two in Odoo's favour**, which is the mark of an evaluation rather than a defence: against the option, the binary-float money model (new, and decisive) and the annual-major/OCA-lag maintenance load; for the option, the shipped hash-chain inalterability and the shipped offline POS PWA, both of which v1.1 had understated. The net movement is downward, and the recommendation is unchanged.

## C.8 Conclusion

v1.0 was structurally sound, arithmetically exact and fully traceable — the audit re-ran every one of its internal checks and confirmed all of them. What v1.0's audit did **not** do was verify its external claims against primary sources, and that is where all 21 new defects were found: four of them critical enough to have produced a broken, unbuildable or fee-bearing platform (a retired ingress controller, a commercially-licensed message bus, a Marten version incompatible with the chosen runtime, and an unmet statutory deadline). The lesson is recorded as L-B2 and BTO-R10: in this ecosystem, *licence and lifecycle facts decay faster than architecture does*, and a build-strategy document must verify them at the component level or it will be confidently wrong.

The corrections did not disturb the conclusion — they strengthened it. Every fix removed a dependency risk from Option 2 (permissive replacements existed in every case), while Option 1 absorbed two new penalties (the ERPNext/PostgreSQL conflict, OFBiz's KEV-listed vulnerability history) and one new cost (upgrade trains UNISOFT does not control). The document is fit for the CTO decision it exists to support: **choose Option 2, confirm OPT2-B vs OPT2-A by the D-11 spike, and fund the own-DC readiness items (D-12), the licence review (D-13) and the TCO baseline (D-14) in month 1.**

---

# Appendix D — Sources

Accessed 15 August 2026. `†` = external to the family. Sources 1–24 supported v1.0; **25–45 were added by the v1.1 verification pass**.

**Open-source ERP bases (Option 1)**

1. † [Odoo 19.0 documentation — Accounting and Invoicing](https://www.odoo.com/documentation/19.0/applications/finance/accounting.html) *(CE ships Invoicing; full Accounting is Enterprise)*
2. † [Odoo — Editions comparison](https://www.odoo.com/page/editions)
3. † [The Ledger Labs — Odoo Community Edition Guide (2026)](https://theledgerlabs.com/odoo-community-edition-guide/) *(LGPL-3.0; CE limits incl. AI Enterprise-only)*
4. † [ERPNext — License and Trademark](https://erpnext.com/license-trademark) *(GPLv3)*
5. † [Frappe Forum — Frappe licensing: framework MIT vs ERPNext GPLv3](https://discuss.frappe.io/t/frappe-licensing-software-architecture/120722)
6. † [Sanskar Technolab — Frappe Framework v16 overview](https://sanskartechnolab.com/frappe-framework/frappe-framework-v16)
7. † [Frappe Forum — PostgreSQL as ERPNext database](https://discuss.frappe.io/t/using-postgresql-as-a-database-for-erpnext/1297)
8. † [The Apache OFBiz Project](https://ofbiz.apache.org/) *(Apache-2.0)*
9. † [Adatasol — Best Open Source ERP Systems in 2026](https://adatasol.com/best-open-source-erp-systems/)
10. † [OpenMercato — Open Source ERP Alternatives in 2026](https://www.openmercato.com/blog-posts/open-source-erp-alternatives)

**Rules engines / tax engines**

11. † [microsoft/RulesEngine](https://github.com/microsoft/RulesEngine) *(MIT, .NET)*
12. † [gorules/zen](https://github.com/gorules/zen) — and [zen/LICENSE](https://github.com/gorules/zen/blob/master/LICENSE) *(**MIT**, corrected in v1.1)*
13. † [NRules](https://github.com/NRules/NRules) *(.NET Rete engine, MIT)*
14. † [kdeldycke/vat-rates](https://github.com/kdeldycke/vat-rates) *(EU rate DB — evidence no BD-class engine exists)*

**Runtimes and platform versions**

15. † [HeroDevs — Spring Boot versions/EOL and latest releases](https://www.herodevs.com/blog-posts/spring-boot-versions-eol-dates-and-latest-releases-april-2026)
16. † [Keycloak — SCIM support survey feedback (Feb 2026)](https://www.keycloak.org/2026/02/scim-support-survey-feedback)

**Storage and infrastructure licences**

17. † [InfoQ — MinIO S3 API alternatives (Dec 2025)](https://www.infoq.com/news/2025/12/minio-s3-api-alternatives/)
18. † [IMTI — Self-hosted S3 with SeaweedFS](https://imti.co/seaweedfs-object-storage/) *(Apache-2.0 alternative)*

**AI models and Bangla AI**

19. † [DataNorth — The best open source LLM in 2026](https://datanorth.ai/blog/the-best-open-source-llm-in-2026) *(Qwen Apache-2.0, DeepSeek MIT, Llama Community License caveats)*
20. † [PaddleOCR (GitHub)](https://github.com/PaddlePaddle/PaddleOCR) *(Apache-2.0; 90+ languages)*
21. † [bbOCR: An Open-source Multi-Domain OCR Pipeline for Bengali Documents (arXiv 2308.10647)](https://ar5iv.labs.arxiv.org/html/2308.10647)
22. † [Spheron — Best open-source OCR/VLM models (2026)](https://www.spheron.network/blog/best-open-source-ocr-vlm-self-host-gpu-cloud-2026/)
23. † [Bangla-WhisperDiar — Fine-tuning Whisper & PyAnnote for Bangla long-form ASR (arXiv, 2026)](https://arxiv.org/html/2605.08214v1)
24. † [HuggingFace — bangla-speech-processing/BanglaASR](https://huggingface.co/bangla-speech-processing/BanglaASR) and [asif00/whisper-bangla](https://huggingface.co/asif00/whisper-bangla)

**Added in v1.1 — licence and lifecycle verification**

25. † [Redis — Redis is now available under the AGPLv3 open source license](https://redis.io/blog/agplv3/) and [Redis — Licenses](https://redis.io/legal/licenses/) *(tri-licence RSALv2 / SSPLv1 / AGPLv3, May 2025)*
26. † [InfoQ — Redis returns to open source with AGPL](https://www.infoq.com/news/2025/05/redis-agpl-license)
27. † [Citus Data — FAQ: Distributed Postgres as an open source extension](https://www.citusdata.com/faq) and [LWN — CitusDB open-sourced](https://lwn.net/Articles/681277/) *(**AGPL-3.0**)*
28. † [Grafana Labs — Grafana, Loki and Tempo will be relicensed to AGPLv3](https://grafana.com/blog/grafana-loki-tempo-relicensing-to-agplv3/) and [Grafana Labs — Licensing](https://grafana.com/licensing/) *(Mimir also AGPLv3)*
29. † [Kubernetes Blog — Ingress NGINX Retirement: What You Need to Know (11 Nov 2025)](https://www.kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/)
30. † [Kubernetes Blog — Ingress NGINX: Statement from the Steering and Security Response Committees (29 Jan 2026)](https://www.kubernetes.io/blog/2026/01/29/ingress-nginx-statement/) *(InGate also retired; migrate to Gateway API)*
31. † [HeroDevs — Ingress NGINX End of Life (March 2026)](https://www.herodevs.com/blog-posts/ingress-nginx-end-of-life-2026-migration-and-support)
32. † [Milan Jovanović — MediatR and MassTransit going commercial](https://milanjovanovic.tech/blog/mediatr-and-masstransit-going-commercial-what-this-means-for-you) and [MassTransit — License configuration](https://masstransit.massient.com/configuration/license) *(v9 commercial; v8 Apache-2.0 patched to end-2026)*
33. † [Jeremy D. Miller — Critter Stack 2026](https://jeremydmiller.com/2026/04/29/critter-stack-2026/) and [JasperFx — Our Products](https://jasperfx.net/our-products/) *(Marten 9 / Wolverine 6 MIT, targeting net9.0 + net10.0; CritterWatch commercial)*
34. † [Marten — Migration guide](https://martendb.io/migration-guide)
35. † [PostgreSQL — PostgreSQL 18 Released! (25 Sept 2025)](https://www.postgresql.org/about/news/postgresql-18-released-3142/) and [PostgreSQL — Versioning policy](https://www.postgresql.org/support/versioning/) *(PG 19 planned Sept 2026)*
36. † [MinIO — Maintenance Mode (issue #21714)](https://github.com/minio/minio/issues/21714) and [Storm Developments — MinIO's community edition is archived (2026)](https://stormdevelopments.ca/blog/minio-s-community-edition-is-archived-what-still-runs-in-2026/) *(maintenance Dec 2025; archived 25 Apr 2026)*
37. † [Pi Stack — Self-hosted S3: Garage vs SeaweedFS vs Ceph RadosGW (May 2026)](https://www.pistack.xyz/posts/2026-05-14-self-hosted-s3-object-storage-garage-vs-seaweedfs-vs-ceph-rgw-guide/) *(SeaweedFS Apache-2.0 and most production-mature; Garage AGPL)*
38. † [Keycloak — 26.7.0 released (9 July 2026)](https://www.keycloak.org/2026/07/keycloak-2670-released) and [Keycloak — SCIM Realm API as an experimental feature (Apr 2026)](https://www.keycloak.org/2026/04/scim-as-experimental-feature)
39. † [Frappe — Version 16 release](https://frappe.io/releases/version-16) and [Frappe Forum — ERPNext/HRMS/Framework v16 release dates](https://discuss.frappe.io/t/erpnext-hrms-frappe-framework-v16-release-dates/156349) *(final 12 Jan 2026)*
40. † [Apache OFBiz — Downloads](https://ofbiz.apache.org/download.html) *(24.09.07, June 2026; 24.09 feature-frozen since Sept 2024)*
41. † [Rapid7 — CVE-2024-45195: Apache OFBiz unauthenticated RCE](https://www.rapid7.com/blog/post/2024/09/05/cve-2024-45195-apache-ofbiz-unauthenticated-remote-code-execution-fixed/) and [CSA Singapore — Active exploitation of critical Apache OFBiz vulnerabilities](https://www.csa.gov.sg/alerts-and-advisories/alerts/al-2024-100/) *(CVE-2024-45195 added to CISA KEV 4 Feb 2025)*
42. † [F5 — NGINX ModSecurity WAF transitioning to end-of-life](https://www.f5.com/company/blog/nginx/f5-nginx-modsecurity-waf-transitioning-to-eol) and [OWASP CRS — Welcome ModSecurity to the OWASP family](https://coreruleset.org/20240115/welcome-the-newest-addition-to-the-owasp-family-modsecurity/) *(Coraza the cloud-native successor)*
43. † [Grafana k6 — LICENSE.md](https://github.com/grafana/k6/blob/master/LICENSE.md) *(**AGPL-3.0**)*
44. † [SCL Insights — The Personal Data Protection Ordinance 2025: what every Bangladeshi business must do](https://bd-scl.com/insights/personal-data-protection-ordinance-2025-compliance.html) and [Securiti — Overview of Bangladesh's Personal Data Protection Act 2026](https://securiti.ai/bangladesh-personal-data-protection-act-overview/) *(gazetted 6 Nov 2025; §29(7)(b) amended Mar 2026 to restricted personal data + CII; ~May 2027 enforcement)*
45. † [The Business Standard — All online VAT activities to be conducted through the eVAT system: NBR](https://www.tbsnews.net/bangladesh/all-online-vat-related-activities-be-conducted-thru-evat-system-nbr-1317306) and [Mushak 6.3 VAT invoice Bangladesh — 2026 compliance guide](https://rakibhassan.eu/mushak-6-3-vat-invoice-bangladesh/) *(iVAS → eVAT rebrand; Mushak 6.3 / 9.1 obligations; EFD/SDC rollout)*
46. † [.NET — Official support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) and [dotnet/core releases](https://github.com/dotnet/core/blob/main/releases.md) *(.NET 10 LTS to 14 Nov 2028; .NET 11 STS Nov 2026)*
47. † [Node.js — End-of-life](https://nodejs.org/en/about/eol) *(Node 24 Active LTS, EOL 30 Apr 2028; Node 26 enters LTS Oct 2026)*
48. † [Spring — Spring Boot 4.1.0 available now (10 June 2026)](https://spring.io/blog/2026/06/10/spring-boot-4/) and [InfoQ — Spring Boot 4.1 adds gRPC auto-configuration](https://www.infoq.com/news/2026/06/spring-boot-4-1/) *(OSS support to 31 July 2027)*
49. † [Qwen3-VL — open vision-language model (Apache 2.0)](https://freeapihub.com/ai-models/qwen3-vl) and [Qwen 3.6 series (2026)](https://aimlapi.com/blog/qwen-3-6-series-alibabas-open-source-llm-revolution-in-2026) *(Apache-2.0 across the line)*
50. † [filedcom/opentax](https://github.com/filedcom/opentax) and [Invaro/opentax-engine](https://github.com/Invaro/opentax-engine) *(US-federal open tax engines, 2025–26 — evidence that the Bangladesh gap remains unfilled)*

**Added in v1.2 — OPT1-A+ variant evaluation (§4.1.1)**

52. † [OCA — OCA and Odoo reach license agreement: AGPLv3 & LGPL](https://www.odoo-community.org/blog/news-updates-1/oca-odoo-meeting-on-licenses-21) *(the binding rule: AGPLv3 OCA contributions cannot be a dependency of any proprietary module)*
53. † [OCA — FAQ: licence policy](https://www.odoo-community.org/resources/faq) and [OCA — Odoo relicensing to LGPLv3: OCA's official position](https://www.odoo-community.org/blog/news-updates-1/odoo-relicensing-11)
54. † [OCA/account-financial-tools — `19.0` branch](https://github.com/OCA/account-financial-tools/tree/19.0) and [`18.0` branch](https://github.com/OCA/account-financial-tools/tree/18.0) *(21 vs 41–47 modules — the v19 porting lag)*
55. † [OCA/account-financial-tools — `account_move_name_sequence/__manifest__.py` (19.0)](https://raw.githubusercontent.com/OCA/account-financial-tools/19.0/account_move_name_sequence/__manifest__.py) *(`"license": "AGPL-3"`)*
56. † [odoo/odoo — `odoo/orm/fields_numeric.py` (branch 19.0)](https://github.com/odoo/odoo/blob/19.0/odoo/orm/fields_numeric.py) *(`Monetary` stored `numeric` but held in memory as Python `float`; `Float` defaults to `float8`)*
57. † [Odoo 19 documentation — Data inalterability check report](https://www.odoo.com/documentation/19.0/applications/finance/accounting/reporting/data_inalterability.html) *(SHA-256 hash chain, `secure_sequence_number`, `restrict_mode_hash_table`)*
58. † [Odoo 19 documentation — Licenses](https://www.odoo.com/documentation/19.0/legal/licenses.html) and [Odoo Apps vendor guidelines](https://apps.odoo.com/apps/vendor-guidelines) *(LGPL-3 for free apps, OPL-1 for paid)*
59. † [Oduist — Odoo Security 101 / most common security issues (Odoo Experience 2025)](https://oduist.com/blog/odoo-experience-2025-ai-summaries-2/178-odoo-security-101-180) *(App Store is not security pre-screened; pentest third-party modules)*
60. † [Odoo 19 documentation — External JSON-2 API](https://www.odoo.com/documentation/19.0/developer/reference/external_api.html) and [Oduist — XMLRPC is dead, all hail JSON-2](https://oduist.com/blog/odoo-experience-2025-ai-summaries-2/286-xmlrpc-is-dead-all-hail-json-2-288) *(XML-RPC/JSON-RPC removal in Odoo 22, autumn 2028)*
61. † [Novobi — The IT leader's guide to Odoo versions and upgrade cadence](https://novobi.com/the-it-leaders-guide-to-odoo-version-releases-determining-your-upgrade-cadence/) and [Geminate — Odoo migration cost factors](https://www.geminatecs.com/blog/odoo-migration-cost-factors) *(custom-module count is the dominant migration cost driver)*
62. † [DeployMonkey — Odoo POS offline mode: how it works (2026)](https://deploymonkey.com/blog/odoo-pos-offline-mode-guide) and [SDLC Corp — Odoo 18 Point of Sale PWA](https://sdlccorp.com/post/odoo-18-point-of-sale-pwa/) *(IndexedDB offline sync; unsynced orders lost on cache clear)*
63. † [Odoo Trailblazer — Is Odoo scalable?](https://www.odootrailblazer.com/p/is-odoo-scalable) and [OCA discussion — What's considered a "large" Odoo database](https://github.com/orgs/OCA/discussions/81) *(1B+ bookings at one client; ORM materialisation cost)*
64. † [OEC.sh — Odoo multi-tenant architecture guide](https://oec.sh/blog/odoo-multi-tenant-architecture) and [PingCAP — 3 tips to scale multi-tenant SaaS data](https://www.pingcap.com/blog/3-tips-scale-multi-tenant-saas-data-without-pain/) *(database-per-tenant sprawl; RDBMS not designed for tens of thousands of databases)*
65. † [Metamorphosis — Odoo localization for Bangladesh](https://www.metamorphosis.com.bd/blog/industry-9/odoo-localization-for-bangladesh-89) and [Metamorphosis — Odoo vs ERPNext for Bangladeshi businesses (2026)](https://metamorphosis.com.bd/insights/odoo-vs-erpnext-bangladesh) *(`l10n_bd` scope; neither ships deep BD compliance)*
66. † [SmartVAT — Top NBR-approved VAT software companies in Bangladesh 2026](https://smartvat.com.bd/blog/top-10-nbr-approved-vat-software-bangladesh-2026/) and [iBOS — NBR-approved top 10 VAT software](https://ibos.io/top-10-vat-software-in-bangladesh/) *(no Odoo-derived product among the NBR-enlisted vendors)*
67. † [The Ledger Labs — Odoo Community Edition: features, limits & cost (2026)](https://theledgerlabs.com/odoo-community-edition-guide/) and [OEC.sh — Odoo Community vs Enterprise: what you actually need in 2026](https://oec.sh/blog/odoo-community-vs-enterprise) *(CE accounting boundaries; AI Enterprise-only in Odoo 19)*

**Internal**

68. `UniVAT_2.0_BRD.md` (`USL-UNIVAT2-BRD-001` v1.1) · `UniVAT_2.0_URD.md` (`USL-UNIVAT2-URD-001` v1.1) · `UniVAT_2.0_Product_Definition_and_Requirements.md` (`USL-UNIVAT2-PDR-001`) · `UniVAT_2.0_PRD.md` (`USL-UNIVAT2-PRD-001`) — same folder.

---

*End of document — UniVAT 2.0 Build Strategy & Technology Options v1.1*
