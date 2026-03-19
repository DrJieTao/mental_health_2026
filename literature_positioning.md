# Literature Positioning — Multi-Agent Mental Health Detection

**Author**: Jie Tao, D.Sc.
**Date**: 2026-03-19 (updated from 2026-03-18)
**Scope**: Dec 2025 – Mar 2026 literature landscape (updated with deep research report on evaluation)
**Purpose**: Position the Decision-Review agent architecture for mental health classification within recent research. Intended for use in a Related Work / Literature Review section.
**Target venues**: HICSS, Information Systems Research, Information Systems Frontiers

> **Note**: All cited papers were verified as publicly accessible via web search as of March 2026. arXiv IDs and DOIs are provided where available.

---

## 1. Introduction: Three Research Dimensions

This work sits at the intersection of three active research dimensions:

1. **LLM-Based Mental Health Detection on Social Media** — the application domain, where recent surveys and benchmarks establish what has been achieved with single-model approaches.
2. **Multi-Agent LLM Systems and Iterative Feedback** — the methodological backbone, where debate, reflexion, and Socratic questioning have been validated in general reasoning but remain largely unapplied to clinical classification.
3. **Clinical Reasoning, Evaluation, and Process Compliance** — the quality assurance dimension, where recent findings reveal a "reasoning paradox" (structured reasoning improves accuracy but degrades consistency) that motivates explicit process auditing.

Since the initial positioning (March 18, 2026), the landscape has further crystallized with new mental health-specific multi-agent systems (AgentMental), structured debate architectures (D2D, PAMAS), and feedback evaluation frameworks (TED, CriticBench) that refine our understanding of each dimension. The following sections survey each dimension, then synthesize the gaps that this work uniquely fills.

---

## 2. LLM-Based Mental Health Detection on Social Media

### 2.1 Landscape

Large language models have rapidly become the dominant approach for mental health detection on social media, progressing from fine-tuned domain-specific transformers to general-purpose LLMs applied via prompting and in-context learning. The period Dec 2025 – Mar 2026 shows clear trends: (a) GPT-4 class models outperform fine-tuned BERT variants on mood disorder detection; (b) retrieval-augmented generation (RAG) achieves state-of-the-art on suicide detection; and (c) agent-based systems are identified as the "next frontier" but remain underexplored.

**Sánchez Rodríguez et al. (2026)** fine-tune MentalBERT on 152K Reddit posts with LIME explainability, achieving 82.2% accuracy on a 4-class mental health classification task (Scientific Reports, Jan 31, 2026). This represents the current best single-model approach for Reddit-based mental health detection but relies on static, single-pass classification with no iterative feedback or reasoning process auditing.

A **systematic review of 205 studies** published in MDPI Electronics (Jan 26, 2026) confirms that GPT-4 outperforms BERT-family models for mood disorder detection, and that RAG architectures achieve 90.7% accuracy on suicide detection tasks. Critically, the review identifies agent-based systems as an *emerging but underexplored* direction and calls for standardized evaluation frameworks.

**Cognitive-Mental-LLM** (arXiv:2503.10095, Jan 7, 2026) directly compares Chain-of-Thought (CoT), Self-Consistency CoT (SC-CoT), and Tree-of-Thought (ToT) reasoning strategies for mental health classification on Dreaddit and SDCNL datasets. While demonstrating that structured reasoning improves performance over direct prompting, all experiments use single-agent configurations with no external review or feedback loop. The paper highlights reliability challenges that multi-agent architectures could address.

**Ge et al. (2025)** provide a comprehensive survey spanning three paradigms for mental health detection — direct prompting, RAG, and agents (arXiv:2504.02800v4). They catalog existing agent systems (PsychoAgent, WellbeingAgent) but note these are "emerging but underexplored." No surveyed work implements a Decision-Review architecture with Socratic feedback for direct classification.

A **JMIR AI systematic review (2026)** covering 35 fine-tuning and 17 prompt engineering papers for healthcare text classification confirms that single-agent approaches dominate the field. The review highlights the HealthPrompt framework but finds no multi-agent feedback architectures in the healthcare text classification literature.

