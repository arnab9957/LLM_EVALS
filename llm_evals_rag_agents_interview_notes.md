# LLM Evals: RAG, Agents, and Application Evaluation

> **Most important GenAI interview question:** “How do you evaluate a RAG chatbot?”
>
> **Core idea:** Evaluate the system at **component, pipeline, and application levels**, then add **safety, operational monitoring, and regression testing**.

## 1. Why Focus on RAG and Agentic Applications?

- A fluent answer can still be wrong, unsupported, incomplete, unsafe, or expensive.
- In RAG, failures may come from retrieval, generation, or the interaction between them.
- Agents add more failure points: planning, tool selection, tool arguments, execution, and final response.
- Evaluation turns subjective impressions into repeatable tests and measurable release decisions.

## 2. The Three-Tier Evaluation Framework

| Level | Main question | What to test | Example metrics |
|---|---|---|---|
| **1. Component** | Does each part work on its own? | Retriever, generator, reranker, tool, parser | Contextual precision/recall/relevancy; faithfulness; tool-call accuracy; schema validity |
| **2. Pipeline / workflow** | Do components work together correctly? | Retrieved context → generated answer; agent plan → tool calls → final answer | RAG triad; end-to-end task success; correct tool sequence; execution success |
| **3. Application** | Does the product solve the user’s real problem? | Correctness, completeness, clarity, tone, refusal/abstention, UX | Answer correctness, resolution rate, completeness, user satisfaction, safety and latency thresholds |

**Why all three?** An overall score may tell you that the system failed, but component tests help identify *where* it failed. Application tests verify that technical improvements actually help users.

## 3. Case Study: CampusX Course Doubt Solver

**Example:** A student asks, “What is the difference between precision and recall?” The chatbot searches course notes and answers with citations.

A useful golden test case contains:

- `input`: the student’s question.
- `expected_output`: the key facts a correct answer must contain.
- `retrieval_context` / expected relevant chunks: authoritative course material.
- Optional expected citations, answer style, difficulty level, and refusal behavior.

**Failure examples**

- **Retrieval failure:** The system retrieves a lesson about accuracy instead of precision and recall.
- **Generation failure:** The correct lesson is retrieved, but the response defines the metrics incorrectly.
- **Citation failure:** The answer is mostly correct, but its citation points to an irrelevant chunk.
- **Application failure:** The answer is technically correct but too vague for a beginner to use.

## 4. Level 1 — Component-Level Evaluation

### A. Evaluate the retriever

| Metric | What it checks | Diagnostic use |
|---|---|---|
| **Contextual relevancy** | Are retrieved chunks relevant to the question? | Tune chunking, embeddings, filters, and `top_k` |
| **Contextual recall** | Does retrieved context contain the information needed for the reference answer? | Find missing evidence; requires reference information for many implementations |
| **Contextual precision** | Are relevant chunks ranked ahead of irrelevant ones? | Tune reranking and retrieval order |

Also inspect retrieval Recall@K, duplicate chunks, source quality, and retrieval latency when applicable.

### B. Evaluate the generator

| Metric | What it checks |
|---|---|
| **Faithfulness / groundedness** | Are claims supported by the retrieved context? |
| **Answer relevancy** | Does the response address the user’s question? |
| **Correctness** | Does the answer agree with a trusted reference or expert judgment? |
| **Citation accuracy** | Does each citation support the claim attached to it? Use a custom or dedicated citation metric. |
| **Completeness** | Does the answer cover all required key points? |

**Important distinction:** A response can be faithful to poor context and still be wrong. Groundedness alone does not prove that the retrieved documents are correct or complete.

## 5. Level 2 — Pipeline Evaluation and the RAG Triad

The commonly used **RAG triad** consists of:

1. **Contextual relevancy:** Is the retrieved context relevant to the input?
2. **Faithfulness:** Is the answer supported by the retrieved context?
3. **Answer relevancy:** Does the answer address the input?

Run all three on complete query-to-answer traces. Add reference-based correctness and retrieval precision/recall when gold answers or evidence are available.

**Debugging guide**

