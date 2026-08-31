prd_content = """# Product Requirements Document (PRD)

**Product Name:** Enterprise License AI Copilot (`enterprise-license-ai-copilot`)  
**Document Version:** 1.0.0  
**Status:** Approved / In Development  
**Target Release:** Q4 2026  
**Document Owner:** AI Product Lead  
**Stakeholders:** IT Asset Management (ITAM/SAM), FinOps, Software Procurement, Legal, Enterprise Architecture  

---

## 1. Executive Summary & Problem Definition

### 1.1 Executive Summary
Large enterprise organizations spend tens of millions of dollars annually on software licenses and SaaS subscriptions across hundreds of vendors (e.g., Microsoft, Oracle, Salesforce, GitHub, Adobe). However, software licensing contracts (Master Service Agreements, Product Addendums, EULAs) are legally complex, dense, and unstructured documents with varied licensing metrics (per-core, per-named-user, capacity-based, concurrent). Concurrently, actual usage telemetry resides in siloed enterprise systems (ServiceNow CMDB, Entra ID, Okta SSO, ERPs, cloud billing consoles).

The **Enterprise License AI Copilot** is an agentic, hybrid AI/deterministic intelligence platform that ingests unstructured contract PDFs alongside real-time usage data to automate license entitlement reconciliation, detect compliance true-up risks, uncover unallocated shelfware, and formulate executive-ready renewal negotiation briefs.

### 1.2 Problem Statement & Market Friction
* **High Financial Exposure:** Unplanned vendor true-up audits result in average enterprise penalty assessments exceeding $250,000 per audit cycle due to unintentional over-deployment.
* **Wasted Software Spend (Shelfware):** 20% to 30% of enterprise SaaS licenses sit dormant (>60/90 days inactive) due to poor deprovisioning visibility.
* **Manual Bottleneck:** ITAM and Procurement analysts spend 40–80 hours per Tier-1 vendor renewal manually combing through 50+ page legal contracts, matching clauses against fragmented spreadsheets.
* **Asymmetric Vendor Negotiations:** Enterprise procurement teams enter renewal discussions without data-backed utilization models, conceding unearned 5–10% annual price escalations.

### 1.3 Value Proposition & ROI Targets
* **80% Reduction** in manual contract analysis and license reconciliation time.
* **100% Elimination** of arithmetic hallucinations via a hybrid architecture separating LLM clause extraction from deterministic calculation logic.
* **15–25% Direct Cost Savings** on addressable SaaS renewals through automated dormant license reclamation and tier down-grading recommendations.
* **Audit Defense Readiness:** Instant audit position generation with full clause-level source provenance.

---

## 2. Target Personas & Jobs-to-be-Done (JTBD)

### 2.1 Persona Matrix

| Persona | Role & Responsibilities | Core Pain Points | Key JTBD |
| :--- | :--- | :--- | :--- |
| **P1: IT Asset Manager (SAM Lead)** | Manages Effective License Position (ELP), audits, and entitlement compliance. | Ambiguous contract clauses (e.g., virtualization rules, multiplexing, failover rights); multi-spreadsheet sprawl. | *"When an audit notice arrives, I want an instant, validated Effective License Position with exact contract clause citations so I can defend our compliance position without manual spreadsheet reconciliation."* |
| **P2: Procurement Director** | Negotiates vendor MSAs, pricing tiers, renewal commitments, and discount schedules. | Lack of usage leverage during renewals; auto-renewal notice deadlines missed in contract fine print. | *"When a major contract renewal is 90 days out, I want an executive negotiation brief detailing actual utilization vs. purchased seats and specific down-tiering recommendations to maximize bargaining leverage."* |
| **P3: FinOps & IT Finance Lead** | Tracks IT run-rate spend, unit economics, SaaS allocation, and cost center chargebacks. | Paying for inactive/dormant licenses; unbudgeted true-up liabilities mid-fiscal year. | *"When reviewing monthly cloud and SaaS spend, I want continuous alerts for underutilized licenses and projected true-up liabilities so I can optimize run-rate cash flows."* |

---

## 3. Product Scope & Functional Architecture

### 3.1 In-Scope vs. Out-of-Scope
+-------------------------------------------------------------------------------+
| IN-SCOPE (Phase 1 & Phase 2)                                                  |
+-------------------------------------------------------------------------------+
| [x] PDF/OCR Contract Parsing (MSAs, SOWs, Order Forms)                        |
| [x] Structured JSON Schema Extraction of Entitlement Terms                    |
| [x] Integration with CSV/SQL Telemetry (Entra ID, Okta, CMDB, SSO Logs)      |
| [x] Deterministic Effective License Position (ELP) Math Engine                |
| [x] Shelfware Identification & Financial Cost Exposure Modeling               |
| [x] Executive Renewal Strategy & Down-Tiering Brief Generation                |
| [x] Human-in-the-Loop (HITL) Clause Verification & Confidence Scoring         |
| [x] Interactive Streamlit Analyst Workspace & PDF/Executive Export            |
+-------------------------------------------------------------------------------+
| OUT-OF-SCOPE (Deferred to Future Releases)                                    |
+-------------------------------------------------------------------------------+
| [-] Automated vendor deprovisioning API triggers (e.g., auto-revoking Okta)   |
| [-] Direct autonomous email negotiation sending to vendor account reps        |
| [-] Live ERP contract signature execution / DocuSign write-back               |
+-------------------------------------------------------------------------------+


### 3.2 Agentic System Architecture

The system uses a Multi-Agent RAG architecture that decouples probabilistic NLP from deterministic business logic:

                  +-----------------------------+
                  |   Enterprise MSA / PDFs     |
                  +--------------+--------------+
                                 |
                                 v
                  +-----------------------------+
                  | PyMuPDF / OCR Preprocessing |
                  +--------------+--------------+
                                 |
                                 v
                  +-----------------------------+
                  |  Vector Store (ChromaDB)    |
                  |  Semantic Chunking & RAG    |
                  +--------------+--------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| Multi-Agent Orchestration Layer                                       |
|                                                                       |
|  [ Agent 1: Contract Parser ]                                         |
|  - Model: GPT-4o-mini / Haiku                                         |
|  - Task: Extracts metric types, caps, dates, penalties into JSON      |
|                                                                       |
|                               v                                       |
|                                                                       |
|  [ Agent 2: Deterministic Reconciler (Zero-LLM Math Engine) ]         |
|  - Language: Pure Python / Pandas                                     |
|  - Task: Reconciles usage vs entitlements; calculates over/under      |
|                                                                       |
|                               v                                       |
|                                                                       |
|  [ Agent 3: FinOps Negotiation Advisor ]                              |
|  - Model: Claude 3.5 Sonnet / GPT-4o                                  |
|  - Task: Synthesizes leverage points, down-tiering & negotiation plan |
+-----------------------------------------------------------------------+
|
v
+-----------------------------+
| Human-in-the-Loop (HITL)    |
| Confidence Threshold Check  |
+--------------+--------------+
|
v
+-----------------------------+
| Interactive Streamlit UI /  |
| Executive Brief PDF Export  |
+-----------------------------+


---

## 4. Detailed Functional Requirements (FRs)

### 4.1 Module 1: Intelligent Contract Ingestion & Parsing
* **FR-1.1 Document Ingestion:** The system must accept unstructured vendor agreements in PDF, DOCX, and TXT formats up to 100MB per file.
* **FR-1.2 Schema-Driven Extraction:** The Parser Agent must extract the following discrete entities conforming strictly to a validated Pydantic schema:
  * `vendor_name` (string)
  * `product_name` (string)
  * `contract_type` (Perpetual, Subscription, Capacity, Consumption)
  * `metric_unit` (Named User, Concurrent User, Per Core, PVU, Flat Tier)
  * `purchased_entitlement_quantity` (integer/float)
  * `unit_cost_usd` (float)
  * `contract_effective_date` & `contract_expiration_date` (ISO-8601 date)
  * `renewal_notice_window_days` (integer, e.g., 30, 60, 90 days)
  * `auto_renewal_clause` (boolean + exact source excerpt)
  * `audit_lookback_period_months` (integer)
  * `true_up_penalty_rate_pct` (float, e.g., 100% list price or 120% penalty)
* **FR-1.3 Grounded Citations:** Every extracted entity must output exact page coordinates, section number, and verbatim source snippet.

### 4.2 Module 2: Telemetry Reconciliation & Arithmetic Engine
* **FR-2.1 Telemetry Ingestion:** Support structured telemetry feeds (CSV, JSON, SQL database query results) containing user IDs, assignment dates, last active login timestamps, and license tier assignments.
* **FR-2.2 Inactivity & Shelfware Classification:**
  * Active: Activity within <= 30 days.
  * Low Utilization: Activity between 31 and 60 days.
  * Dormant / Shelfware Candidate: Inactivity >= 61 days.
* **FR-2.3 Effective License Position (ELP) Math:**
  $$\\text{ELP Variance} = \\text{Purchased Entitlements} - \\text{Active Deployed Units}$$
  * If $\\text{ELP Variance} < 0$: Flag as **Under-Licensed (Audit Exposure Risk)**. Calculate financial exposure:
    $$\\text{Exposure (\\$)} = |\\text{ELP Variance}| \\times \\text{Unit Cost} \\times (1 + \\text{Penalty Rate})$$
  * If $\\text{ELP Variance} > 0$: Flag as **Over-Licensed (Shelfware Waste)**. Calculate annual reclaimable capital:
    $$\\text{Waste (\\$)} = (\\text{Dormant Units} + \\text{Unassigned Units}) \\times \\text{Unit Cost}$$
* **FR-2.4 Pure Deterministic Isolation:** No LLM is permitted to perform arithmetic addition, subtraction, multiplication, or financial totals.

### 4.3 Module 3: Autonomous Renewal Strategy Generator
* **FR-3.1 Brief Generation:** Synthesize contract terms, reconciliation numbers, and market benchmarks into an executive briefing document containing:
  1. Executive Summary & Audit Posture
  2. Entitlement vs. Usage Summary Table
  3. Identified Shelfware & Recommended License Reallocation
  4. Contractual Risks (Auto-renewal deadlines, price index caps)
  5. Structured 3-Step Negotiation Script for Procurement
* **FR-3.2 Guardrailed Generation:** The model must adhere to strict negative constraints (no unsubstantiated legal claims, no speculative market rates unless cited).

### 4.4 Module 4: Human-in-the-Loop (HITL) & Governance
* **FR-4.1 Confidence Flagging:** Extractions with confidence scores $< 0.85$ must trigger an inline verification modal in the UI.
* **FR-4.2 Manual Override & Audit Logging:** Users can manually correct extracted terms. All overrides must be logged with timestamp and user ID.