The **MIND-SAFE** framework utilizes layered prompts and multi-tier safety mechanisms to ensure that mental health diagnostic outputs are context-appropriate and safe. Here, the review function serves as an *ethical safeguard* — auditing for biases or acute risks — rather than a reasoning process auditor. This positions MIND-SAFE as an advanced prompt-engineering approach, not a multi-agent feedback architecture.

### 2.2 What's Missing

All surveyed approaches share three limitations:

- **Single-pass classification**: No iterative self-correction mechanism. Once a classification is produced, it stands.
- **No process auditing**: Explainability methods (LIME, attention visualization) are applied post hoc; they do not influence the classification process itself.
- **Uniform processing depth**: Every post receives identical computational resources regardless of clinical severity or classification difficulty.

---

## 3. Multi-Agent Feedback Systems

### 3.1 Landscape

Multi-agent debate and iterative feedback have been extensively validated in general reasoning tasks. The Dec 2025 – Mar 2026 period shows the first applications to mental health, though with important scope limitations.

**Mao et al. (2026)** present Confidence-Aware Fine-Grained Debate (CFD) among open-source LLMs for mental health and online safety data enrichment (arXiv:2512.06227v2, Mar 3, 2026). This is the **closest existing work** to ours. CFD uses multi-agent debate with confidence estimation on 350 Reddit posts. However, it targets *data enrichment* (generating labels for unlabeled data), not direct classification with iterative self-improvement. It lacks structured clinical reasoning protocols and does not audit the reasoning process.

**AgentMental** (arXiv:2508.11567, 2025) introduces an interactive multi-agent framework for mental health assessment with four specialized agents: a Question Generator (PHQ-8-based structured dialogue), an Evaluation Agent (adequacy and necessity analysis of user responses), a Scoring Agent (final psychological report), and an Updating Agent (tree-structured memory management). AgentMental is a significant multi-agent contribution to the mental health domain but differs fundamentally from our work in three respects: (1) it operates through *interactive clinical interview simulation*, not classification of existing social media posts; (2) its Evaluation Agent assesses *information sufficiency* of user responses, whereas our Review Agent audits the *reasoning process* of the Decision Agent; and (3) it has no Socratic feedback mechanism or process compliance auditing — its feedback is limited to triggering follow-up questions when responses are inadequate.

**MAR: Multi-Agent Reflexion** (Ozer et al., arXiv:2512.20845, Dec 23, 2025) separates acting, diagnosing, critiquing, and aggregating into distinct agent roles to prevent thought degeneration. Applied to HotPotQA and HumanEval (general reasoning), MAR shares the same structural pattern as our Decision-Review architecture — but operates outside the clinical domain. Our work adapts this pattern with domain-specific clinical reasoning rules, mandatory Devil's Advocate analysis, and clinical signal vocabularies.

**GroupDebate** (AAMAS 2026, Dec 26, 2025) introduces group-structured debate that achieves 51.7% token reduction while maintaining or improving accuracy. This efficiency-focused contribution is relevant for scaling multi-agent mental health systems but does not address domain-specific clinical reasoning.

**Holub et al. (2026)** integrate a Socratic questioning framework into educational AI (arXiv:2601.14798, Jan 22, 2026), demonstrating progressive questioning strategies that deepen understanding. This validates Socratic feedback as a mechanism for iterative improvement, though applied to pedagogy rather than classification.

**SSR: Socratic Self-Refine** (arXiv:2511.10621, Nov 2025) decomposes reasoning into verifiable (sub-question, sub-answer) pairs with step-level self-consistency checking for mathematical reasoning. SSR provides strong methodological precedent for Socratic self-correction, but uses a single agent performing self-refinement. Our work employs a *separate* Review Agent for the Socratic review, avoiding the self-consistency bias inherent in single-agent approaches.