- Low contextual relevancy → investigate retrieval, chunk size, filters, embeddings, and `top_k`.
- Good context but low faithfulness → investigate prompt, model, context length, and instruction adherence.
- High faithfulness but low correctness → the context itself may be incomplete, stale, or incorrect.
- Good scores but poor user outcomes → add application-level tests for completeness, tone, and task resolution.

## 6. Level 3 — Application-Level Evaluation

Evaluate the product against real user goals, not just model output quality.

- **Correctness:** Is the answer factually right?
- **Completeness:** Does it fully resolve the doubt or explain what is missing?
- **Tone and clarity:** Is it understandable for the target audience?
- **Citation quality:** Can users verify important claims?
- **Abstention:** Does it say “I don’t know” when evidence is insufficient?
- **Task success:** Did the user achieve the intended outcome?
- **Consistency:** Do paraphrased versions of the same question receive compatible answers?

Use a mix of exact checks, reference-based metrics, rubric-based LLM judges, and human review. Calibrate LLM-as-a-judge results against a set of human-rated examples.

## 7. Safety and Operational Audits

### Safety

- **PII leakage:** Does the system expose private or sensitive information?
- **Jailbreak resistance:** Can adversarial instructions bypass system rules?
- **Prompt injection:** Can retrieved documents or user content manipulate the model into ignoring trusted instructions?
- **Unsafe or policy-violating output:** Does the system follow the expected refusal and safe-completion policy?
- **Access control:** Does retrieval enforce user and document permissions?

Use a dedicated red-team test set and measure attack success rate, unsafe-answer rate, and false refusals. Do not rely solely on ordinary happy-path questions.

### Operations

Track at least:

- p50 / p95 / p99 latency, including retrieval and generation stages.
- Cost per request and cost per successfully resolved task.
- Error rate, timeout rate, rate-limit failures, and availability.
- Token usage, context length, and retrieval volume.
- User feedback, escalation rate, and unanswered-question rate.

A quality score is not enough if the application is too slow, costly, or unreliable for production.

## 8. Evaluation Strategy for Different AI Applications

| Application | Important evaluations |
|---|---|
| **RAG chatbot** | Contextual precision/recall/relevancy, faithfulness, answer relevancy, correctness, citation accuracy, abstention |
| **Agent** | Goal/task success, tool selection accuracy, tool-argument correctness, tool execution success, trajectory quality, safety, latency and cost |
| **Simple chatbot** | Answer relevancy, correctness, tone, role adherence, conversation completeness, multi-turn consistency, safety |
| **Multimodal application** | Input perception accuracy (image/audio/video), cross-modal grounding, answer correctness, hallucination, robustness to poor-quality inputs, latency |
| **Fixed-schema / structured-output app** | JSON/schema validity, required-field coverage, types/enums/ranges, constraint satisfaction, semantic correctness, invalid-output rate |
| **Summarization app** | Factual consistency, key-information coverage, relevance, conciseness, and omission of critical details |

Select metrics based on the actual task and failure risks; do not apply every metric to every app.

## 9. Why Use DeepEval? How Does It Compare with Ragas?

**DeepEval** is useful when evaluation should feel like software testing: test cases, metrics, thresholds, assertions, repeatable runs, and integration with CI/CD. It includes RAG metrics and supports evaluation of safety, conversations, and agentic behavior.

**Ragas** is also a strong evaluation framework, especially for RAG and agentic workflows, with metrics for retrieval, faithfulness, answer relevance, tool use, and more.

| Consideration | DeepEval | Ragas |
|---|---|---|
| Good fit | Test-suite / pytest-like workflows, threshold assertions, CI regression gates, broad application evaluation | RAG-focused experimentation and evaluation, with a broad and evolving set of RAG/agent metrics |
| RAG metrics | Faithfulness, answer relevancy, contextual relevancy, contextual precision and recall | Faithfulness, response relevancy, context precision/recall and other RAG metrics |
| Practical choice | Useful when you want evaluation to behave like automated tests in your engineering workflow | Useful when its metrics, dataset workflows, or integrations fit your RAG experiments |

