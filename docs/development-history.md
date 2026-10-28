# FRIS Development History and Technical Milestones

> Archived from the project README during documentation standardisation on 28 October 2026. Historical performance and implementation claims are preserved as originally documented and may require revalidation against current code.

## Extraction Development Checkpoint

The current extraction pipeline uses PyMuPDF, Tesseract OCR, deterministic Python table
parsing, canonical financial mappings, and validation rules. Complete-row extraction has
been tested against Apple, Amazon, and Colgate reports. The next extraction milestone is to
add model-assisted visual table reconstruction for unfamiliar or ambiguous layouts.

### Recommended models

| Model | Proposed responsibility | Deployment |
| --- | --- | --- |
| GLM-OCR | Primary visual table recognition, layout reconstruction, OCR, and structured row extraction | Local through Ollama |
| Qwen3-VL 8B | Optional financial label interpretation, period alignment, canonical mapping, and visual verification | Hosted GPU or a higher-memory development machine |

If only one model is introduced initially, start with **GLM-OCR** because the immediate
problem is accurate document and table reconstruction. Add **Qwen3-VL 8B** as the semantic
verification layer after the GLM-OCR integration is stable.

Install the primary local extraction model with:

```bash
ollama pull glm-ocr:latest
```

Qwen3-VL 8B is not recommended for the current 8 GB development machine. It remains an
optional future verification model that can be hosted remotely or used on higher-memory
hardware after the GLM-OCR integration is stable.

### Proposed hybrid extraction flow

```text
PDF page
  → embedded text or Tesseract OCR
  → deterministic table parser
  → completeness and confidence assessment
  → GLM-OCR visual table reconstruction when required
  → Qwen3-VL financial interpretation and alignment when required
  → deterministic Python calculations
  → Data Quality Agent validation
  → human review when validation fails
```

Models must not silently replace validated figures or perform authoritative calculations.
Every model-assisted row must retain its exact source label, statement section, period,
numeric value, currency, unit, PDF page, evidence, extraction method, and confidence score.

### Model-assisted extraction checklist

- [x] Add a model-provider interface under the Document Processing Agent.
- [x] Add an Ollama client with configurable model names and timeouts.
- [x] Render candidate statement pages to images for GLM-OCR.
- [x] Normalize GLM-OCR HTML/Markdown tables into structured statement rows.
- [x] Trigger automatic model assistance for incomplete or failed-validation statements.
- [x] Re-run deterministic accounting and cash-flow validations after model assistance.
- [x] Display extraction method and confidence in Streamlit for human approval.
- [x] Complete a live deterministic-versus-model Colgate benchmark.
- [ ] Benchmark live GLM-OCR output against Apple and Amazon.
- [ ] Evaluate hosted semantic verification only if GLM-OCR ambiguity remains.

## Local AI Model Milestone - 2 August 2026

The first model required for model-assisted extraction is now installed and verified on the
development machine. This moves the project from model selection into implementation and
benchmarking.

### Verified environment

| Component | Confirmed configuration |
| --- | --- |
| Development computer | Apple M2 with 8 GB unified memory |
| Model storage | External `Bintu2TB07` drive through the existing Ollama model directory |
| Ollama model | `glm-ocr:latest` (`6effedd0dc8a`) |
| GLM-OCR size | 2.2 GB, F16 |
| GLM-OCR architecture | 1.1B parameters, 131,072-token context |
| Capabilities | Vision, completion, and tools |
| Inference | Local Apple Metal, one financial-statement page at a time |

The installed supporting text models are `phi3:mini`, `llama3.1:8b`, and
`starcoder2:3b`. They are not document-vision substitutes for GLM-OCR. The intended local
allocation is:

- **GLM-OCR:** visual document, table, and statement extraction.
- **Phi-3 Mini:** lightweight explanations and narrative synthesis from validated JSON.
- **Llama 3.1 8B:** optional deeper text analysis, run separately because of memory limits.
- **StarCoder2 3B:** optional Python, SQL, DAX, and dashboard-code assistance.

The application will keep deterministic Python calculations and accounting validation as the
authoritative layer. Language models may extract or explain evidence, but must not silently
replace validated financial values.

### Model integration completed

Agent 01 now includes an Ollama/GLM-OCR provider, health checks, sequential page rendering,
HTML/Markdown table normalization, provenance, and safe deterministic fallback. Streamlit
offers automatic, model-assisted, and rules-only modes and displays the model and extraction
method used. Model-assisted figures are routed through the existing structured-statement
schema, calculations, and accounting validation before display or export.

The next implementation milestone is to benchmark live GLM-OCR output against Apple, Amazon,
and Colgate, tune the page prompts and table normalization where required, and record
deterministic-versus-model row accuracy and processing time.

## Insight Intelligence Milestone - 3 August 2026

Agents 05, 06, and 07 now form a traceable financial interpretation workflow after statement
extraction and calculation:

```text
Validated facts and ratios
  → adjacent-period trend and variance analysis
  → deterministic financial risk assessment
  → evidence-grounded insight generation
  → optional local-model management-theme synthesis
  → post-model numerical and provenance verification
```

Agent 05 classifies multi-period movements as favorable, adverse, stable, turnaround,
deterioration, or contextual. Agent 07 applies transparent profitability, liquidity, solvency,
cash-flow, performance, and reporting-quality risk thresholds. Agent 06 ranks the resulting
insights and groups them into profitability, cash-flow, liquidity, leverage, and shareholder-
return themes before any language model is called.