**Baban et al. (2025)** propose a multi-agent collaboration framework leveraging BERT with five specialized agents (Lexical, Contextual, Logic, Consensus, Explainability) and threshold-based escalation (arXiv:2502.18653, Feb 2025). Applied to text classification (IMDB), it achieves +5.5% accuracy improvement. This ensemble-style approach differs from our structured Decision-Review loop — Baban et al. use consensus voting, while our Review Agent provides directed Socratic feedback targeting specific signal gaps and process violations.

**Debate-to-Detect (D2D)** reformulates diagnostic classification as a structured adversarial debate between agents with fixed stances, mediated by a Judge agent across five stages: Opening, Rebuttal, Free Debate, Closing, and Judgment. The tit-for-tat exchange facilitates stepwise evidence refinement and improves decision transparency. However, **ChatEval** (ICLR 2024) identifies the "spurious consensus problem" in multi-agent debate — agents can converge on an incorrect answer, entrenching error in subsequent rounds. ChatEval emphasizes diverse role prompts to mitigate homogeneous perspectives. Our architecture addresses spurious consensus through mandatory Devil's Advocate analysis and a minimum 3-iteration protocol, ensuring that the Decision Agent is *required* to consider alternative interpretations before finalizing its classification.

**PAMAS** (arXiv:2602.03158) introduces a perspective-aware multi-agent system for misinformation detection that addresses the "information drowning" problem — where deceptive cues are overwhelmed by normal signals in full-context exposure. PAMAS organizes agents into hierarchical roles (Auditors, Coordinators, Decision-Maker) with targeted correction strategies for self-improvement. Our curation pipeline addresses information drowning differently: severity-weighted pre-filtering concentrates analysis on the most clinically salient posts rather than distributing perspectives across all content.

### 3.2 Domain Gap

Multi-agent debate and Socratic feedback are proven effective in general reasoning (math, QA, code generation). The application to mental health remains nascent:

- CFD (Mao et al., 2026) brings multi-agent debate to the mental health domain but only for data enrichment, not direct classification with iterative feedback.
- AgentMental (2025) brings multi-agent architecture to mental health *assessment* but targets interactive PHQ-8 interview simulation, not direct post-level classification from existing social media text.
- Multi-agent debate systems face known challenges including spurious consensus (ChatEval, ICLR 2024) and premature convergence that our architecture mitigates through mandatory Devil's Advocate analysis and a minimum 3-iteration protocol.
- No existing work combines a Decision-Review feedback loop with clinical process auditing for direct mental health post classification.
- No existing work applies Socratic feedback (signal gaps + process errors) to iterative clinical text classification.

---

## 4. Clinical Reasoning and Process Compliance

### 4.1 The Reasoning Paradox

Recent research reveals a critical paradox: structured reasoning (Chain-of-Thought, Tree-of-Thought) improves accuracy on clinical tasks but *degrades consistency* and can even *worsen* overall performance in certain configurations.

**"Why Chain of Thought Fails in Clinical Text Understanding"** (arXiv:2509.21933, submitted ICLR 2026, late 2025) evaluates 95 LLMs on 87 clinical tasks and finds that 86.3% of LLM-task pairs suffer CoT degradation. Identified failure modes include hallucination, omission, and reasoning drift. This finding directly motivates our structured protocol: mandatory Devil's Advocate analysis, 3-iteration minimum, explicit process rules — all designed to constrain reasoning within clinically productive bounds.

**"Can Reasoning LLMs Enhance Clinical Document Classification?"** (Springer, 2026) tests 8 LLMs on ICD-10 classification and finds that reasoning models improve accuracy by ~3% but are *less consistent* (84% vs. 91% consistency rates). This accuracy-consistency paradox is exactly what our Review Agent addresses: by explicitly auditing *process consistency* alongside outcome correctness, the system can detect and correct reasoning drift.

### 4.2 Process Verification

**Process Reward Models That Think** (arXiv:2504.16828, Dec 2025) introduces PRMs with reasoning chains before step-level rewards, enhancing verification in mathematical reasoning. Our Review Agent functions analogously as a *generative process reward model* — it reasons through the Decision Agent's trajectory before issuing approval or rejection, but operates in natural language rather than scalar rewards.

