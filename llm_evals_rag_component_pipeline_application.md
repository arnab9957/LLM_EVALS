# LLM Evals — RAG Component, Pipeline, and Application Evaluation

> Concise study notes based on the supplied video agenda. These notes summarize the concepts and workflow; they are not a verbatim transcript.

## 1. Evaluation Levels

| Level | What is tested? | Main question | Example |
|---|---|---|---|
| **Component** | Retriever or generator in isolation | Which component is failing? | Did retrieval find useful chunks? |
| **Pipeline** | Retriever + generator working together | Does the complete RAG flow work? | Are retrieved contexts relevant and answers grounded? |
| **Application** | Full user-facing behavior | Does the app solve the user's task safely and well? | Is the answer correct, complete, clear, safe, and timely? |

Evaluate each level separately: a good retriever and a good generator can still combine into a poor application.

## 2. Suggested Project Structure

```text
rag-eval-project/
├── src/
│   ├── retriever.py
│   ├── generator.py
│   └── rag_pipeline.py
├── evals/
│   ├── eval_retriever.py
│   ├── eval_generator.py
│   ├── eval_rag_pipeline.py
│   └── eval_application.py
├── goldens/
│   ├── retriever_goldens.csv
│   └── rag_goldens.csv
├── data/
│   └── transcripts/
├── pyproject.toml
└── README.md
```

- `src/`: reusable application code.
- `evals/`: evaluation scripts and metrics.
- `goldens/`: reviewed evaluation examples with questions, expected answers, and relevant reference information.
- Keep evaluation data versioned and avoid placing API keys in source control.

## 3. Build the RAG System

1. Ingest source documents or transcripts.
2. Clean and preprocess the text.
3. Split documents into chunks; record chunk size and overlap.
4. Generate embeddings and store chunks in a vector database such as ChromaDB.
5. Retrieve the top-*k* chunks for a question.
6. Pass the question and retrieved context to the generator.
7. Generate an answer with a clear instruction to use the supplied context and acknowledge missing information.

**Important:** Chunk size, overlap, embedding model, top-*k*, reranking, prompt, and generator model are experimental variables. Change them one at a time when possible.

## 4. Retriever Evaluation

### Common failure modes

- **Misses:** relevant information is not retrieved.
- **Noise:** irrelevant chunks are retrieved and may distract the generator.

### Precision and recall

Let:
- `Relevant retrieved` = number of retrieved chunks judged relevant.
- `Relevant available` = number of all relevant chunks in the reference set.

```text
Precision@k = relevant retrieved in top-k / k
Recall@k    = relevant retrieved in top-k / relevant available
```

- **High recall:** the retriever finds most of the needed evidence.
- **High precision:** most retrieved chunks are useful.
- Recall and precision are reference-based: they require an annotation of what counts as relevant.

### Golden dataset design

Avoid relying only on exact chunk/document IDs. Chunking changes can alter IDs even when the evidence remains equivalent.

Prefer goldens containing:
- user question;
- expected answer or required facts;
- reference source/evidence, when available;
- optional relevant passages and metadata.

This makes evaluation more robust when chunk sizes or indexes change.

### Improving retrieval

Test and compare:
1. Baseline chunk size and overlap.
2. Larger or smaller chunks.
3. Different `top-k` values.
4. Alternative embedding models.
5. A cross-encoder reranker.
6. Hybrid retrieval (keyword + vector), where useful.

Compare retrieval quality alongside latency and cost; do not select a configuration based on one metric alone.

## 5. Generator Evaluation

### Two core failure modes

1. **Unfaithfulness / hallucination:** the answer states claims unsupported by the retrieved context.
2. **Irrelevance:** the answer is off-topic or fails to answer the question, even if its claims are grounded.

### Key metrics

- **Faithfulness:** Are the answer's factual claims supported by the retrieved context?
- **Answer relevancy:** Does the answer address the user's question?
- **Citation accuracy:** Do cited sources actually support the claims they accompany?
- **Correctness:** Does the answer match verified facts or an appropriate reference answer?

A grounded answer can still be wrong if the retrieved context is wrong. Faithfulness and correctness are therefore different checks.

### Practical generator improvements

- State that answers must be grounded in the provided context.
- Instruct the model to say when evidence is insufficient.
- Use consistent generation settings during comparisons.
- Inspect failed examples and improve the prompt or context selection.
- Re-run the same goldens after every change.

## 6. Pipeline-Level Evaluation: The RAG Triad

The commonly used RAG triad checks three relationships:

| Metric | Relationship checked | Diagnostic question |
|---|---|---|
| **Contextual relevancy** | Retrieved context ↔ question | Is the retrieved information relevant to the query? |
| **Faithfulness** | Answer ↔ retrieved context | Is the answer supported by the context? |
| **Answer relevancy** | Answer ↔ question | Does the answer address the question? |

Interpretation:
- Low contextual relevancy → investigate retrieval, chunking, `top-k`, embeddings, or reranking.
- Low faithfulness → investigate unsupported generation, ambiguous context, or prompt instructions.
- Low answer relevancy → investigate question understanding, prompt design, and answer focus.

**Do not confuse contextual relevancy with recall/precision.** They use different definitions and evaluation procedures. A judge may score semantic relevance, while precision/recall compare retrieved items against annotated relevant evidence.

## 7. LLM-as-a-Judge and Claim-Based Metrics

An LLM judge can evaluate semantic properties that are hard to measure with exact string matching.