The optional narrative layer uses `phi3:mini` by default on the 8 GB M2. A live Amazon 2025
test produced five verified management themes covering cash generation and investment, capital
structure and leverage, liquidity and working capital, growth and profitability, and shareholder
returns and efficiency. The model connected operating cash-flow growth with the free-cash-flow
decline and balanced debt growth against the change in debt-to-equity.

Two live `llama3.1:8b` attempts exceeded the practical local timeout, so it remains selectable
for higher-memory or deliberately slower workflows rather than being the default. Model output
cannot change calculations, priorities, metrics, risks, or evidence. Python rejects invented
figures and unsupported causal claims, restores any omitted validated metric sentence, strips
unverified currency symbols, and retains deterministic insights whenever augmentation is
unavailable or invalid. The complete automated suite contains 37 passing tests.

## Development Update - 2 August 2026

Today’s work strengthened the financial-statement extraction foundation and made the project
easier to continue in a future development session.

### Completed today

- Replaced the limited KPI allowlist as the primary extractor with complete-row statement
  extraction that preserves exact PDF labels, sections, periods, values, ordering, and evidence.
- Corrected cash-flow and balance-sheet extraction across different annual-report layouts.
- Added Amazon and Colgate annual reports alongside Apple as regression samples.
- Visually compared extracted tables with rendered source pages.
- Expanded deterministic validation to include balance-sheet totals, cash rollforwards, and
  operating + investing + financing + foreign-exchange reconciliations.
- Improved Streamlit statement inspection with exact labels, section names, row counts,
  formatted amounts, larger tables, and consistent ratio formatting.
- Numbered the agent packages from `agent_01_...` through `agent_10_...` so their intended
  workflow order is visible in the repository.
- Documented the next model-assisted extraction milestone using GLM-OCR for visual table
  reconstruction and Qwen3-VL 8B for financial interpretation and verification.

### Validation snapshot

| Report | Income rows | Balance-sheet rows | Cash-flow rows | Reconciliations |
| --- | ---: | ---: | ---: | ---: |
| Apple | 19 | 27 | 33 | 12/12 passed |
| Amazon | 23 | 26 | 29 | 10/10 passed |
| Colgate | 17 | 31 | 33 | 15/15 passed |

The automated suite now contains 37 passing tests, including provider, table-normalization,
page-rendering, provenance, and safe-fallback coverage. Live Colgate model-assisted extraction
processed the three primary statements in 419 seconds on the 8 GB M2 and retained 17 income,
31 balance-sheet, and 33 cash-flow rows with all 15 validations passing.

## Milestone: Phase 1 Complete

Phase 1 was completed and validated against the included Apple, Amazon, and Colgate annual
reports. The Apple pipeline automatically detected corrupt embedded PDF text, classified the
document at low resolution, and applied detailed OCR only to the primary statements.

### Verified extraction

| Statement | Source page | Reporting periods | Currency and unit |
| --- | ---: | --- | --- |
| Consolidated statements of operations | 32 | 2022, 2021, 2020 | USD millions |
| Consolidated balance sheets | 34 | 2022, 2021 | USD millions |
| Consolidated statements of cash flows | 36 | 2022, 2021, 2020 | USD millions |

### Verified 2022 calculations

| KPI | Result |
| --- | ---: |
| Gross margin | 43.31% |
| Operating margin | 30.29% |
| Net margin | 25.31% |
| Current ratio | 0.88x |
| Working capital | $(18,577) million |
| Debt-to-equity | 2.37x |
| Debt ratio | 0.34x |
| Asset turnover | 1.12x |
| Return on average assets | 28.36% |
| Return on average equity | 175.46% |
| Free cash flow | $111,443 million |
| Basic / diluted EPS | $6.15 / $6.11 |

### Cross-report extraction coverage

| Sample | Income rows | Balance-sheet rows | Cash-flow rows | Checks passed |
| --- | ---: | ---: | ---: | ---: |
| Apple 2022 Form 10-K | 19 | 27 | 33 | 12/12 |
| Amazon 2025 Annual Report | 23 | 26 | 29 | 10/10 |
| Colgate 2025 Annual Report | 17 | 31 | 33 | 15/15 |

An external EE Limited FY2015 IFRS report was also validated against the Key Financial Facts
workflow. The parser identified `Group income statement`, `Statements of financial position`,
and the two-page `Statements of cash flows`; selected Group rather than Company columns; and
returned 17/21 canonical facts in approximately 0.35 seconds without model assistance. Missing
facts remained explicit rather than being inferred from narrative notes.

The complete automated suite contains 37 passing tests. Full two-pass Apple OCR and analysis
completes in approximately 48 seconds on the development machine; text-native Amazon and
Colgate reports complete in approximately one second each.

### Agent implementation status

| Agent | Status |
| --- | --- |
| Document Processing Agent | Implemented |
| Metrics Extraction Agent | Implemented |
| Financial Calculation Engine | Implemented |
| Data Quality Agent | Implemented |
| Financial Analysis Agent | Implemented |
| Insight Generation Agent | Implemented |
| Risk Assessment Agent | Implemented |
| Visual Blueprint Agent | Planned folder ready |
| Narrative Synthesis Agent | Baseline implemented |
| Export Agent | JSON available; additional formats planned |

