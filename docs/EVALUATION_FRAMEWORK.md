# AI Evaluation Framework & Quality Guardrails

**Project:** Enterprise License AI Copilot (`enterprise-license-ai-copilot`)  
**Document Version:** 1.0.0  
**Owner:** AI Product Lead  
**Scope:** Offline Benchmark Harness, Online Guardrails, and LLM-as-a-Judge Evaluation Pipeline  

---

## 1. Evaluation Philosophy & Core Tenets

Enterprise contract analysis and software license reconciliation operate under strict regulatory and financial scrutiny. Our AI product evaluation framework is built on four non-negotiable tenets:

1. **Zero Arithmetic Drift:** No LLM is permitted to perform math. Financial liabilities and Effective License Position (ELP) calculations must be deterministic.
2. **Strict Provenance & Groundedness:** Every extracted contract entity must cite exact document coordinates (page number, section header, verbatim text excerpt).
3. **Structured Schema Adherence:** LLM outputs must strictly conform to Pydantic JSON schemas with zero tolerance for missing required fields.
4. **Multi-Tiered Model Routing:** Fast, low-cost models are benchmarked for entity extraction; high-reasoning frontier models are evaluated on negotiation strategy synthesis.

---

## 2. Quantitative Metric Hierarchy & Target Benchmarks

| Metric Dimension | Target Threshold | Evaluation Methodology | Tooling / Framework |
| :--- | :--- | :--- | :--- |
| **Legal Clause Extraction Precision** | $\ge 96.0\%$ | Field-level exact and semantic match vs. annotated golden dataset (50 MSAs). | Custom Pydantic Validator |
| **Legal Clause Extraction Recall** | $\ge 92.0\%$ | Measurement of missed contractual obligations (e.g., auto-renewal notice windows). | Custom Test Harness |
| **Faithfulness / Groundedness** | $\ge 0.95$ | Claim-level verification determining if every claim in the generated brief is supported by retrieved context. | DeepEval / Ragas |
| **Hallucination Rate** | $\le 2.0\%$ | Percentage of generated statements unsupported by source legal documents. | LLM-as-a-Judge (GPT-4o) |
| **RAG Retrieval Context Hit Rate** | $\ge 95.0\%$ | Top-k vector retrieval hit rate for relevant legal clauses. | ChromaDB Cosine Eval |
| **Arithmetic Integrity Error Rate** | $0.0\%$ | Unit testing of deterministic calculation layers across 10,000 synthetic permutations. | PyTest |
| **P95 Ingestion & Parsing Latency** | $\le 12.0\text{s}$ | Total execution time from raw 50-page PDF ingestion to structured JSON schema output. | OpenTelemetry / Python Timer |
| **Average Inference Cost per Run** | $\le \$0.12$ | Token usage tracking across Parser Agent and FinOps Strategy Agent. | LiteLLM / Langfuse |

---

## 3. Golden Benchmark Dataset Specification (`ground_truth_evals.json`)

The evaluation test suite runs against 50 annotated, anonymized enterprise software agreements covering major publisher licensing archetypes:

```
data/ground_truth_evals.json
├── Tier 1: SaaS Named User Contracts (Salesforce, GitHub, Adobe, Workday)
├── Tier 2: Cloud Committed Capacity (Datadog, Snowflake, AWS Enterprise)
└── Tier 3: Core & Factor-Based Licensing (Oracle DB, Microsoft SQL Server, IBM PVU)
```

### 3.1 Ground Truth Data Schema
```json
{
  "test_case_id": "MSA-GH-2026-001",
  "document_name": "GitHub_Enterprise_Cloud_Agreement_Mock.pdf",
  "expected_extractions": {
    "vendor_name": "GitHub / Microsoft",
    "product_name": "GitHub Enterprise Cloud + Copilot",
    "metric_type": "Named Active User",
    "purchased_quantity": 1250,
    "unit_rate_usd": 468.0,
    "notice_window_days": 60,
    "auto_renewal": true,
    "audit_penalty_rate_pct": 20.0
  },
  "ground_truth_context_chunks": [
    "Sec 3.2: Named user licenses are assigned to unique GitHub IDs...",
    "Sec 11.4: Agreement auto-renews for 12-month terms unless written notice is given 60 days prior...",
    "Sec 14.1: Unlicensed excess provisioning subject to retroactive list price true-up plus 20% surcharge."
  ]
}
```