A typical process:
1. Extract claims from the generated answer.
2. Check each claim against the supplied context or reference.
3. Aggregate the supported/unsupported judgments into a score.
4. Inspect borderline and failed cases.

### Limitations

- Judge outputs can vary across runs or models.
- Judges may favor longer answers, particular writing styles, or their own model family.
- Claim counting can misrepresent severity: one critical false claim may matter more than several minor correct claims.
- Use a clear rubric, calibrated examples, repeated trials where needed, and human review of a sample.

For reproducibility, record the judge model/version, prompt, metric settings, threshold, and evaluation dataset version.

## 8. Application-Level Evaluation

Beyond the RAG triad, evaluate whether the application succeeds for real users.

- **Correctness:** Is the response factually correct?
- **Completeness:** Does it cover all parts of the question?
- **Tone and style:** Is it clear, appropriate, and consistent with the product?
- **Citation quality:** Can users verify important claims?
- **Abstention:** Does it acknowledge missing evidence rather than inventing an answer?
- **Safety and privacy:** Does it avoid disclosing sensitive data or following malicious instructions?
- **Operations:** Does it meet latency, reliability, and cost targets?

### Safety and operational checks

Test prompt injection, jailbreak attempts, PII leakage, unauthorized data access, and unsafe requests. Measure latency (including percentiles such as p50/p95), error rate, token usage, and cost per request under realistic load.

## 9. Golden Dataset Creation

A useful RAG golden example can include:

```json
{
  "input": "How long is the course?",
  "expected_output": "The duration stated in the official course material.",
  "expected_facts": ["The verified duration"],
  "reference_context": "Relevant excerpt from the official source.",
  "metadata": {"topic": "course_duration"}
}
```

- **Hand-authored goldens:** usually more controlled and trustworthy, but costly to create.
- **LLM-assisted synthesis:** can expand coverage quickly, but generated examples must be reviewed and deduplicated.
- Include common questions, multi-part questions, ambiguous questions, unanswerable questions, edge cases, and adversarial inputs.
- Keep a stable test set for regression checks and a separate set for ongoing expansion.

## 10. DeepEval Essentials

DeepEval provides test cases and metrics for evaluating LLM applications.

Conceptually:
1. Create a test case containing input, actual output, expected output (if available), retrieval context, and relevant metadata.
2. Select metrics appropriate to the evaluation level.
3. Run the evaluation.
4. Review scores and individual failures.
5. Compare results across experiments.

Metric names and API parameters can change between versions; consult the current official documentation before copying code into a project.

## 11. Example Experiment Sequence

| Trial | Change | What to compare |
|---|---|---|
| Baseline | Current chunks, embeddings, and `top-k` | Establish initial scores |
| Trial 2 | Change chunk size/overlap | Recall, precision, contextual relevancy |
| Trial 3 | Add reranking | Retrieval quality, latency, cost |
| Trial 4 | Change embedding model | Retrieval metrics and total pipeline quality |
| Trial 5 | Refine generator prompt | Faithfulness, relevancy, correctness, completeness |

Treat any numeric results in a tutorial or demo as that experiment's results—not as a guaranteed outcome for another dataset or application.

## 12. Regression Testing and Continuous Improvement

After any change to prompts, models, chunking, embeddings, retrieval settings, or code:

1. Run the same versioned golden dataset.
2. Compare scores with the baseline.
3. Inspect regressions and critical failures.
4. Check safety, latency, reliability, and cost.
5. Block release if important metrics fall below agreed thresholds.
6. Deploy and monitor real usage.
7. Add reviewed production failures to the dataset.
8. Re-evaluate the next version.

This is a **self-improving evaluation loop**, but dataset additions and model changes should be reviewed rather than applied blindly.

## 13. Interview Answer: “How Do You Evaluate a RAG Chatbot?”

“I evaluate a RAG chatbot at three levels. First, at the **component level**, I measure retriever quality using metrics such as Recall@k and Precision@k, and evaluate the generator for faithfulness and answer relevancy. Second, at the **pipeline level**, I use the RAG triad—contextual relevancy, faithfulness, and answer relevancy—to find failures across retrieval and generation. Third, at the **application level**, I measure correctness, completeness, citations, tone, safety, latency, and cost. I build a reviewed golden dataset, use deterministic metrics where possible and an LLM judge for semantic checks, validate judge results with human review, and run regression evaluations in CI/CD after every change. In production, I monitor traces and failures, sample interactions for evaluation, and add reviewed failures to the test set.”

## 14. Quick Revision

- **Retriever:** Did the system find the right evidence?
- **Generator:** Did it use the evidence faithfully and answer the question?
- **Pipeline:** Do retrieval and generation work together?
- **Application:** Does the user receive a correct, complete, safe, useful response within operational limits?
- **Regression test:** Did a change improve one area while breaking another?
- **Best practice:** Combine metrics, human review, trace inspection, and production monitoring.

## Further Reading

- [DeepEval documentation](https://docs.confident-ai.com/)
- [Ragas documentation](https://docs.ragas.io/)
- [Chroma documentation](https://docs.trychroma.com/)
- [OpenAI text-embedding-3-large documentation](https://platform.openai.com/docs/guides/embeddings)
- [UV Python package and project manager](https://docs.astral.sh/uv/)
- [Supplied retriever/generator evaluation video](https://www.youtube.com/watch?v=9Dkz3ckRj8c)
- [Supplied generator and RAG pipeline video](https://www.youtube.com/watch?v=PATGn2XhmCY)
- [Supplied application metrics and G-Eval video](https://www.youtube.com/watch?v=nlyxlKD5cvU)
