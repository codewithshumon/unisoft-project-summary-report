# UniVAT 2.0 — Technology Stack Evaluation, Comparison & Recommendations

> **Document Context:** Synthesised directly from the authoritative specifications in `/home/shumon/Music/univat/project-docs/primary_doc/`, specifically:
> - `UniVAT_2.0_Build_Strategy_and_Technology_Options.md` (`USL-UNIVAT2-BTO-001` v1.2)
> - `UniVAT_2.0_Technology_Stack.md` (`USL-UNIVAT2-TSD-001` v1.1)
> - Traced against `UniVAT_2.0_BRD.md`, `UniVAT_2.0_URD.md`, `UniVAT_2.0_PRD.md`, and `UniVAT_2.0_Product_Definition_and_Requirements.md` (PDR).

---

## 1. Executive Summary & Governing Verdict

Across the primary evaluation documents, seven candidate build strategies were scored across two overarching categories:
1. **Option 1 — Open-Source ERP Base Strategies** (Odoo 19, Frappe/ERPNext, Apache OFBiz).
2. **Option 2 — Fully Custom Development on a Zero-Licence Commodity Layer** (TypeScript end-to-end, .NET 10 LTS + TypeScript, Java + TypeScript).

### The Recommendations:
* 🏆 **Primary Recommended Stack: Choice OPT2-B (Score: 88.1 / 100)**
  * **Core Architecture:** .NET 10 LTS / C# 14 Modular Monolith (`UniVat.App`) + Marten 9 (PostgreSQL 18 bitemporal event-sourced ledger) + Wolverine 6 (transactional outbox) + Next.js 16 / React 19 / TypeScript 6 PWA frontend + RKE2 Kubernetes in UNISOFT's own private data center.
  * **Key Justification:** Solves the two hardest statutory requirements in the project without architectural compromise: **TEC-11** (first-class, native `System.Decimal` money arithmetic — strictly eliminating binary floating-point errors) and **TEC-07** (production-grade append-only, bitemporal audit ledger via Marten 9), while maintaining a 100% permissive/clean licence posture (Class A/B).

* 🥈 **Fallback / Co-Finalist: Choice OPT2-A (Score: 84.3 / 100)**
  * **Core Architecture:** TypeScript end-to-end (Node.js v24 LTS / Fastify + Next.js 16 + PostgreSQL 18 + Drizzle ORM + ZEN Rules Engine).
  * **Role:** Designated fallback if team hiring or technical spike (`D-11`) determines the engineering team lacks C# depth. Requires strict runtime discipline (`decimal.js` + integer poisha storage) and hand-building the bitemporal event store.

* ❌ **Option 1 Rejected for Flagship Core (Scores: 53.6 – 63.2 / 100)**
  * Adopting an open-source ERP base (Odoo, ERPNext, OFBiz) was rejected because the statutory Bangladesh VAT core must be custom-built regardless, and the base platforms introduce severe licence infection risks (AGPL-3.0), multi-tenancy operational drag (database-per-tenant), floating-point money precision defects, and unaligned upstream upgrade cadences.

---

## 2. Comparative Evaluation Matrix (All Evaluated Stacks)

