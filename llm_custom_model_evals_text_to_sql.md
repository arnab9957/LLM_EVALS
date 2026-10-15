# Running Custom LLM Model Evals — Text-to-SQL Case Study

> **Core idea:** Public leaderboards help shortlist candidate models. A custom evaluation decides which model performs best on *your task, data, constraints, and production environment*.

## 1. Why Run Custom Model Evals?

A model can rank highly overall and still fail on your schema, SQL dialect, query complexity, latency target, or budget. Custom evals provide an apples-to-apples test using the same task, prompt, dataset, and scoring rules for every candidate.

**Selection pipeline:**

1. Define requirements and hard limits.
2. Use public benchmarks and leaderboards to shortlist candidates.
3. Build and validate a task-specific golden dataset.
4. Run every candidate under the same conditions.
5. Compare correctness, latency, cost, robustness, and deployment fit.
6. Select a model, then validate on a holdout set and monitor after deployment.

---

## 2. Define the Requirements

| Requirement | Questions to answer | Text-to-SQL example |
|---|---|---|
| **Task** | What must the model do? | Convert a natural-language question plus database schema into executable SQL. |
| **Cost** | What is the maximum cost per request/month? | Estimate input and output tokens per query and multiply by expected traffic. |
| **Latency** | What response time is acceptable? | Set a target for p50 and p95 end-to-end latency, not just average latency. |
| **Context** | How much input must fit? | User question + schema + instructions + examples. Measure actual tokens, not only the advertised context window. |
| **Deployment** | Where and how will it run? | API availability, rate limits, uptime, privacy, data retention, regional needs, provider terms, and fallback options. |
| **Correctness** | What counts as a valid answer? | SQL executes safely and returns the correct result for the test database. |

### Hard constraints vs. preferences

- **Hard constraints** are pass/fail gates: for example, minimum execution accuracy, maximum p95 latency, maximum cost, required SQL dialect, or privacy requirements.
- **Preferences** are used to rank models that pass all hard constraints: for example, better accuracy, faster output, or lower cost.

Do not let a very low price compensate for failing a critical correctness or safety requirement.

---

## 3. Public Leaderboards: Shortlist, Don't Blindly Trust