---

## 4. LLM-as-a-Judge Evaluation Pipeline

To evaluate unstructured negotiation strategies, an independent LLM-as-a-Judge evaluates generated output against the ground-truth contract context using three primary criteria:

```
              +------------------------------------------+
              | Retrieved Context Chunks + User Telemetry|
              +--------------------+---------------------+
                                   |
                                   v
              +--------------------+---------------------+
              | Generated Renewal Strategy Playbook      |
              +--------------------+---------------------+
                                   |
                                   v
              +--------------------+---------------------+
              | LLM Judge (GPT-4o / Claude 3.5 Sonnet)   |
              | 1. Groundedness Score (0.0 - 1.0)        |
              | 2. Negotiation Relevance (0.0 - 1.0)     |
              | 3. Negative Constraint Compliance (Pass) |
              +--------------------+---------------------+
```

### 4.1 Judge Prompt Template (`src/eval/judge_prompt.yaml`)
```yaml
judge_system_prompt: |
  You are an expert enterprise legal counsel and IT procurement auditor.
  Evaluate the submitted Renewal Negotiation Brief against the Provided Ground Truth Context.

  Evaluation Scoring Criteria:
  1. Faithfulness (Score 0-5): Does the brief claim any contractual terms, deadlines, or penalty clauses NOT present in the context?
  2. Arithmetic Provenance (Score 0-5): Are all dollar calculations derived from deterministic inputs?
  3. Actionability (Score 0-5): Does the brief provide a concrete 3-step procurement strategy?

  Output valid JSON containing:
  {
    "faithfulness_score": float,
    "groundedness_score": float,
    "actionability_score": float,
    "hallucination_detected": bool,
    "unsupported_claims": list[str],
    "verdict": "PASS" | "FAIL"
  }
```

---

## 5. Automated CI/CD Regression Test Suite (`run_evals.py`)

Every prompt adjustment, chunking change, or model update must pass automated CI/CD gating:

```python
# src/eval/run_evals.py (Excerpt)
import pytest
from pydantic import BaseModel
from deepeval.metrics import FaithfulnessMetric, ContextualRelevancyMetric
from deepeval.test_case import LLMTestCase

def test_contract_parser_extraction_precision(benchmark_data):
    """Validates that field extraction precision meets or exceeds 96%."""
    correct_fields = 0
    total_fields = 0
    
    for case in benchmark_data:
        extracted = parse_contract_pipeline(case["document_name"])
        for field, expected_val in case["expected_extractions"].items():
            total_fields += 1
            if getattr(extracted, field) == expected_val:
                correct_fields += 1
                
    precision = correct_fields / total_fields
    assert precision >= 0.96, f"Precision failed threshold: {precision:.2%}"

def test_renewal_strategy_groundedness(sample_test_case):
    """Ensures generated renewal strategy has zero hallucinated clauses."""
    test_case = LLMTestCase(
        input=sample_test_case["query"],
        actual_output=sample_test_case["generated_strategy"],
        retrieval_context=sample_test_case["retrieved_chunks"]
    )
    metric = FaithfulnessMetric(threshold=0.95)
    metric.measure(test_case)
    assert metric.is_successful(), f"Faithfulness score {metric.score} below 0.95 threshold."
```

---

## 6. Failure Mode Analysis & Guardrail Triggers

| Identified Failure Mode | System Detection Method | Guardrail Action |
| :--- | :--- | :--- |
| **Low Extraction Confidence** | Token log-probability score $< 0.85$ on critical legal terms. | Flags field with `badge-warn` in UI; requires human analyst one-click sign-off. |
| **Schema Validation Failure** | Pydantic validation error (e.g., negative seats, invalid date formats). | Automated retry with temperature $0.0$ and explicit schema error feedback. |
| **Out-of-Distribution Metric** | Metric unit not in standard taxonomy (e.g., custom IoT payload tier). | Routes to human-in-the-loop (HITL) custom formula definition modal. |
| **Arithmetic Hallucination** | Non-zero variance between LLM output and Python math runtime. | Strict bypass: LLM calculation outputs are discarded; pure Python metrics are rendered. |