| Dimension / Metric | OPT1-A (Odoo 19 CE) | OPT1-A+ (Odoo 19 + OCA / AppStore) | OPT1-B (Frappe / ERPNext) | OPT1-C (Apache OFBiz) | OPT2-A (TypeScript End-to-End) | 🏆 OPT2-B (.NET 10 + TypeScript) | OPT2-C (Java 21/25 + TypeScript) |
|---|---|---|---|---|---|---|---|
| **Overall Score** | **57.4** | **53.6** *(Lowest)* | **60.8** (B1: 63.2 / B2: 59.6) | **60.7** | **84.3** *(Co-Finalist)* | **88.1** *(Winner)* | **85.1** |
| **Licence Posture** | LGPL-3.0 base | **LGPL base + AGPL modules (Forces proprietary source disclosure)** | MIT base (GPL-3.0 if ERPNext used) | Apache-2.0 base | All Permissive (MIT / Apache-2.0) | **All Permissive; core libs MIT / Apache-2.0** | OpenJDK GPL-CPE + Apache-2.0 |
| **TEC-11 Decimal Money** | ❌ **Fails** (ORM holds Python `float`) | ❌ **Fails** (ORM holds Python `float`) | Custom Python decimal wrapper | `BigDecimal` available | High discipline (`decimal.js` + integer poisha) | ✅ **Native `System.Decimal` (First-Class)** | ✅ Native `BigDecimal` |
| **TEC-07 Event Ledger** | Partial (`restrict_mode_ hash_table`) | Partial | High custom engineering | Custom relational engine | Hand-built event store | ✅ **Native Marten 9 (PostgreSQL Event Store)** | Hand-built event store (Axon/custom) |
| **Statutory Core Risk** | High (custom core + ERP friction) | High | High | Medium | Medium (decimal discipline needed) | ✅ **Low (native decimal + event store)** | Low |
| **Multi-Tenancy (10k Tenants)** | DB-per-tenant drag | DB-per-tenant drag | Site-per-tenant drag | Framework-managed | Shared-schema PostgreSQL RLS | ✅ **Shared-schema PostgreSQL RLS** | Shared-schema PostgreSQL RLS |
| **On-Prem Appliance** | LGPL separability work needed | ❌ **Infected** (Must publish engine source) | Clean if Frappe-only (B1) | Clean | Clean | ✅ **Clean** | Clean |
| **TEC-09 PostgreSQL Core** | Native | Native | ❌ **Conflicts** (ERPNext bound to MariaDB) | Native | Native | ✅ **Native PostgreSQL 18** | Native |
| **Upgrade Control** | No (Odoo S.A. cadence) | No (OCA porting lag ~45% at 11 mo.) | No (Frappe major version hops) | Frozen branch | Yes (Controlled) | ✅ **Yes (.NET LTS 3-year cadence)** | Partly (Spring minors ~12-mo support) |

---

## 3. Detailed Review of Candidate Stacks

### 3.1 Option 1: Open-Source Base Strategies

#### 1. OPT1-A: Odoo 19 Community Edition + Custom Bangladesh VAT Modules
* **Architecture:** Python 3.10+, Odoo ORM, PostgreSQL, QWeb/OWL front-end.
* **Score:** 57.4 / 100
* **Disqualifiers & Risks:**
  * **Precision Violation:** Odoo's `fields.Monetary` stores monetary values in memory as standard Python `float` (binary floating-point), directly violating requirement `TEC-11`.
  * **Database-per-tenant:** Heavy operational drag to host 10,000+ tenants compared to shared-schema Row-Level Security (RLS).
  * **Upstream Friction:** Major release changes (e.g. Odoo 19 → 20) are controlled by Odoo S.A., requiring extensive recurring migration effort.

#### 2. OPT1-A+: Odoo 19 CE + OCA Accounting + Free App Store Modules (Maximum Reuse Variant)
* **Architecture:** Evaluated at business request to maximize free modules and pre-built accounting features.
* **Score:** 53.6 / 100 *(Lowest score of all evaluated options)*
* **Disqualifiers & Risks:**
  * **Licence Infection (AGPL-3.0):** The free OCA accounting catalogue is predominantly AGPL-3.0. The binding Odoo/OCA licensing rules prohibit proprietary commercial modules from depending on AGPL modules without publishing the entire derivative source code. Choosing this variant would legally force UNISOFT to publish its core Bangladesh VAT engine.
  * **OCA Lag:** Porting OCA modules across major Odoo versions experiences ~45% unported lag after 11 months.

#### 3. OPT1-B: Frappe Framework (+ ERPNext) + Custom UniVAT App
* **Architecture:** Python, MariaDB/PostgreSQL, Frappe REST API, Desk UI.
* **Score:** 60.8 / 100 (Frappe-only B1: 63.2; ERPNext B2: 59.6)
* **Disqualifiers & Risks:**
  * **Database Conflict:** ERPNext is fundamentally tightly coupled to MariaDB, violating requirement `TEC-09` (relational core must be PostgreSQL).
  * **GPL-3.0 Risk:** Adopting ERPNext pulls in GPL-3.0 copyleft obligations.

#### 4. OPT1-C: Apache OFBiz + Custom VAT Domain
* **Architecture:** Java, Apache OFBiz framework, Entity Engine, Service Engine.
* **Score:** 60.7 / 100
* **Disqualifiers & Risks:**
  * Antiquated user interface, complex architectural conventions, low developer availability in Bangladesh, and very slow time-to-market.

---

### 3.2 Option 2: Fully Custom Development Strategies