The **npj Digital Medicine (2026)** scalable framework for evaluating health language models establishes multi-dimensional evaluation as a necessity for clinical AI. Our four-rule process audit (iteration count, Devil's Advocate compliance, prior evaluation consideration, feedback integration) provides a concrete, domain-specific implementation of structured clinical evaluation.

**MindBench.ai** (Nature Portfolio, 2025) offers an open evaluation platform for mental health AI performance and profiling. Our system's built-in process compliance metrics (process_valid rate, APPROVED rate, per-round improvement deltas) align with MindBench's goals of transparent, actionable evaluation and could serve as a contribution to the mental health AI evaluation ecosystem.

Recent studies on **process vs. outcome review** in medical diagnostic reasoning demonstrate that reviewing the reasoning *procedure* — the step-by-step logic — reduces incorrect knowledge by 63%, whereas reviewing only the final diagnosis yields much smaller gains. This finding directly validates our design choice: the Review Agent audits the Decision Agent's reasoning trajectory (Devil's Advocate execution, prior evaluation consideration, feedback integration), not merely the final classification label.

**Pathology-CoT** (arXiv:2510.04587) provides a template for process-level supervision by converting expert viewing behaviors into standardized behavioral commands, enabling review of an agent's "attention" during reasoning. This approach prevents "hallucinated correctness" — where an agent arrives at the right answer for the wrong reasons. Our Review Agent performs an analogous function: it can flag a classification as process-invalid even when the label is correct, if the Decision Agent's reasoning chain shows it focused on irrelevant signals while ignoring clinically salient ones.

**AgentDebug** enables agents to recover from failures by grounding an error taxonomy in real-world diagnostic traces, resulting in up to 26% improvement in task success across complex environments. This validates the trajectory-grounded feedback approach that our Review Agent implements: feedback is generated from analysis of the Decision Agent's full reasoning trajectory, not from the final classification alone.

### 4.3 Feedback Quality and Actionability

The quality and actionability of review feedback is itself a subject of recent research, with direct implications for how Decision-Review architectures generate improvement.

The **TED (Talk, Evaluate, Diagnose)** framework (arXiv:2603.15483) introduces automated error analysis that bridges evaluation and diagnosis. TED clusters identified errors into categories (tool-signature mismatches, logical inconsistencies, persona-alignment failures) and translates them into specific remedies, achieving 8–10% performance gains. Our Review Agent's structured feedback fields — `signal_gap` (missed clinical signals) and `process_error` (protocol violations) — function analogously to TED's error categorization, enabling targeted correction rather than generic re-evaluation.

The broader **Agent-as-a-Judge** paradigm (arXiv:2601.05111) moves beyond simple correctness evaluation to encompass holistic analysis of conversation quality, turn efficiency, and systematic error diagnosis. Our Review Agent embodies this paradigm: it evaluates not just whether the Decision Agent's classification is correct, but whether its reasoning process was clinically sound, complete, and protocol-compliant.

The **ReviewAgent** framework demonstrates that automated review can generate actionable insights 47.62% more effectively than expert human reviews, validating the scalability of structured review loops. Meanwhile, **CriticBench** — a comprehensive benchmark for critique abilities — reveals an important challenge: the "saturation of review," where as the decision agent's initial output improves, the review agent finds it increasingly difficult to provide meaningful feedback. Our adaptive processing depth directly addresses this — the screening pass identifies high-confidence, likely-correct classifications that bypass deep review, allocating review resources to cases where the Review Agent can have the greatest marginal impact.