**There is no universal winner.** Compare metric definitions, judge model, cost, reproducibility, integrations, and how well the scores agree with human judgments. Scores from different frameworks may not be directly comparable.

## 10. Automated Regression Testing and CI/CD

**Regression testing** checks that a code, prompt, model, retriever, embedding, or configuration change has not broken previously working behavior.

1. Build a versioned **golden dataset** of representative questions and expected behavior.
2. Run it against the current production baseline.
3. Run the same tests against the proposed change.
4. Compare quality, safety, latency, and cost metrics by category.
5. Fail the release gate if a critical threshold is violated or a protected metric regresses beyond an agreed tolerance.
6. Review failures before accepting the release; do not rely on one aggregate score.

Include normal queries, edge cases, ambiguous questions, unanswerable questions, adversarial prompts, multilingual inputs if relevant, and examples from production failures.

Example release gates (set thresholds from product requirements and a validated baseline):

- Faithfulness and correctness must remain above their required thresholds.
- Schema validation and tool execution must not regress beyond an allowed tolerance.
- No critical PII or access-control test may fail.
- p95 latency and cost per successful task must stay within budget.

## 11. MLOps, Online Evaluation, and the Self-Improving Loop

**Offline evaluation** uses curated or historical examples before release. It is repeatable and supports controlled comparisons.

**Online evaluation** monitors live production traces to detect quality drift, new failure patterns, unusual costs, latency issues, and safety incidents. Sample high-volume traffic rather than sending every trace to an expensive LLM judge. Run additional checks on low-confidence, negative-feedback, high-risk, or anomalous cases.

### Self-improving loop

`Log traces → Sample and evaluate → Inspect failures → Label new examples → Expand golden dataset → Improve retrieval/prompt/model/tools → Run regression tests → Deploy safely → Monitor again`

The loop should preserve privacy: redact or minimize sensitive data, respect retention policies, and control who can access evaluation traces.

## 12. Interview-Ready Answer: “How Do You Evaluate a RAG Chatbot?”

> “I evaluate a RAG chatbot at three levels. First, at the **component level**, I evaluate retrieval using contextual precision, recall, and relevancy, and generation using faithfulness, answer relevancy, correctness, and citation accuracy. Second, at the **pipeline level**, I test the complete query-to-retrieval-to-answer flow, using the RAG triad and reference-based tests where gold answers are available. Third, at the **application level**, I measure whether the response is correct, complete, understandable, safe, and actually resolves the user’s need.
>
> I create a versioned golden dataset containing common questions, edge cases, unanswerable queries, and adversarial examples. I automate regression tests in CI/CD and set release gates for critical quality and safety metrics. After deployment, I monitor sampled production traces for drift, latency, cost, and new failure modes. Those failures become new test cases, creating a continuous improvement loop. I use tools such as DeepEval or Ragas for offline evaluations and a tracing/observability platform for production monitoring, while validating LLM-judge scores against human review.”

## 13. Quick Checklist

- [ ] Define the user task and failure risks.
- [ ] Evaluate retriever and generator separately.
- [ ] Test the full pipeline and application outcome.
- [ ] Include safety, access control, latency, cost, and reliability.
- [ ] Build a representative, versioned golden dataset.
- [ ] Calibrate LLM judges against human ratings.
- [ ] Add regression checks and release gates to CI/CD.
- [ ] Monitor sampled production traces and feed failures back into the dataset.

## 14. Official Resources

- DeepEval — RAG evaluation guide: https://deepeval.com/guides/guides-rag-evaluation
- DeepEval — RAG quickstart: https://deepeval.com/docs/getting-started-rag
- DeepEval — Metrics overview: https://deepeval.com/docs/metrics-introduction
- DeepEval — Faithfulness metric: https://deepeval.com/docs/metrics-faithfulness
- DeepEval source repository: https://github.com/confident-ai/deepeval
- Ragas — Available metrics: https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/
- LangSmith — Evaluation types (offline and online): https://docs.langchain.com/langsmith/evaluation-types
- LangSmith — Online LLM-as-judge evaluation: https://docs.langchain.com/langsmith/online-evaluations-llm-as-judge