#### 1. 🏆 OPT2-B: .NET 10 LTS Modular Monolith + TypeScript Frontend (PRIMARY RECOMMENDED)
* **Score:** 88.1 / 100
* **Key Components:**
  * **Language & Runtime:** .NET 10 LTS / C# 14.
  * **Architecture:** Modular Monolith (`UniVat.App`) partitioned into 12 bounded contexts across 4 tiers; boundaries mechanically enforced by `NetArchTest`.
  * **Event Ledger (`TEC-07`):** Marten 9 on PostgreSQL 18 (bitemporal, append-only, projection support).
  * **Outbox Pattern (`TEC-06`):** Wolverine 6 (transactional outbox with RabbitMQ, zero dual-writes).
  * **Financial Math (`TEC-11`):** Native `System.Decimal` across all models and calculations.
  * **Frontend:** Next.js 16 (App Router) + React 19 + TypeScript 6 + Tailwind CSS v4 + Radix UI / shadcn/ui.
  * **Licencing:** Permissive (MIT / Apache-2.0 / BSD-3); zero copyleft infection.

#### 2. 🥈 OPT2-A: TypeScript End-to-End (CO-FINALIST / FALLBACK)
* **Score:** 84.3 / 100
* **Key Components:** Node.js v24 LTS / Fastify + TypeScript + Drizzle/Kysely ORM + PostgreSQL 18 + Next.js 16 PWA + ZEN Business Rules Engine (MIT).
* **Strengths:** Unified language across entire stack; large Bangladesh hiring pool.
* **Compromises:** JavaScript/TypeScript lacks native decimal numbers (requires strict discipline using `decimal.js` and storing monetary values as integer poisha); requires writing a custom bitemporal event store from scratch.

#### 3. OPT2-C: Java 21/25 LTS + Spring Boot 3.x + TypeScript Frontend
* **Score:** 85.1 / 100
* **Key Components:** OpenJDK 21/25 LTS, Spring Boot 3.x, Spring Data / jOOQ, `BigDecimal`, PostgreSQL 18, Next.js 16 PWA.
* **Strengths:** Proven enterprise engine; native `BigDecimal`.
* **Compromises:** Spring Boot minor release upgrade tax (~12–13 months open-source support per minor, risking maintenance drag or costly commercial support); heavier JVM memory and GC overhead.

---

## 4. The Mandated 18-Layer Component Blueprint (Stack OPT2-B)

The primary documents establish an explicit, layer-by-layer inventory of ≈110 named components:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ CLIENT LAYER                                                                 │
│  Next.js 16 PWA (React 19, TypeScript 6)  ·  Tailwind CSS v4 / shadcn / Radix│
│  Bilingual i18n (Bangla-first, English equal)  ·  Counter/POS offline shell │
│  decimal.js client precision  ·  Scalar API interactive reference            │
└───────────────────────────────┬──────────────────────────────────────────────┘
                                │ HTTPS · OAuth 2.1 / Scoped API Keys · /v1
┌───────────────────────────────▼──────────────────────────────────────────────┐
│ EDGE / INGRESS (RKE2)                                                         │
│  Gateway API (Envoy Gateway)  ·  OWASP Coraza WAF (SecLang/CRS)              │
│  cert-manager (automated TLS) ·  Per-tenant rate limiting                    │
└───────────────────────────────┬──────────────────────────────────────────────┘
┌───────────────────────────────▼──────────────────────────────────────────────┐
│ CORE ENGINE (UniVat.App — .NET 10 LTS Modular Monolith)                      │
│                                                                              │
│  Tier 0: UniVat.Platform        (Tenancy RLS, Decimal Money, Clock, Audit)   │
│  Tier 1: Identity & Masters     (Users, Roles, Parties, HS Codes, Items)     │
│  Tier 2: Domain Core            (Determination/ZEN, Ledger, Inventory, Docs) │
│  Tier 3: Application Services   (Purchases, Filing, Reconcile, AI Gateway)   │
│                                                                              │
│  Cross-cutting: Native System.Decimal  ·  NetArchTest boundary enforcement  │
└───┬────────────┬─────────────┬─────────────┬──────────────┬──────────────────┘
    │            │             │             │              │
┌───▼────┐ ┌─────▼──────┐ ┌────▼─────┐ ┌─────▼──────┐ ┌─────▼─────────────────┐
│Postgre-│ │ RabbitMQ   │ │ Valkey   │ │ SeaweedFS  │ │ OpenSearch  ClickHouse│
│SQL 18  │ │ (Wolverine │ │ (Hybrid- │ │ (S3 objects│ │ (search +   (analytics│
│+pgvector│ │  outbox)   │ │  Cache)  │ │  storage)  │ │  Bengali)   warehouse)│
│CloudNativePG│         │ │          │ │            │ │                       │
└────────┘ └────────────┘ └──────────┘ └────────────┘ └───────────────────────┘
    │            │
