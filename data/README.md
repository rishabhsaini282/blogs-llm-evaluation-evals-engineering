# data/ — Column Reference & Sources

## evals-engineer-compensation.csv

Compensation data for LLM Evals Engineer roles as of Q1–Q2 2026. USD figures sourced from public job listings (Scale AI, Anthropic) and compensation research published at aidevdayindia.org. India GCC figures are in INR.

| Column | Description |
|---|---|
| `level` | Level code (L4, L5, L6, or India-specific label) |
| `level_label` | Human-readable seniority label |
| `company_type` | Category of employer |
| `company_examples` | Specific companies referenced in source data |
| `base_salary_usd_low` | Lower bound of base salary in USD (or INR for India rows) |
| `base_salary_usd_high` | Upper bound of base salary in USD (or INR for India rows) |
| `total_comp_usd_low` | Lower bound of total comp including equity |
| `total_comp_usd_high` | Upper bound of total comp including equity |
| `equity_band_percent` | Equity percentage offered (applicable to Series B startup row) |
| `notes` | Context and caveats for the row |
| `source` | Source domain |

**Important**: India GCC rows (`base_salary_usd_low`, `base_salary_usd_high`, etc.) represent INR values, not USD. See the `notes` column for clarification.

## eval-platforms.csv

Platform comparison data for production LLM evaluation tools. Ratings are qualitative assessments (High / Medium / Low / N/A) based on published documentation and the analysis at aidevdayindia.org as of May 2026. Platform capabilities change — verify against current documentation before procurement.

| Column | Description |
|---|---|
| `platform` | Tool name |
| `primary_use_case` | Main function (observability vs. evaluation framework vs. full platform) |
| `pricing_model` | How the platform charges |
| `trace_volume_ceiling` | Known or documented limits before performance degrades |
| `self_hosting_available` | Whether self-hosted deployment is supported |
| `devops_overhead` | Operational complexity of running at scale |
| `otel_support` | Native OpenTelemetry / OTLP ingestion support |
| `rag_metrics_native` | Whether RAG-specific metrics (faithfulness, context precision) are built in |
| `langchain_integration` | Native LangChain / LangGraph callback support |
| `best_for` | Recommended use case |
| `primary_limitation` | Key failure mode at scale |

**Source**: [aidevdayindia.org](https://aidevdayindia.org) — published May 2026.
