- Model Evals:
    - to measure the capabilites of llm
    - decide what to measure
    - get a test: A publich benchmark, own eval set
    - run the model on the test
    - score and interprete

- LLM benchmarks:
    - 8 core capebilities (MMLU)
    1. knowledge and reasoning
    2. Instruction following
    3. sefty and alignmment
    4. Agentic & tool use
    5. Vision & Multimodal
    6. Long context
    7. Mathematics
    8. Coding & SWE

- Benchmarks:
    - all details on this
    - the dataset
    - the running config
    - scoring method
    - Aggrigation
    - different prompt technique
    - pass@k
    - maj@k
    - Benchmark is a Exam paper and Evaluation harness is complete exam administration system

    - EX- GSM8k, MMLU and some more recents 

    -lm-evaluation-harness by EleutherAi
    - Deepleval

- problems on benchmark

- a detailed analysis on benchmark on all 8 capabilities

- https://benchwiki.vercel.app/

- Defining the LLM Benchmark: The Standardized Model Exam
01:45 - The 4 Core Architectural Components of Every Global AI Test
02:30 - Dissecting the GSM8K (Grade School Mathematics) Dataset
04:20 - Live Demo: Sourcing the GSM8K Repository on Hugging Face
05:30 - Component 2: Breaking Down Run Configurations & Guidelines
06:12 - Prompt Construction: Zero-Shot vs. Few-Shot (8-Shot Prompting)
08:24 - Enforcing Logic: Activating Chain of Thought (CoT) Frameworks
09:10 - Parameter Controls: Locking Temperatures & Managing Token Budgets
10:05 - Component 3: Evaluating Strict Pass@1 vs. Lenient Pass@K Scoring
11:18 - Statistical Certainty: Implementing Majority@K (Mode Selection)
12:15 - Tool Integration Constraints: Allowing Web Browsing & Code Interpreters
13:34 - Component 4: Scoring Mechanisms (Regex Parsing vs. Model Graded)
15:53 - Component 5: Data Aggregation Methods (Simple Means vs. Weighted Means)
17:46 - Exploring the Origins of AI Exams: Peer-Reviewed Research Papers
21:15 - Execution: How an Evaluation Harness Acts as the Exam Administrator
26:40 - The Plumbing Problem: Handling Rate Limits, Retries, and Token Batching
29:00 - Live Python Demo: Configuring the EleutherAI LM Evaluation Harness
30:35 - Executing the Code Command: Tracking Latency & Parsing JSON Trace Logs
32:40 - Reviewing the Output: Interpreting a 90% Accuracy Mean on 20 Samples
35:36 - The Flaws of AI Testing: Why You Cannot Blindly Trust Leaderboards
43:10 - Pitfall 1: Benchmark Contamination & Scraped Pre-Training Memorization
45:50 - Pitfall 2: Benchmark Saturation & Statistical Score Clustering
48:10 - Pitfall 3: Configuration Gaming & Concealing Domain-Specific Failures
50:45 - Final Summary: Implementing Your Own Validation Frameworks

The 8 Core LLM Capabilities & Structuring Benchmark Study
04:22 - What is Knowledge Capability? (Parametric Memory)
07:32 - 2020: MMLU Arrives (Testing Breadth Across 57 Subjects)
10:11 - The First Branch: Testing Reliability with TruthfulQA
14:12 - The Second Branch: Human Exam Baselines with AGI Eval
16:47 - The Third Branch: Depth of Knowledge with GPQA
18:28 - Fixing Flaws: Upgrading to MMLU Pro (10 Options & Reasoning)
20:01 - The Modern Frontier: Humanity's Last Exam (HLE) & SimpleQA
24:00 - Summary Roadmap of the 7 Key Knowledge Benchmarks
25:35 - Introducing BenchWiki: Our Open Benchmark Database
28:47 - Deep Dive 1: MMLU Architecture, Log-Likelihoods & Saturated Limits
43:23 - Deep Dive 2: TruthfulQA, Misconceptions, & Alignment-Stage Contamination
57:46 - Deep Dive 3: AGI Eval, Bilingual Exams, & Human vs. LLM Baselines
01:05:38 - Deep Dive 4: GPQA (Google-Proof Q&A) & The Diamond Dataset
01:16:46 - Deep Dive 5: MMLU Pro (10 Options, 14 Disciplines, & Reasoning Bias)
01:27:04 - Deep Dive 6: SimpleQA (Open-Ended Short Answers & Calibration Metrics)
01:39:29 - Deep Dive 7: Humanity's Last Exam (Breadth x Depth, Multimodality, & Private Sets)
01:49:21 - Conclusion & Next Steps in the Series