┌───▼────────────▼──────────────────────────────────────────────────────────────┐
│ PLATFORM & INTEGRATION SUBSYSTEMS                                             │
│  Identity / Access: Keycloak 26.7 (OIDC / SAML / WebAuthn FIDO2 passkeys)      │
│  Secrets: OpenBao (MPL-2.0 Vault fork) · Feature Flags: flagd / OpenFeature    │
│  Workflows & Sagas: Temporal · Scheduled Tasks: Quartz.NET                    │
│  Print / Render Farm: Typst (Apache-2.0) + Headless Chromium + ESC/POS Raster │
│  AI Workbench: Python 3.14 + FastAPI + vLLM (Qwen3.x-VL) + PaddleOCR (Own GPU)│
│  Integration ACL: Mock services for eVAT/iVAS, a-Challan, EFDMS, ERP adapters │
└────────────────────────────────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────────────────────────────────┐
│ OBSERVABILITY & SUPPLY CHAIN                                                   │
│  Telemetry: OpenTelemetry (OTel) → Prometheus + Grafana (Loki / Tempo / Mimir) │
│  CI/CD: GitLab CE (26 quality gates) · SBOM & Supply Chain: Syft + Cosign     │
│  GitOps & IaC: ArgoCD + OpenTofu (Terraform fork) + Ansible                   │
│  Infrastructure: Bare-metal RKE2 Kubernetes (UNISOFT Private Data Center)      │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Prohibited, Rejected & Replaced Technologies

The primary documents establish an explicit prohibited technology list (§12 of `UniVAT_2.0_Technology_Stack.md` and §5.4 of `UniVAT_2.0_Build_Strategy_and_Technology_Options.md`):

| Technology | Status | Reason for Rejection | Mandated Replacement |
|---|---|---|---|
| **EF Core in Ledger** | ❌ **Strictly Forbidden** | Change-tracking and lazy loading break append-only (`TEC-07`) and determinism (`TEC-08`). | **Marten 9** for event store; EF Core restricted to static masters only. |
| **`double` / `float` for Money** | ❌ **Compile Error** | Binary floating point causes rounding errors violating statutory law (`TEC-11`). | **`System.Decimal`** (.NET) / `decimal.js` (Frontend). |
| **Microservices-First** | ❌ **Rejected** | Violates `TEC-03`. Causes distributed transaction and operational overhead. | **Modular Monolith** with compile-time boundary enforcement. |
| **ingress-nginx** | ❌ **Forbidden** | Kubernetes project officially retired/unmaintained in March 2026. | **Gateway API** (Envoy Gateway / Traefik v3). |
| **HashiCorp Vault / Terraform** | ❌ **Forbidden** | Shifted to non-open source BUSL-1.1 license. | **OpenBao** (MPL-2.0) and **OpenTofu** (MPL-2.0). |
| **MinIO** | ❌ **Forbidden** | Community edition archived/read-only April 2026; AGPLv3 copyleft risk. | **SeaweedFS** (Apache-2.0) or Ceph RGW. |
| **MediatR / AutoMapper** | ❌ **Forbidden** | Moved to commercial licensing ($5M revenue threshold) violating zero-cost rule `BC-6`. | **Wolverine** (MIT) and **Riok.Mapperly** (Apache-2.0). |
| **MassTransit** | ❌ **Forbidden** | v9 is commercial ($400/month min); v8 Apache line unsupported after 2026. | **Wolverine 6** (MIT) or dotnetcore/CAP (MIT). |
| **QuestPDF** | ❌ **Forbidden** | Non-OSI commercial license, excludes public-sector usage. | **Typst** (Apache-2.0) + Headless Chromium. |
| **Citus** | ❌ **Excluded from Appliance**| AGPL-3.0 license prevents shipping in on-premises commercial appliance. | **ClickHouse** (Apache-2.0) as default warehouse. |
| **MongoDB / NoSQL Ledger** | ❌ **Rejected** | Lacks relational transactional integrity and gapless numbering (`TEC-09`, `TEC-10`). | **PostgreSQL 18** with sequence generators. |
| **Public Cloud Proprietary APIs** | ❌ **Rejected** | Violates data sovereignty (`BC-5`) and statutory data residency (PDPO §29(7)(b)). | **Self-hosted RKE2 on private bare-metal DC**. |
