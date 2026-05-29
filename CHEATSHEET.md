# LLM Evals Engineering — Practitioner Cheatsheet (2026)

---

## Core Metrics Glossary

| Metric | Definition | Measured By |
|---|---|---|
| **Faithfulness** | Does the generated output contradict the retrieved context? (intrinsic hallucination check) | DeepEval, Ragas, Galileo |
| **Answer Relevance** | Does the response actually address the user's query? | DeepEval, Ragas |
| **Context Precision** | Was the retrieved context actually relevant to the query? | DeepEval, Ragas |
| **Context Recall** | Did the retrieval step find all the relevant documents? | Ragas |
| **G-Eval** | General LLM-as-a-judge scoring using a task-specific rubric on an arbitrary dimension | DeepEval |
| **Hallucination** | Combined detection of intrinsic + extrinsic fabrications | DeepEval, Galileo, Patronus AI |
| **Task Completion Rate** | For agentic workflows: did the model complete all required steps? | Custom rubric |
| **P95 Latency** | 95th percentile LLM API call duration — isolates slow tail responses | OpenTelemetry → Grafana/Arize |

---

## Hallucination Taxonomy

```
Hallucination
├── Intrinsic — Output CONTRADICTS the provided context
│   └── Measurable with faithfulness scoring (high confidence)
│   └── Caught by: DeepEval faithfulness, Ragas faithfulness
│
└── Extrinsic — Output INVENTS detail not present in context
    └── Neither supported nor contradicted by retrieved docs
    └── Most dangerous in production — plausible-sounding fabrications
    └── Caught by: FActScore, Galileo automated judge, Patronus AI
```

---

## Benchmark Reference

| Benchmark | Type | What It Catches | Best Used For |
|---|---|---|---|
| TruthfulQA | Open-domain factual | Imitative falsehoods; common misconceptions | Baseline model comparison |
| HaluEval | Task-specific | Summarization, QA, dialogue hallucination | RAG pipeline evaluation |
| FActScore | Atomic fact verification | Long-form generation accuracy at claim level | Knowledge-intensive generation |
| MT-Bench | Pairwise preference | General multi-turn quality (0.80–0.88 Spearman vs. human) | Judge model baseline validation |

---

## Tool Layer Map

```
User Request
    │
    ▼
[Application / RAG Pipeline]
    │
    ├──► OpenTelemetry spans ──► OTLP Exporter ──► Langfuse (production traces)
    │                                           └──► Arize AI (enterprise monitoring)
    │                                           └──► Grafana Tempo (infrastructure)
    │
    └──► CI/CD (GitHub Actions)
              │
              ▼
         DeepEval / Ragas
         (offline evaluation on golden dataset)
              │
              ├── PASS → merge allowed
              └── FAIL → PR blocked (faithfulness < 0.85)
```

---

## Required OTel Span Attributes (v1.30 GenAI Conventions)

```python
# Mandatory for compliant LLM traces
gen_ai.system               # "openai" | "anthropic" | "cohere" | "google"
gen_ai.request.model        # "gpt-4o-2024-05-13" (what your code requested)
gen_ai.response.model       # Actual model that served (detects silent deprecations)
gen_ai.usage.input_tokens   # Prompt token count
gen_ai.usage.output_tokens  # Completion token count

# OTLP Exporter config for Langfuse
OTEL_EXPORTER_OTLP_ENDPOINT = "<langfuse-ingestion-url>"
OTEL_EXPORTER_OTLP_HEADERS  = "Authorization=Bearer <key>"
```

---

## LLM-as-a-Judge: Bias & Calibration Quick Reference

| Risk | Description | Mitigation |
|---|---|---|
| Position bias | Judge favors first response in pairwise; inflates scores 15–22% | Concordance check: run comparison twice, swap order, only accept agreement |
| Length bias | Judge rewards verbose responses regardless of quality | Calibrate on human-annotated examples; check correlation |
| Calibration failure | Judge's scores don't correlate with human expert judgment | Spearman ρ ≥ 0.70 required before production use |
| Reward hacking | Model learns to optimize for judge's stylistic preferences | Monitor external benchmarks (TruthfulQA, HaluEval) continuously |

**Calibration protocol:**
1. Select 100 representative production examples
2. Have domain experts score on your rubric
3. Run same examples through judge model
4. Calculate Spearman correlation
5. If ρ < 0.70: rewrite rubric or change judge model

---

## Enterprise Rubric Quick Reference (7 Dimensions)

Each scored 1–5. Cohen's Kappa ≥ 0.70 required before automating.

```
1. Factuality & Grounding     — every claim traceable to retrieved context
2. Regulatory Constraint      — appropriate refusals; avoids unauthorized advice  
3. Brand Safety & Tone        — professional, neutral, aligned with style guide
4. Context Precision          — answer uses relevant context for the actual query
5. Toxicity & Bias            — no demographic bias, exclusionary language, toxicity
6. Task Completion Rate       — all steps completed in agentic workflows
7. Formatting & Output Schema — adherence to requested output format (JSON, etc.)
```

**Regulatory mapping:**
- NIST AI RMF "Measure" component → dimensions 1, 2, 5, 7
- EU AI Act Article 9 (August 2026 deadline) → all 7 dimensions + historical log requirement

---

## CI/CD Quality Gate Setup (GitHub Actions)

```yaml
# .github/workflows/llm-eval.yml
name: LLM Eval Gate
on: [pull_request]

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install deps
        run: pip install deepeval
      - name: Run eval suite
        run: deepeval test run tests/eval_suite.py
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      # Build fails if faithfulness < 0.85 threshold
```

**Two-tier threshold system:**
- Hard block: metric score < 0.80 → PR blocked automatically
- Soft warning: metric score 0.80–0.85 → PR flagged for mandatory human review

---

## RLAIF vs RLHF Cost Comparison

| Scale | RLHF ($0.08/sample) | RLAIF ($0.0003/sample) | Savings |
|---|---|---|---|
| 10,000 samples | $800 | $3 | 267× |
| 100,000 samples | $8,000 | $30 | 267× |
| 1,000,000 samples | $80,000 | $300 | 267× |
| 10,000,000 samples | $800,000 | $3,000 | 267× |

**Use RLHF (not RLAIF) for:** medical diagnosis, legal drafting, financial compliance, any domain where AI judge calibration cannot be fully validated against expert knowledge.

---

## Golden Dataset Build Checklist

- [ ] Minimum 150–250 examples (larger = more statistical power)
- [ ] Sourced from actual production logs (not synthetic data)
- [ ] PII stripped before annotation
- [ ] Representative of all task types in your taxonomy
- [ ] Edge cases explicitly included (adversarial inputs, domain-specific terminology)
- [ ] Human expert annotation for ideal outputs on every example
- [ ] Version-controlled in Git alongside your eval scripts
- [ ] IRR check: two annotators, same 20 examples, Cohen's Kappa ≥ 0.70

---

*Source: [aidevdayindia.org](https://aidevdayindia.org) — May 2026. Open a GitHub issue if any figure is outdated.*
