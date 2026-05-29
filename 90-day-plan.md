# The 90-Day Evals Engineer Transition Roadmap

Traditional software testing mindsets do not apply to probabilistic LLMs. This 90-day plan is the execution curriculum for Software Engineers and QA Specialists transitioning into an Evals Engineering role.

## Month 1: Mastering Golden Datasets & Metric Design
You cannot evaluate an LLM if you do not have a statistically significant baseline of truth. Avoid public benchmarks and focus strictly on proprietary data.

* **Objective:** Extract real, domain-specific production logs and identify edge cases.
* **Action:** Build a 150-250 row "Golden Dataset" representing your production environment (e.g., specific customer support queries or legal contract contexts). 
* **Annotation:** Manually define human-annotated "ideal" outputs for every single prompt. Do not use synthetic data.
* **Metrics:** Define your baseline targets. Focus strictly on Faithfulness (hallucination reduction), Context Precision, and Answer Relevance for RAG applications.

## Month 2: The Automated Tool Stack
Navigate the fragmented tooling ecosystem by separating the observability layer from the testing layer.

* **Observability Layer:** Implement OpenTelemetry (OTel v1.30 GenAI specs) or Langfuse/LangSmith to capture LLM calls, latency, and nested agent traces. Ensure attributes like `gen_ai.system` and `gen_ai.usage.input_tokens` are firing.
* **Evaluation Layer:** Write Python scripts using DeepEval or Ragas.
* **Deliverable:** A local GitHub repository containing a fully functional test suite that ingests your Month 1 dataset, queries an LLM API, and outputs a JSON report of quality scores.

## Month 3: CI/CD Pipelines & Regression Testing
Evaluation is useless if it does not block bad code from reaching production. Transition from offline notebook scripting to hard engineering gates.

* **Version Control:** Move all system prompts into version-controlled files (no environment variables).
* **Automation:** Configure GitHub Actions or GitLab CI. When a developer modifies a prompt or updates an API version, trigger the DeepEval suite automatically.
* **The Quality Gate:** Establish hard failure blockers. For example, if a PR drops the faithfulness score below 0.85, the CI/CD pipeline must fail the build.
* **Deliverable:** A public portfolio repo demonstrating an automated CI/CD workflow blocking a pull request based on probabilistic metric degradation.