| Resource | Most useful for | What to remember |
|---|---|---|
| [Artificial Analysis](https://artificialanalysis.ai/) | Comparing model intelligence, price, and inference performance | Inspect the methodology, workload, provider endpoint, and date. Price and speed can vary by provider. |
| [LiveBench](https://livebench.github.io/) | General objective tasks across reasoning, coding, math, and other categories | It refreshes questions to reduce contamination; check the release/version and task coverage. |
| [Arena](https://arena.ai/leaderboard) | Human-preference rankings for open-ended chat | Preference is not the same as factual correctness or success on a specific application. |
| [OpenRouter Models](https://openrouter.ai/models) | Discovering models available through OpenRouter and comparing their listed context and pricing | Verify model ID, provider, and current pricing before running the test. |
| [Spider](https://yale-lily.github.io/spider) | Cross-domain text-to-SQL evaluation | Useful baseline, but not a substitute for your own database schema. |
| [BIRD](https://bird-bench.github.io/) | More realistic text-to-SQL with large databases and database content | Check dataset version, database setup, and evaluation protocol. |
| [Spider 2.0](https://spider2-sql.github.io/) | Enterprise-style text-to-SQL workflows | More complex than generating a short query from a small schema; match it to your product scope. |

**Shortlisting rule:** Keep models that appear capable on relevant public tests and meet initial cost, context, availability, and deployment constraints. Then run a custom eval. Leaderboard rank alone is not a production decision.

---

## 4. Cost and Latency Calculations

### 4.1 Estimated cost per request

If prices are stated per **one million tokens**:

\[
\text{Cost} = \frac{T_{in}P_{in} + T_{out}P_{out}}{1{,}000{,}000}
\]

Where:
- \(T_{in}\): input tokens (question, schema, instructions, examples)
- \(T_{out}\): generated output tokens
- \(P_{in}\), \(P_{out}\): input and output price per million tokens

Include billed reasoning tokens, cached tokens, tool calls, retries, and other charges when applicable. For real cost, use the usage and pricing information returned by the provider and the current OpenRouter model listing. Longer prompts and reasoning-heavy responses can cost more even when the displayed token price is the same.

**Monthly estimate:**

\[
\text{Monthly cost} \approx \text{requests/month} \times \text{mean cost/request}
\]

Add a safety margin for retries, traffic peaks, prompt growth, and model/provider price changes.

### 4.2 Measure latency correctly

Record at least:

- **End-to-end latency:** request start to complete response.
- **Time to first token (TTFT):** useful for streaming applications.
- **p50 latency:** median response time.
- **p95 latency:** 95% of requests complete at or below this value.
- **Error / timeout rate:** failed requests divided by total requests.

Benchmark under realistic input lengths and concurrency. Do not compare one warm, low-load run against another model under load.

### 4.3 Normalizing cost and latency

For a set of candidate models, min–max normalization gives values from 0 to 1.

For a **higher-is-better** metric such as correctness:

\[
N_{high}(x)=\frac{x-x_{min}}{x_{max}-x_{min}}
\]

For a **lower-is-better** metric such as cost or latency:

\[
N_{low}(x)=\frac{x_{max}-x}{x_{max}-x_{min}}
\]

A score of 1 is best within the candidate set; 0 is worst. If all candidates have the same value for a metric, define a common score for that metric or omit it from ranking. Min–max scores are relative to the candidate set, so adding a new model changes the normalized scores.

### 4.4 Example weighted ranking

After applying hard constraints, you could calculate:

\[
\text{Overall}=0.70N_{correctness}+0.15N_{latency}+0.15N_{cost}
\]

Example values (illustrative only):

| Candidate | Execution accuracy | Mean cost/query | p95 latency |
|---|---:|---:|---:|
| Model A | 91% | $0.012 | 1.8 s |
| Model B | 89% | $0.004 | 0.8 s |
| Model C | 94% | $0.025 | 2.9 s |

Model C is most accurate, Model B is fastest/cheapest, and Model A is a middle ground. Apply your **minimum accuracy, maximum cost, and p95 limits first**; then normalize and rank the candidates that passed. The weights should reflect business needs and be chosen before looking at the results to avoid cherry-picking.

**Better still:** report the Pareto frontier—models for which no other candidate is both more accurate and cheaper/faster—instead of hiding every trade-off in a single score.

---

## 5. Text-to-SQL Custom Evaluation: Full Workflow

### Step 1 — Find and filter the dataset

Choose realistic natural-language questions and a database that represents your intended use case. Sources may include curated questions, anonymized real queries, domain experts, or public Text-to-SQL datasets.

Filter out duplicates, malformed examples, out-of-scope questions, sensitive data, and questions that cannot be answered from the available schema. Keep an untouched holdout set for final validation.

### Step 2 — Extract and document the schema

Collect table names, columns, data types, primary/foreign keys, and relevant relationships. Include business definitions and database-specific rules where needed. Keep the schema used in the test identical for every candidate.

### Step 3 — Build the golden dataset

A **golden dataset** is a set of inputs with human-verified expected outputs or a trustworthy evaluation target.

Recommended fields:

| Field | Purpose |
|---|---|
| `id` | Stable example identifier |
| `question` | Natural-language input |
| `difficulty` | Easy / medium / hard, or domain-specific label |
| `gold_sql` | Verified reference SQL for auditing |
| `gold_result_json` | Expected result rows and, where appropriate, column metadata |
| `order_sensitive` | Whether row ordering is part of the expected answer |
| `category` | Joins, aggregation, filtering, NULL handling, subquery, date logic, etc. |

You can store the dataset in CSV with JSON encoded in a cell, or use JSONL/Parquet if nested data becomes awkward. Use stable IDs and version the dataset.

### Step 4 — Validate the golden dataset

- Execute every reference SQL against the intended database.
- Check that each question is answerable from the schema.
- Inspect empty, duplicate, NULL, numeric, date, and ordering cases.
- Have a domain expert review ambiguous or business-sensitive definitions.
- Record the database version and dataset version.

A faulty gold answer makes the evaluation untrustworthy—even if the model output is reasonable.

### Step 5 — Check the evaluation flow with one request

Before spending credits on the full suite, test one prompt end-to-end:

1. Load schema and question.
2. Call one candidate through OpenRouter.
3. Extract SQL from the response.
4. Execute SQL against a test database.
5. Compare its result with the golden result.
6. Record usage, cost, latency, errors, and raw output.

### Step 6 — Select candidate models

Use public benchmarks and OpenRouter model details to create a short list. Confirm the exact model/provider ID, current price, context size, supported features, rate limits, privacy requirements, and any licensing or deployment constraints.

### Step 7 — Build the evaluator

For Text-to-SQL, **plain string equality is not enough**: different SQL statements can correctly answer the same question.

Useful metrics include:

- **Execution accuracy:** fraction of questions for which generated SQL returns an equivalent result to the validated expected result.
- **SQL execution success rate:** fraction of generated queries that execute without error.
- **Exact-match accuracy:** optional structural comparison with the reference SQL; useful diagnostically but too strict as the sole correctness measure.
- **Schema validity:** whether tables/columns referenced by SQL exist.
- **Safety rate:** whether the query obeys read-only and access restrictions.
- **Cost and latency:** measured per example and summarized by model.

For execution accuracy:

\[
\text{Execution Accuracy}=\frac{\text{correct result sets}}{\text{total test questions}}\times100\%
\]

Result comparison must respect task semantics. Do not always ignore column identity, duplicates, or row ordering: ignore ordering only when order is not meaningful, and do not treat different outputs as equal merely because they contain similar values. Test the comparator with known edge cases.

### Step 8 — Protect the database

Generated SQL is untrusted input. Run it only against a disposable or read-only test database with restricted permissions, query timeouts, row/output limits, and an allowlist of permitted statement types. Block writes, schema changes, multiple statements, and access to sensitive tables. Do not evaluate unknown generated queries against production credentials.

### Step 9 — Run all models under the same conditions

For every model:

1. Send the same schema, question, instructions, examples, and SQL dialect.
2. Keep temperature, token limits, retry policy, and timeout policy consistent where possible.
3. Run the same version of the golden dataset.
4. Capture model ID/provider, output SQL, execution result, correctness, input/output usage, measured cost, latency, and failure reason.
5. Store per-example records—not only the final aggregate score.

Randomness and provider load can change results. Use deterministic settings where available and repeat a subset of examples when variance matters.

### Step 10 — Analyze failures, not just averages

Break results down by difficulty and category: joins, nested queries, aggregations, date ranges, NULL semantics, ranking, and domain-specific business rules. Inspect failed SQL and classify causes such as schema hallucination, wrong join, logic error, invalid syntax, timeout, or formatting issue.

A 90% aggregate score can hide a 40% failure rate on the one category your application uses most.

### Step 11 — Make the decision

1. Remove candidates that fail hard requirements.
2. Compare execution accuracy and safety first.
3. Compare p95 latency, cost per successful answer, and reliability.
4. Review category-level failures and manual samples.
5. Run a holdout test on finalists.
6. Select based on the application’s actual priorities; document the trade-off.

Consider **cost per correct answer** as well as cost per request:

\[
\text{Cost per correct answer}\approx\frac{\text{total evaluation cost}}{\text{number of correct answers}}
\]

This can reveal that a cheaper model is more expensive in practice if it makes many mistakes or requires retries.

---

## 6. OpenRouter: Practical Setup

The referenced CampusX project uses `ChatOpenRouter`, a model list, a schema file, a golden CSV, an evaluator, and a main orchestration script.

Repository: [campusx-official/llm-sql-eval](https://github.com/campusx-official/llm-sql-eval)

Key files in the repository include:

- `schema_extractor.py` — extracts schema information.
- `schema.sql` — schema supplied to the model.
- `golden_dataset.csv` / dataset-generation scripts — test questions and expected targets.
- `model_openrouter_slug.py` — list of candidate model IDs.
- `evaluator.py` — compares generated and expected results.
- `main.py` — orchestrates generation, SQL execution, scoring, and result logging.
- `eval_results.csv` — stores run results.

### Basic setup pattern

1. Clone the repository and inspect its README/files before running it.
2. Create a virtual environment.
3. Install the dependencies listed in the project, such as `langchain-openrouter`, `python-dotenv`, and `pandas`.
4. Create a `.env` file with `OPENROUTER_API_KEY=...`; never commit this file or expose the key in logs.
5. Configure candidate model IDs using the current IDs shown in [OpenRouter Models](https://openrouter.ai/models).
6. Test one request before launching the full evaluation.
7. Save results and record the dataset version, prompt, model ID, and run configuration.

The code and current provider catalog may change. Check the repository and official [OpenRouter quickstart](https://openrouter.ai/docs/quickstart) for current setup details. OpenRouter reports usage and cost information; confirm the returned usage fields and the pricing shown for the selected model/provider.

---

## 7. Common Evaluation Mistakes

- **Using only a public leaderboard:** it may not reflect your schema or traffic.
- **Comparing different prompts or token budgets:** the test is no longer apples-to-apples.
- **Treating exact SQL string match as correctness:** equivalent SQL may be formatted or structured differently.
- **Using an unvalidated golden dataset:** incorrect reference results corrupt the score.
- **Using too few or too-easy questions:** the ranking may not generalize to real workload.
- **Ignoring long-tail latency and failures:** average latency hides slow requests; mean accuracy hides difficult categories.
- **Ignoring price per successful answer:** the cheapest call may not be the cheapest solution.
- **Ignoring SQL safety:** execution accuracy is not a security audit.
- **Tuning on the holdout set:** repeated optimization against the final test causes leakage. Keep a separate development set.
- **Choosing weights after viewing results:** this can bias the winner. Define weights and thresholds in advance.

---

## 8. Compact Checklist

- [ ] Define task, SQL dialect, correctness target, cost ceiling, p95 latency limit, context needs, and deployment constraints.
- [ ] Shortlist models using relevant public benchmarks and the current OpenRouter catalog.
- [ ] Build a diverse, versioned golden dataset and validate every expected result.
- [ ] Smoke-test the complete evaluation pipeline with one request.
- [ ] Execute generated SQL only in a protected, read-only environment.
- [ ] Use execution accuracy plus safety, latency, cost, and failure metrics.
- [ ] Run all candidates under comparable conditions and save per-example traces.
- [ ] Analyze failures by category and difficulty.
- [ ] Filter by hard requirements before any weighted ranking.
- [ ] Validate finalists on holdout data and document the final trade-off.

## References

- [CampusX Text-to-SQL evaluation repository](https://github.com/campusx-official/llm-sql-eval)
- [OpenRouter Models](https://openrouter.ai/models)
- [OpenRouter Quickstart](https://openrouter.ai/docs/quickstart)
- [Artificial Analysis methodology](https://artificialanalysis.ai/methodology)
- [LiveBench](https://livebench.github.io/)
- [Arena leaderboard](https://arena.ai/leaderboard)
- [Spider benchmark](https://yale-lily.github.io/spider)
- [BIRD benchmark](https://bird-bench.github.io/)
- [Spider 2.0](https://spider2-sql.github.io/)