**Criteria Match Validator Agents (CMVA)** verify outputs against predefined formal criteria such as temporal logic formulas or rubric-based checklists, decoupling validation logic from stochastic output. Our four-rule process audit (iteration count, Devil's Advocate compliance, prior evaluation consideration, feedback integration) is a domain-specific implementation of this principle, providing automated validation that is both robust and interpretable.

### 4.4 Implication for This Work

The clinical reasoning literature makes a strong case that unstructured LLM reasoning is *insufficient* for clinical text classification. Our response is fourfold:

1. **Structured reasoning protocol**: The Decision Agent follows a mandatory 3-iteration evaluation with Devil's Advocate analysis, not open-ended chain-of-thought.
2. **External process auditing**: The Review Agent validates adherence to the protocol, catching reasoning drift, omission, and hallucination — consistent with evidence that process-level review reduces incorrect knowledge by 63% compared to outcome-only review.
3. **Feedback-driven correction**: When process violations are detected, Socratic feedback targets the specific failure mode rather than requesting generic re-evaluation.
4. **Structured feedback categorization**: Feedback is delivered through typed fields (`signal_gap`, `process_error`) that categorize the type of deficiency, enabling targeted correction — consistent with the error categorization approach validated by TED and the actionability principle from ReviewAgent.

---

## 5. Positioning Summary Table

| Dimension | This Work | Sánchez Rodríguez et al. (2026) | Mao et al. CFD (2026) | AgentMental (2025) | Cognitive-Mental-LLM (2026) | MAR (2025) | D2D / PAMAS | "Why CoT Fails" (2025) |
|---|---|---|---|---|---|---|---|---|
| **Domain** | Mental health (Reddit, ANGST 4-class) | Mental health (Reddit, 4-class) | Mental health (Reddit, data enrichment) | Mental health (PHQ-8 interview) | Mental health (Dreaddit, SDCNL) | General QA/Code | Misinformation / general debate | Clinical NLP (87 tasks) |
| **Architecture** | Multi-agent (Decision + Review) | Single model (MentalBERT) | Multi-agent debate | Multi-agent (4 specialized roles) | Single agent (CoT/ToT) | Multi-agent (4 roles) | Multi-agent adversarial debate / hierarchical aggregation | Evaluation study |
| **Feedback mechanism** | Socratic (signal gap + process error), iterative | None | Confidence-aware debate | Adequacy-driven follow-up questions | None | Reflexion loop | Adversarial debate / perspective aggregation | N/A |
| **Process auditing** | 4-rule compliance check during classification | Post-hoc LIME | None | Information sufficiency check | None | Critique role (general) | Judge-based stage evaluation | Identifies failure modes |
| **Clinical reasoning structure** | Mandatory Devil's Advocate, 3-iteration protocol | None | None | PHQ-8 structured interview protocol | Compared CoT/ToT strategies | Domain-agnostic | Domain-agnostic debate stages | Documents CoT failures |
| **Processing depth** | Adaptive (screening → curation → deep analysis) | Uniform | Uniform | Adaptive (adequacy-triggered follow-ups) | Uniform | Uniform | Uniform (fixed debate rounds) | N/A |
| **Ground truth usage** | Review Agent only (Decision Agent is blinded) | Training labels | Confidence calibration | Clinical scoring rubric | Training/evaluation | N/A | Claim-level ground truth | N/A |
| **Fine-tuning required** | No (pure ICL) | Yes (MentalBERT) | No | No | Compared both | No | No | N/A |

---

## 6. Gap Analysis: What This Work Uniquely Contributes

### Gap 1: No Decision-Review Architecture for Direct Mental Health Post Classification

Mao et al. (Mar 2026) apply multi-agent debate to mental health data but for *label generation* (data enrichment), not direct classification with iterative Decision-Review feedback. AgentMental (arXiv:2508.11567) introduces a multi-agent framework for mental health *assessment* but operates through interactive PHQ-8-guided dialogue, not direct classification of existing social media posts — its Evaluation Agent checks information adequacy of user responses, not reasoning process compliance. Ge et al.'s survey identifies agents as "emerging but underexplored" without finding any Decision-Review implementation for mental health post classification. **This work is the first to implement a Decision-Review feedback loop for direct mental health post classification**, where the Review Agent audits both the classification outcome and the reasoning process, and Socratic feedback drives iterative improvement on existing social media text rather than interactive dialogue.

### Gap 2: No Process Auditing in Mental Health NLP

Sánchez Rodríguez et al. (Jan 2026) provide post-hoc explainability (LIME) but no *process compliance* enforcement during classification. Cognitive-Mental-LLM evaluates reasoning strategies but doesn't validate the reasoning process itself. "Why CoT Fails" (2025) demonstrates that unstructured reasoning degrades clinical performance in 86.3% of cases. Recent research confirms that process-level review (auditing reasoning steps) reduces incorrect knowledge by 63%, far exceeding outcome-only review — yet no mental health NLP system implements this approach. **This work introduces four-rule process compliance auditing** — iteration count, Devil's Advocate execution, prior evaluation consideration, and feedback integration — enforced *during* classification by a separate Review Agent, implementing the process-level review paradigm validated in broader clinical AI research.

### Gap 3: No Socratic Feedback for Clinical Classification

SSR (Nov 2025) validates Socratic questioning for mathematical reasoning. Holub et al. (Jan 2026) apply it to educational AI. The TED framework (arXiv:2603.15483) demonstrates that categorized error analysis with specific remedies achieves 8–10% performance gains, and ReviewAgent shows that structured automated review produces 47.62% more actionable insights than expert human reviews. Yet no prior work applies structured, categorized Socratic feedback (signal gaps + process errors) to iterative clinical text classification. **This work adapts Socratic feedback to clinical signal analysis** — feedback is blinded to ground truth labels, delivered through typed fields (`signal_gap`, `process_error`), and expressed in clinical terms (hopelessness, hypervigilance, catastrophizing, social withdrawal) to guide the Decision Agent toward overlooked clinical signals.

### Gap 4: Adaptive Processing Depth via Screening-Curation-Deep Analysis

All existing mental health classification systems apply uniform computational depth across all posts. CriticBench reveals a "saturation of review" problem — as the decision agent's initial output improves, the review agent finds it increasingly difficult to provide meaningful feedback. **This work introduces a two-pass batch pipeline** with severity-weighted curation (recency + F2-scored impact), allocating deeper multi-round analysis exclusively to clinically important or difficult cases. This directly addresses both the practical constraint that full multi-agent feedback loops are expensive and the review saturation problem — the pipeline screens all posts cheaply (Pass 1), bypasses deep review for high-confidence classifications, and invests additional rounds only where the Review Agent can have the greatest marginal impact (Pass 2).

---

## 7. Positioning Statement

> Recent surveys identify three paradigms for LLM-based mental health detection: direct prompting, retrieval-augmented generation, and agent-based systems (Ge et al., 2025; MDPI Systematic Review, 2026). While the first two are well-explored, agent-based approaches remain nascent. The closest work — Mao et al. (2026) — applies multi-agent debate to mental health *data enrichment* but does not address direct post classification or process compliance. AgentMental (2025) introduces multi-agent mental health *assessment* via interactive PHQ-8 dialogue but does not address direct post-level classification or reasoning process auditing. Meanwhile, research on clinical reasoning reveals a paradox: structured reasoning (CoT) improves accuracy but *reduces consistency* (Reasoning LLMs for Clinical Classification, 2026) and can even *degrade* performance in 86% of LLM-clinical task pairs (Why CoT Fails, 2025), while process-level review reduces incorrect knowledge by 63% compared to outcome-only review. This paper addresses these gaps by introducing a Decision-Review agent architecture with Socratic feedback, where a Review Agent audits not just classification correctness but adherence to a structured clinical reasoning protocol — mandatory Devil's Advocate analysis, iterative signal evaluation, and explicit feedback integration — to achieve both accuracy *and* process consistency.

---

## 8. Suggested Citation Groups

### A. LLM-Based Mental Health Detection

| Citation | Year | Relevance |
|----------|------|-----------|
| Sánchez Rodríguez et al. — "Understanding mental health discourse on Reddit with transformers and explainability" (Scientific Reports) | 2026 | Current best single-model Reddit MH classification |
| MDPI Electronics — "A Systematic Review of LLMs in Mental Health" (205 studies) | 2026 | Comprehensive survey; identifies agent gap |
| Cognitive-Mental-LLM — "Evaluating Reasoning in LLMs for Mental Health Prediction" (arXiv:2503.10095) | 2026 | Compares CoT/SC-CoT/ToT for MH classification |
| Ge et al. — "Survey: From LLMs and RAG to Agents for Mental Disorder Detection" (arXiv:2504.02800v4) | 2025 | Agent paradigm survey; identifies gap we fill |
| JMIR AI — "LLMs for Health Care Text Classification: Systematic Review" | 2026 | Healthcare text classification landscape |
| Mental-LLM (Xu et al.) — LLM evaluation on Reddit mental health | 2024 | Baseline LLM evaluation for Reddit MH |
| MIND-SAFE — Layered prompts and multi-tier safety mechanisms for MH detection | 2025 | Safety-first prompt engineering; review as ethical safeguard |

### B. Multi-Agent Feedback and Debate

| Citation | Year | Relevance |
|----------|------|-----------|
| Mao et al. — "Automated Data Enrichment using CFD among Open-Source LLMs" (arXiv:2512.06227v2) | 2026 | **Closest competitor** — multi-agent debate for MH data enrichment |
| Ozer et al. — "MAR: Multi-Agent Reflexion" (arXiv:2512.20845) | 2025 | Role separation (act/diagnose/critique/aggregate) |
| GroupDebate — "Enhancing Efficiency of Multi-Agent Debate" (AAMAS 2026) | 2025 | Efficient debate structures |
| Holub et al. — "Reflecting in the Reflection: Integrating a Socratic Questioning Framework" (arXiv:2601.14798) | 2026 | Socratic questioning for iterative improvement |
| SSR: Socratic Self-Refine (arXiv:2511.10621) | 2025 | Step-level Socratic self-correction |
| Baban et al. — "Multi-Agent Collaboration Framework Leveraging BERT" (arXiv:2502.18653) | 2025 | Multi-agent text classification (ensemble style) |
| AgentMental — "Interactive Multi-Agent Framework for Mental Health Assessment" (arXiv:2508.11567) | 2025 | Multi-agent MH assessment via PHQ-8 interview; key comparator |
| D2D (Debate-to-Detect) — Structured adversarial debate for diagnostics | 2025 | 5-stage debate format; spurious consensus mitigation |
| ChatEval — "Towards Better LLM-based Evaluators through Multi-Agent Debate" (ICLR 2024) | 2024 | Spurious consensus problem; diverse role prompts |
| PAMAS — "Self-Adaptive Multi-Agent System with Perspective Aggregation" (arXiv:2602.03158) | 2026 | Information drowning mitigation; hierarchical aggregation |

### C. Clinical Reasoning and Process Evaluation

| Citation | Year | Relevance |
|----------|------|-----------|
| "Why Chain of Thought Fails in Clinical Text Understanding" (arXiv:2509.21933) | 2025 | 86.3% CoT degradation in clinical NLP |
| "Can Reasoning LLMs Enhance Clinical Document Classification?" (Springer) | 2026 | Accuracy-consistency paradox |
| Process Reward Models That Think (arXiv:2504.16828) | 2025 | Step-level reasoning verification |
| npj Digital Medicine — "A Scalable Framework for Evaluating Health Language Models" | 2026 | Multi-dimensional health LLM evaluation |
| MindBench.ai — "An Actionable Platform to Evaluate LLMs in Mental Healthcare" (Nature Portfolio) | 2025 | Mental health AI evaluation platform |
| TED — "Talk, Evaluate, Diagnose: User-aware Agent Evaluation with Automated Error Analysis" (arXiv:2603.15483) | 2026 | Error categorization → specific remedies; 8–10% gains |
| Agent-as-a-Judge — "A Survey on Agent-as-a-Judge" (arXiv:2601.05111) | 2026 | Holistic quality evaluation beyond correctness |
| ReviewAgent — Automated review with actionable insights metric | 2025 | 47.62% improvement in actionable insights over human reviews |
| CriticBench — "Evaluating Large Language Models as Critic" | 2025 | GQC benchmark; reveals "saturation of review" problem |
| Process vs. Outcome Review — Medical diagnostic reasoning studies | 2025 | Process-level review reduces incorrect knowledge by 63% |
| Pathology-CoT — "Learning Visual Chain-of-Thought from Expert Diagnosis Behavior" (arXiv:2510.04587) | 2025 | Process-level supervision; prevents "hallucinated correctness" |
| AgentDebug — Trajectory-grounded error diagnosis for agent recovery | 2025 | Up to 26% improvement via diagnostic trace analysis |
| CMVA (Criteria Match Validator Agent) — Formal criteria verification | 2025 | Decouples validation from stochastic output |

### D. Foundational References (from project brief)

| Citation | Year | Relevance |
|----------|------|-----------|
| Zhang et al. — KETCH model for suicidal ideation detection (ISR) | 2024 | Directly foundational; clinical lexicons |
| Ji et al. — Transformer models for suicidal ideation detection | 2023 | State-of-the-art benchmark |
| Tadesse et al. — MentalBERT | 2022 | Baseline comparison |
| Chancellor et al. — GPT-3.5 for depressive symptom detection | 2023 | Related LLM work |
| Min et al. — In-context learning vs. fine-tuning | 2022 | Methodological basis for ICL approach |
| Wei et al. — Domain-specific prompting in GPT-4 | 2023 | Supports ICL approach |
| Amann et al. — Human-AI collaboration transparency | 2023 | Motivates Review Agent design |

### E. Future Directions (training loop and workflow formalization)

| Citation | Year | Relevance |
|----------|------|-----------|
| MultiCritique — "Training Language Models to Critique with Multi-Agent Feedback" | 2025 | SFT + RL using multi-agent feedback; annotation-free training loop |
| Reflexion — "Verbal Reinforcement Learning" | 2025 | Episodic memory of self-reflections; 22% improvement without weight updates |
| Constrained Process Maps — "Multi-Agent Generative AI Workflows" (arXiv:2602.02034) | 2026 | DAG-MDP formalization; reduces human review by 85x |

---

## 9. Related Work Section Outline (suggested structure)

For a manuscript targeting HICSS / ISR / ISF, the Related Work section could be organized as:

### 2.1 LLM-Based Mental Health Detection on Social Media
- Evolution from fine-tuned transformers (MentalBERT) to general LLMs (GPT-4, Claude)
- Current state: single-model approaches dominate (Sánchez Rodríguez et al., 2026; JMIR AI, 2026)
- Safety-aware prompt engineering emerging (MIND-SAFE, 2025)
- Surveys identify agent-based systems as the next frontier (MDPI, 2026; Ge et al., 2025)
- **Transition**: while detection accuracy has improved, existing approaches lack iterative self-correction and process transparency

### 2.2 Multi-Agent Systems for Text Classification
- Multi-agent debate improves reasoning quality (GroupDebate, 2025; MAR, 2025)
- Structured adversarial debate with known challenges: spurious consensus (ChatEval, 2024), information drowning (PAMAS, 2026)
- Socratic feedback mechanisms validated in education and math (Holub et al., 2026; SSR, 2025)
- Mental health-specific multi-agent systems emerge: AgentMental for interactive PHQ-8 assessment (2025), CFD for data enrichment (Mao et al., 2026)
- **Transition**: these techniques have not been applied to direct post-level clinical classification with process compliance auditing

### 2.3 Clinical Reasoning and the Need for Process Compliance
- The reasoning paradox: CoT helps accuracy but hurts consistency (Why CoT Fails, 2025; Reasoning LLMs for Clinical Classification, 2026)
- Process-level review reduces incorrect knowledge by 63%, far exceeding outcome-only review
- Process reward models as a verification mechanism (Process Reward Models That Think, 2025)
- Feedback quality frameworks: error categorization (TED, 2026), actionable insights (ReviewAgent), and the "saturation of review" problem (CriticBench)
- Evaluation frameworks for health AI (npj Digital Medicine, 2026; MindBench.ai, 2025)
- **Transition**: the literature calls for structured reasoning with external verification and actionable feedback — exactly what our Decision-Review architecture provides

### 2.4 Positioning of This Work
- Synthesize gaps 1–4 from Section 6 above
- State contributions explicitly against the closest comparators (CFD, AgentMental, D2D/PAMAS)
