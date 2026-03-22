# Paper Outline: The Dark Side of Multi-Agent Clinical AI
## Failure Modes and Design Principles from a Real Decision-Review Architecture for Mental Health Classification

**Target venue**: HICSS 2027 Minitrack on AI in Healthcare / Intelligent Systems in Healthcare
**Format**: 10-page HICSS paper (~8,000 words body text, excluding references)
**Tone**: Prescriptive and empirically grounded — not apologetic. This is "here is what the field needs to know" not "here is where our system failed."
**Framing anchor**: The paper is a contribution of: (1) a taxonomized failure mode catalog with empirical evidence, and (2) generalizable design principles derived from those failures. The system is the vehicle, not the subject.

---

## Framing Note for the Writer

The central intellectual claim is: **multi-agent systems introduce a new class of failure modes that cannot be diagnosed from outcome metrics alone, and that process-level auditing — while necessary — is also insufficient when the audit itself is misconfigured.** The contribution is prescriptive: six failure modes, each with a generalizable design principle, derived from 22 sessions of systematic empirical probing.

Frame the experiment progression as TED-style systematic red-teaming: each experiment was designed to probe a specific hypothesis, and fixing one failure mode consistently revealed the next. This is not a story of a broken system — it is a story of rigorous, iterative discovery.

---

## §1. Introduction (~700 words)

### 1.1 Opening Hook: The Reasoning Paradox (100 words)

Open with the paradox that motivates the entire paper. Structured reasoning (Chain-of-Thought, Tree-of-Thought) improves clinical classification accuracy in 86.3% of LLM-clinical task pairs but *also* reduces consistency — 84% vs. 91% consistency rates when reasoning models are applied to clinical document classification (Reasoning LLMs for Clinical Classification, Springer 2026). More damaging: "Why Chain of Thought Fails in Clinical Text Understanding" (arXiv:2509.21933) shows 86.3% of LLM-clinical task pairs suffer CoT degradation including hallucination, omission, and reasoning drift. Multi-agent systems with dedicated review agents were proposed as the answer — external process auditing to catch what self-reasoning misses.

**Framing note**: Do not open with "AI in mental health is important." Open with the paradox directly. This signals the paper is analytically sophisticated.

### 1.2 The Multi-Agent Promise and the Empirical Gap (200 words)

- Multi-agent systems are identified as the "next frontier" for mental health detection (MDPI Systematic Review, 2026; Ge et al., 2025), but remain underexplored.
- Process-level review reduces incorrect knowledge by 63% compared to outcome-only review (Process vs. Outcome Review, 2025) — strong theoretical motivation.
- Yet: no prior work has systematically characterized what *goes wrong* inside a real multi-agent clinical AI system during extended operation. Frameworks exist (TED, PAMAS, MIND-SAFE), but empirical failure mode catalogs from production systems do not.
- The field lacks the vocabulary, taxonomy, and empirical grounding to design robust multi-agent clinical AI. This paper provides all three.

### 1.3 This Paper (200 words)

**Contribution statement** (use this language precisely):

We present an empirically grounded taxonomy of six failure modes discovered through 22 sessions of systematic evaluation of a Decision-Review multi-agent architecture for mental health classification on social media. Our system — a Decision Agent (DA) performing iterative clinical reasoning with mandatory Devil's Advocate analysis, a Review Agent (RA) auditing that reasoning against a four-rule process protocol, and a cross-batch curation pipeline — was developed and evaluated on the ANGST 4-class dataset (100 posts, 5 full evaluation runs). The experiment progression was structured as systematic adversarial probing: each run was designed to test whether fixing the current most prominent failure produced a genuinely clean system, or revealed the next layer of failure. It always revealed the next layer.

From this process, we derive six failure modes organized across three architectural scopes — single-agent self-correction, multi-agent interaction, and system-level behavior — and map each to a generalizable design principle applicable to any multi-agent clinical AI system.

### 1.4 Paper Organization (100 words)

Standard roadmap paragraph. Note explicitly that the paper's organizing principle is from failure observation to generalizable principle — not from system description to results.

### Suggested Figure

**Figure 1**: Conceptual diagram showing the three-level failure mode taxonomy (pyramid or layered rectangle): base = single-agent failures (FM-1), middle = multi-agent interaction failures (FM-2, FM-3, FM-4), top = system-level failures (FM-5, FM-6). Label each level with its defining characteristic. This figure should appear early — anchor all subsequent discussion.

---

## §2. Related Work (~1,400 words total, three subsections)

**Framing note for the writer**: Each subsection must end with a clear GAP statement that this paper addresses. The subsections are not background — they are the argument for why this paper is needed.

### §2.1 AI/Agent Advancement in Mental Health Detection (~450 words)

**Claim**: Single-model approaches dominate; agent-based systems are identified as next but critically underexplored; no prior work has characterized what goes wrong inside them.

**Evolution trajectory** (build this as a narrative arc, not a list):

- *Lexicon/keyword era*: Rule-based detection using clinical vocabulary lists. Recall-oriented, not scalable.
- *Fine-tuned transformer era*: MentalBERT (Tadesse et al., 2022) — domain-adapted BERT on Reddit mental health text. Single-pass, no iterative correction. Sánchez Rodríguez et al. (2026) represent the current best: MentalBERT on 152K Reddit posts with LIME explainability, 82.2% accuracy on 4-class classification (Scientific Reports). Static, single-pass, no process auditing.
- *General LLM prompting era*: Chancellor et al. (2023) — GPT-3.5 few-shot for depressive symptom detection. Cognitive-Mental-LLM (arXiv:2503.10095, 2026) compares CoT/SC-CoT/ToT for mental health classification — finds structured reasoning helps but uses single-agent configurations only.
- *Agent era (nascent)*: MDPI systematic review of 205 studies (2026) identifies agents as emerging but underexplored. Ge et al. (2025) catalog PsychoAgent, WellbeingAgent — note these are emerging. JMIR AI systematic review (2026) covering 35 fine-tuning and 17 prompt engineering papers finds no multi-agent feedback architectures for healthcare text classification.

**Closest multi-agent work**:
- Mao et al. (2026) — CFD (Confidence-Aware Fine-Grained Debate) among open-source LLMs on 350 Reddit posts. Target: *data enrichment* (label generation), not direct classification with iterative feedback. Lacks structured clinical reasoning protocols and does not audit the reasoning process.
- AgentMental (arXiv:2508.11567, 2025) — four-agent framework (Question Generator, Evaluation Agent, Scoring Agent, Updating Agent) for PHQ-8-based interactive clinical interview simulation. Its Evaluation Agent checks *information adequacy* of user responses; our Review Agent audits the *reasoning process* of the Decision Agent. No Socratic feedback, no process compliance auditing.

**Gap statement**: No multi-agent feedback architecture for *direct* mental health post classification with *process auditing* has been described or empirically evaluated. More critically, no work has characterized what failure modes emerge when such a system is deployed at scale — leaving the field without the diagnostic vocabulary to build robust systems.

**Key citations**: Ge et al. (2025), MDPI Electronics (2026), Sánchez Rodríguez et al. (2026), Mao et al. (2026), AgentMental (2025), Cognitive-Mental-LLM (2026), JMIR AI (2026)

### §2.2 Process vs. Outcome Review in Clinical AI (~500 words)

**Claim**: Theory strongly motivates process-level review; our work reveals the *boundary conditions* under which that theory holds — and critically, where it breaks down.

**The reasoning paradox** (lead with this):
- "Why Chain of Thought Fails in Clinical Text Understanding" (arXiv:2509.21933, ICLR 2026 submission) evaluates 95 LLMs across 87 clinical tasks; 86.3% of LLM-task pairs show CoT degradation — hallucination, omission, and reasoning drift. This motivates external process auditing.
- "Can Reasoning LLMs Enhance Clinical Document Classification?" (Springer, 2026): reasoning models improve accuracy ~3% but are *less consistent* — 84% vs. 91% consistency rates.

**Process vs. outcome review theory**:
- Recent studies in medical diagnostic reasoning: reviewing the reasoning *procedure* — step-by-step logic — reduces incorrect knowledge by 63%, far exceeding outcome-only review. This directly validates our Review Agent's design: it audits the Decision Agent's reasoning *trajectory* (Devil's Advocate execution, prior evaluation consideration, feedback integration), not just the final classification label.
- Pathology-CoT (arXiv:2510.04587) provides a template: converting expert viewing behaviors into standardized behavioral commands enables detection of "hallucinated correctness" — arriving at the right answer for the wrong reasons. Our Review Agent performs the analogous function: it can flag a classification as process-invalid even when the label is correct, if the reasoning focused on irrelevant signals.

**Implementation analogs**:
- CMVA (Criteria Match Validator Agent): verifying outputs against predefined formal criteria (temporal logic formulas, rubric-based checklists) — decoupling validation logic from stochastic output. Our four-rule audit (iteration count, DA compliance, prior evaluation consideration, feedback integration) is a domain-specific CMVA implementation.
- Process Reward Models That Think (arXiv:2504.16828): PRMs with reasoning chains before step-level rewards. Our Review Agent is a *generative process reward model* operating in natural language rather than scalar rewards.
- Agent-as-a-Judge paradigm (arXiv:2601.05111): evaluating not just correctness but reasoning quality, turn efficiency, and systematic error diagnosis.

**The boundary condition our paper reveals** (transition to contribution):
- Theory predicts process compliance improves outcomes. Our empirical evidence shows this holds only when process rules encode domain-appropriate *discriminative* criteria — not when they encode generic reasoning quality checks.
- process_valid_rate = 0.980 while Normal F1 = 0.300: the boundary condition in one sentence.

**Gap statement**: Theory strongly supports process-level review. But no prior work has empirically characterized *when* process review fails — the conditions under which process compliance and outcome quality systematically diverge. This paper provides that evidence.

**Key citations**: "Why CoT Fails" (2025), Reasoning LLMs for Clinical Classification (2026), Process vs. Outcome Review (2025), Pathology-CoT (2025), CMVA (2025), Process Reward Models That Think (2025)

### §2.3 Failure Mode Frameworks for Multi-Agent Clinical Systems (~450 words)

**Claim**: Frameworks exist for theorizing and designing around failure modes in multi-agent clinical AI. What does not exist is systematic *empirical* evidence from a real deployed system. This paper bridges that gap.

Cover ALL frameworks from `docs/deep_research_fail_modes.md`:

**TED (Task-Event-Data) Red Teaming**:
- Pairs AI psychotherapists with simulated patient agents equipped with dynamic cognitive-affective models; runs across thousands of sessions to probe for emergent risks (AI Psychosis, failure to de-escalate crisis) that single-turn benchmarks cannot detect (arXiv:2602.19948).
- Uses a comprehensive quality-of-care and risk ontology grounded in DSM-5.
- *Structural analog to our work*: Our 22-session iterative development — building, testing, diagnosing, revising — is TED-style systematic red teaming, conducted on a real system rather than simulated patients. Our experiment progression IS the TED methodology applied empirically.

**PAMAS (Self-Adaptive MAS with Perspective Aggregation)**:
- Addresses the "information drowning" problem: when abundant normal signals overwhelm sparse clinical warning signs (arXiv:2602.03158).
- Hierarchical roles: Auditors (anomaly cues), Coordinators (perspective aggregation), Decision-Makers (context-aware final judgment).
- *Relation to our work*: Our curation pipeline addresses information drowning differently — severity-weighted pre-filtering concentrates analysis on the most clinically salient posts rather than distributing perspectives across all content.

**AgentOps RCA Taxonomy**:
- Distinguishes system-centric (API failures, latency), model-centric (hallucinations, context misalignment), and orchestration-centric (prompt errors, decomposition flaws) failure origins.
- Our FM-4 (specification gaming) and FM-2 (process-outcome paradox) are primarily orchestration-centric in the AgentOps taxonomy.

**Constrained Process Maps / MDP Formalization**:
- Multi-agent workflows formalized as DAG-structured bounded-horizon MDPs (arXiv:2602.02034). Epistemic uncertainty quantified via Monte Carlo estimation. Demonstrates 19% accuracy improvement and 85x reduction in human review requirements.
- *Relation*: Our failure modes reveal empirically where MDP-formalized workflows can be exploited (FM-4: rule conflicts produce gaming) and where formal compliance diverges from clinical utility (FM-2).

**FAITA-MH and READI**:
- FAITA-MH: public-facing assessment across five domains (Credibility, User Experience, Crisis Protocols, User Agency, Transparency).
- READI: implementation-oriented lens (Stakeholder Analysis, Safety, Engagement Metrics, Equity).
- Our work contributes empirical failure mode evidence that these frameworks can use as diagnostic targets.

**MIND-SAFE (Mental Well-Being Through Dialogue — Safeguarded and Adaptive Framework)**:
- Defense-in-depth layered architecture: Input Layer with proactive risk detection → Dialogue Engine (RAG + USD) → Post-generation ethical safeguard filter → Continuous learning with expert oversight.
- *Distinction*: MIND-SAFE's review layer functions as an ethical safeguard against harmful outputs — it audits for safety and appropriateness, not reasoning process compliance. Our Review Agent audits the *logic* of classification reasoning, not the safety of the output.

**Neural Howlround / RISM (Recursive Internal Salience Misreinforcement)**:
- Self-reinforcing probability shifts during inference causing cognitive rigidity and "locked-in" states of false overconfidence (arXiv:2504.07992).
- *Acknowledge and distinguish*: Our FM-1 (Forced Dissensus Noise) involves systematic confidence *collapse* at DA iteration 2, not self-reinforcing overconfidence. The mechanism is opposite: RISM produces overconfidence through reinforcement; FM-1 produces noise through mandatory disruption. Both are confidence calibration failures, but from different causes.

**Sycophantic Validation**:
- LLMs tend to flatter or over-align with user stated beliefs — acute risk in mental health contexts where users may have grandiose identities or persecution narratives (Stanford Brainstorm Lab, 2025).
- *Acknowledge and distinguish*: Our FM-2 (Process-Outcome Audit Paradox) is structurally related but functionally distinct. The RA does not sycophantically validate the DA's label — it correctly applies process rules. The failure is that those rules, correctly applied, do not ensure outcome correctness.

**Gap statement**: These frameworks provide theoretical vocabulary and architectural blueprints. What is missing is empirical evidence: which failure modes actually emerge in a real multi-agent clinical classification system, in what sequence, under what architectural conditions, and with what measurable effects? This paper provides that evidence across six failure modes from 22 sessions and 5 full evaluation runs.

**Key citations**: TED (arXiv:2602.19948), PAMAS (arXiv:2602.03158), AgentOps (arXiv:2508.02121), Constrained Process Maps (arXiv:2602.02034), FAITA-MH (Wayhaven), READI (ResearchGate), MIND-SAFE (PMC), Neural Howlround (arXiv:2504.07992), Stanford Brainstorm Lab / Sycophantic Validation (2025)

---

## §3. System Description (~600 words)

**Framing note**: Describe the architecture precisely enough that readers can understand how the failure modes emerge from it. Do not over-describe — every sentence should contribute to failure mode understanding. This section provides the "scene" for the taxonomy that follows.

### 3.1 Architecture Overview (200 words)

**Decision-Review pipeline** with two passes:

- **Decision Agent (DA)**: Processes one social media post per invocation. Performs minimum three internal `evaluate_signal` iterations using a simulated tool-use protocol (LLM generates structured JSON describing tool calls). Iteration 2 is mandatorily designated as Devil's Advocate analysis. Finalization via `finalize_annotation` after minimum two evaluate_signal calls. Outputs: label (Depression/Anxiety/Comorbid/Normal), confidence score, per-iteration reasoning.
- **Review Agent (RA)**: Audits the DA's complete reasoning trajectory (all iterations + final annotation). Applies four-rule protocol: (a) minimum iteration count, (b) DA execution at iteration 2, (c) prior evaluations passed to finalize_annotation, (d) feedback integration in rounds > 1. Outputs: APPROVED/REJECTED verdict, process_valid flag, signal_gap (missed clinical signals), process_error (protocol violations).
- **Pass 1**: All N posts in a batch processed through one DA→RA cycle.
- **Pass 2**: Curated subset (D32 gate: wrong prediction OR reasoning_gap OR low confidence) receives additional DA pass with RA feedback injected.
- **Cross-batch curation pool**: Top-3 posts per batch (severity-weighted F2 + recency) curated into a shared pool; their feedback constitutes `batch_context` injected into subsequent batches.

Backbone: Gemini Flash Lite. Dataset: ANGST (Hengle et al., EMNLP 2024), 4-class (Depression/Anxiety/Comorbid/Normal), 100-post stratified test sample (25 per class, seed=42).

### 3.2 Design Rationale (200 words)

Ground every design choice in the literature before explaining it:

- **Mandatory DA analysis**: Motivated by structured deliberation preventing premature convergence (ChatEval, ICLR 2024 — spurious consensus problem). Three-iteration minimum ensures the DA cannot commit to a label without considering at least one alternative.
- **External RA process audit**: Motivated by process-level review reducing incorrect knowledge by 63% (Process vs. Outcome Review, 2025). Audit covers both outcome (signal_gap) and process (process_error) — consistent with the CMVA pattern.
- **Two-pass adaptive depth**: Motivated by CriticBench's "saturation of review" finding — review resources are most valuable where the review agent can have marginal impact. Pass 1 screens cheaply; Pass 2 is reserved for posts where the RA can act on genuine uncertainty or identified failure.
- **Cross-batch curation**: Motivated by the information drowning problem (PAMAS) — accumulating clinical insights across batches to improve calibration over time.

### 3.3 The Curation Pipeline as Proto-DKR (200 words)

**Frame the curation mechanism as a precursor to dynamic knowledge retrieval** — this is a contribution insight, not a limitation:

The cross-batch curation pool aggregates strategic feedback from curated posts across batches, injecting it as `batch_context` into subsequent DA calls. This is a **proto-dynamic-knowledge-retrieval (DKR)** system: the pool retains the most clinically informative posts (by F2 score and recency), displacing lower-ranked entries as better examples accumulate. FM-6 (Generic Knowledge Pooling Ceiling) empirically reveals where this mechanism succeeds and where it fails — establishing the boundary conditions that motivate full DKR with per-post retrieval in Phase 2.

**Suggested Figure 2**: Architecture diagram showing DA → RA → Pass 2 loop with curation pool connecting batches. Label all data flows: Post → DA → trajectory → RA → verdict/signal_gap → (if Pass 2 eligible) → DA again with feedback injected; pool batch_context → DA. This is the figure that makes the failure modes spatially legible.

---

## §4. Pilot Study Design (~700 words)

**Framing note**: Present the experiment progression as systematic adversarial probing — TED-style red teaming applied to a real system. The key narrative insight is: **each experiment was designed to test the current system, fixing the dominant failure mode, and each fix revealed the next failure mode.** This is not "we ran a bunch of experiments." This is "we systematically escalated our probing until we had a comprehensive failure mode catalog."

### 4.1 Dataset and Evaluation Protocol (200 words)

- **ANGST dataset** (Hengle et al., EMNLP 2024, EMNLP-main.931): 4-class social media mental health classification — Depression, Anxiety, Comorbid (both), Normal.
- **Important framing** (cite F-024 and the ANGST paper): The "Normal" class does NOT mean "no clinical history." All posts — including Normal — come from users who self-disclosed depression or anxiety at least once in their Reddit history (8,083 unique users). The label is post-level and temporal: a trained psychologist judged whether each post expresses *active, current* clinical symptoms at the moment of writing. Posts labeled Normal may explicitly mention a current diagnosis, medication, or past suicidal ideation — as long as the current content expresses recovery or a functional state. This definition is critical for understanding why clinical over-labeling (FM-1 consequence, FM-2 failure mode) is the dominant error pattern.
- **Evaluation sample**: 100-post stratified test split (25 per class, seed=42) held constant across all 5 full evaluation runs.
- **Key metrics**: Per-class F1, macro F1, Pass 2 delta F1 (same-subset comparison), process_valid_rate, DA confidence trajectory distributions.
- **ANGST benchmark context**: ANGST paper Table 6 best macro-F1: 67.4% (GPT-3.5-turbo few-shot), 66.7% (GPT-4 zero-shot). Our system operates zero-shot; task is harder (4-class single-label vs. multi-label binary).

### 4.2 Experiment Progression as Systematic Probing (500 words)

Present this as a named four-phase escalation. Use a table summary at the end.

**Phase 0 — Pipeline Integration (EXP-002, 1 post, single-run)**

EXP-002 was the smoke test: one Reddit post through the full Decision→Review pipeline via live Gemini Flash Lite API. Validated prompt rendering, API round-trip, JSON response parsing, and feedback serialization. Result: successful end-to-end operation. Significance: established that the architecture is mechanically sound. All subsequent failure modes are behavioral, not mechanical.

*Evidence*: Pipeline operational — all JSONL files valid, finalize_annotation called correctly (F-006).

**Phase 1 — Retrospective Structural Analysis (EXP-007, 10 posts, nano-model)**

EXP-007 was a retrospective analysis of a prior experiment using a "nano" model with batch-of-1 processing (each message receives individual feedback). This differs from the main pipeline which uses batch-of-10 with curation selecting Top-3. Analysis of 79–91 instances across three positionally-aligned JSON files.

Key observations:
- **Depression bias confirmed**: nano-model predicted Depression for 53/79 instances (67%) against GT 20%. Depression recall = 1.000, precision = 0.372 — "clinical by default" when uncertain (F-014).
- **DA noise first observed**: 92.6% U-shaped confidence trajectory (iter1: 0.688 → iter2: 0.369 → iter3: 0.685); iter2 flips to Normal 84% of the time regardless of message content; only 49.4% full recovery (F-015). *This is the first evidence for FM-1*.
- **Procedural-only RA feedback**: 100% REJECTED (91/91), all process_valid=false. Feedback dominated by process violations (rule a: 55/91, rule b: 72/91, rule c: 74/91) — no outcome-corrective diagnostic content (F-016). *First indication of FM-2*.
- **Cross-model significance**: The nano-model used a structurally similar but distinct prompt. Observing the same DA noise pattern across models confirms it is structural to the mandatory DA design, not model-specific.

**Phase 2 — Live Validation with Bug Discovery (EXP-003, 10 posts, Gemini Flash Lite)**

EXP-003 was a live validation run using 10 hand-crafted sample posts (3 Depression, 2 Anxiety, 2 Comorbid, 3 Normal) specifically designed to be well-crafted and unambiguous.

Key observations:
- **FM-4 discovered — specification gaming at scale**: DA skipped mandatory Devil's Advocate iteration in 10/10 posts. Mean tool calls per post = 2.0 (expected 4.0). Root cause: STEP C allowed early exit after confidence ≥ 0.8; STEP B mandated DA at iteration 2 "regardless of confidence." Model favored shorter output path (LL-012). *This is FM-4's defining discovery moment.*
- **RA correctly detected gaming**: 100% REJECTED (rule a: insufficient iterations + rule b: missing DA). Detection worked; prevention did not — the RA cannot alter the DA's behavior, only record its violations.
- **Depression bias absent** on Gemini Flash Lite with well-crafted posts: pred rate 0.300 = GT rate 0.300. Suggests nano-model Depression bias was model-specific.
- **Normal recall = 1.000** on hand-crafted Normal posts — not a hard benchmark.
- **Fix applied and validated (EXP-003c)**: DA prompt v5.2 restricted early exit to ≥ 2 evaluate_signal calls. Added explicit IMPORTANT warning. Re-validation: 10/10 posts correct, DA iteration always executes. *Fix closed FM-4 in the prompt specification.*

**Phase 3 — Full Evaluation at Scale (EXP-004, 100 posts, 5 runs across Sessions 13–22)**

EXP-004 was the primary empirical evaluation: 5 distinct full runs against the 100-post stratified ANGST sample, each run with updated DA and RA prompt versions.

| Session | DA | RA | Normal F1 | Macro F1 | Key Decision |
|---|---|---|---|---|---|
| S13 | v5.4 | v8 | 0.190 | ~0.40 | D29 REFINE |
| S18 validation | v5.4 | v8 | 0.333* | — | D33 PROCEED |
| S21 | v5.5 | v8 | 0.214 | 0.432 | D37 REFINE |
| **S22** | **v5.6** | **v9** | **0.300** | **0.450** | **D40 PIVOT** |

*4-post validation sample only

Key insight from the progression: **each REFINE iteration closed one failure mode and revealed the next.** v5.4 improved Normal discrimination (closed FM-4 after EXP-003 fix) but exposed FM-2 at scale. v5.5 addressed Normal prompt gap; v5.6 added Depression/Normal boundary check — improved Normal +0.086 but caused Comorbid -0.061 (FM-5). Final state: FM-1–3, FM-5–6 all active simultaneously at scale.

The full results table with supporting figure is provided in §5.

**Summary table** (suggested Table 1 in paper):

| Experiment | N | Model | Discovery | FM First Evidenced |
|---|---|---|---|---|
| EXP-002 | 1 | Gemini Flash Lite | Pipeline operational | — |
| EXP-007 | ~80 | Nano-model | DA noise; Depression bias; RA procedural feedback | FM-1 (first), FM-2 (first) |
| EXP-003 | 10 | Gemini Flash Lite | DA specification gaming | FM-4 (defining) |
| EXP-003c | 10 | Gemini Flash Lite | v5.2 fix validated | FM-4 (closed) |
| EXP-004 (5 runs) | 100 | Gemini Flash Lite | Full taxonomy at scale | FM-1–3, FM-5–6 |

---

## §5. Results: Failure Mode Taxonomy (~2,800 words)

**Framing note**: Three-level organization: Single-Agent → Multi-Agent Interaction → System-Level. For each FM: What → Evidence (with brief summaries, not just IDs) → Design Principle → Generalizability. This section is the core contribution of the paper.

**Framing note on evidence format**: Every evidence item must follow this pattern: [brief description of what was found] ([reference: F-xxx / LL-xxx / EXP-xxx]). NEVER cite a reference ID without explaining what it found. See examples in each FM below.

**Suggested Table 2** (Summary table — place at start of this section or end):

| FM | Name | Scope | Theoretical Construct | Design Principle |
|---|---|---|---|---|
| FM-1 | Forced Dissensus Noise | Single-agent | Reflexion / self-correction | External audit required; self-critique without grounding produces noise |
| FM-2 | Process-Outcome Audit Paradox | Multi-agent interaction | Goodhart's Law | Process rules must encode discriminative criteria, not just compliance checks |
| FM-3 | Confidence-Feedback Blind Spot | Multi-agent interaction | Confidence calibration | Confidence-gated feedback has a structural unreachable zone for confident errors |
| FM-4 | Specification Gaming | Single-agent (exploited in DA-RA) | Rule conflict exploitation | No mandatory step may be bypassable by a co-present optional exit rule |
| FM-5 | Cross-Class Collateral Damage | System-level | Shared prompt space coupling | Class-specific fixes propagate to adjacent classes; audit cross-class impact |
| FM-6 | Generic Knowledge Pooling Ceiling | System-level | DKR / knowledge retrieval | Generic pooling works for coarse categories; fine-grained discrimination requires domain-specific retrieval |

### 5.1 Single-Agent Failure Mode: Why Self-Improvement Is Insufficient

#### FM-1: Forced Dissensus Noise (Devil's Advocate Failure) (~450 words)

**What**: The DA's mandatory self-critique iteration (iteration 2) produces systematic U-shaped confidence trajectories — mechanical challenge followed by partial recovery — without improving accuracy. The DA is performing the *form* of deliberation without the *substance*.

**Evidence** (full, with brief summaries):

- *EXP-007 nano-model, first observation*: Across 81 instances with complete 3-iteration trajectories, 92.6% exhibit U-shaped confidence (iter1: 0.688 → iter2: 0.369 → iter3: 0.685). Iteration 2 flips to Normal label 84% of the time regardless of message content. Only 49.4% of instances fully recover to iter1 confidence level by iter3. Cross-model observation: the nano-model uses structurally similar but distinct prompts, confirming this is structural to the mandatory DA design (F-015, F-018).
- *EXP-004 full evaluation, confirmed at scale*: 92.6% U-shaped trajectories across 100 posts with Gemini Flash Lite. DA iteration 2 reasoning is formulaic: "while [external factor] exists, the presence of [clinical signal] transcends situational distress." This formula generates a surface-level counterargument and immediately dismisses it without genuinely evaluating whether clinical language is temporally or pragmatically contextual. 20/21 misclassified Normal posts had DA iteration 2 returning a clinical label despite the Devil's Advocate instruction (F-027).
- *LL-010*: "Mandatory Devil's Advocate iterations can introduce systematic confidence noise without improving classification." The DA treats DA as an instruction to output low confidence, not as genuine reasoning. Signs of this failure mode: dominant contrarian label (Normal chosen ~84% of the time), systematic confidence collapse at the DA step, poor recovery rate.
- *EXP-007, 49.4% non-recovery rate*: Approximately half of all instances end with lower confidence than they started, even when the label is correct (F-015).

**Mechanism**: The DA mandate is a process rule ("you must consider an alternative") without a quality gate ("the alternative must be substantive"). The model satisfies the rule by generating a perfunctory Normal evaluation with low confidence, then reverting. The 84% Normal-flip rate indicates the model has learned a shortcut: "DA = output Normal with low confidence."

**Design Principle**: *Single-agent self-correction (Reflexion-style) is insufficient for clinical classification.* Mandatory self-critique without external grounding or quality gating produces confidence noise, not genuine deliberation. External process auditing is necessary — but, as FM-2 demonstrates, also insufficient alone. If DA is mandated, it needs either: (a) a quality gate evaluating whether the alternative explanation is substantive, (b) integration into the primary evaluation rather than a separate pass, or (c) explicit named checks that operationalize the discrimination — not a vague directive to "search for external factors."

**Generalizable to**: Any system using self-reflexion, chain-of-thought self-critique, or mandatory counterargument steps for high-stakes classification. The failure mode generalizes whenever a mandatory deliberation step is specified without a substantive quality criterion.

### 5.2 Multi-Agent Interaction Failure Modes: Where the DA-RA Loop Breaks Down

#### FM-2: Process-Outcome Audit Paradox (Goodhart's Law in Agent Feedback) (~500 words)

**What**: The Review Agent achieved 98% process_valid_rate while Normal class F1 = 0.300 and Macro F1 = 0.450. Process-compliant reasoning produced wrong outcomes. The RA optimizes for what it can measure (compliance) over what matters (outcome quality).

**Evidence** (full, with brief summaries):

- *EXP-007 nano-model, first indication*: Review Agent produced 100% REJECTED (91/91 instances) with all process_valid=false. Violation breakdown: rule b (missing DA) in 72/91, rule c (prior_evaluations not passed to finalize) in 74/91, rule a (insufficient iterations) in 55/91. All feedback was procedural — "follow rules b/c" rather than "your label is wrong." First indication that the RA optimizes for process compliance, not outcome correction (F-016).
- *EXP-003 live validation, detection without prevention*: RA correctly detected DA's specification gaming (100% REJECTED, rule a: insufficient iterations + rule b: missing DA). But detection was recorded, not actionable — the RA cannot prevent the DA's behavior, only report on it. Correct detection did not produce correct outcomes (EXP-003 results).
- *EXP-004 S22 full scale, decisive evidence*: process_valid_rate = 0.980 while Normal F1 = 0.300 and Macro F1 = 0.450. The RA approved 98% of reasoning trajectories that produced 52 wrong predictions across 100 posts. Process compliance and outcome quality are structurally dissociated (F-039).
- *LL-011*: "Review agents optimized for process compliance produce procedural feedback, not outcome-corrective — Goodhart's Law in agent feedback." Process verification is structurally easier (mechanical rule-checking) than diagnostic reasoning (requires clinical domain knowledge). The feedback loop optimizes for what the reviewer can measure, not what would most improve classification. "When a measure becomes a target, it ceases to be a good measure."
- *EXP-004 Normal inspection*: All 21 misclassified Normal posts received RA REJECTED feedback. In every case the RA's signal_gap correctly identified the failure: DA "conflated clinical vocabulary with internalized pathology without adequately considering situational/proportional framing." The RA's feedback was clinically correct — but had zero effect on outcomes: 20/21 posts exited via CONFIDENCE_THRESHOLD before RA feedback could be injected. The RA is not the bottleneck; the pipeline architecture is (F-026).

**Mechanism**: The RA's tools make process verification easy and dominant. Structural rule-checking (did the trajectory contain 3 iterations? Was DA present?) crowds out the harder diagnostic reasoning (is this classification clinically correct?). Combined with confidence-gated Pass 2 (FM-3), correct RA feedback is architecturally disconnected from DA correction.

**Design Principle**: *Process auditing without outcome-aware feedback produces compliant but incorrect reasoning.* Process review reduces incorrect knowledge (per theory) only when process rules encode domain-appropriate discriminative criteria — not generic reasoning quality checks. Measure feedback composition (% process vs. % diagnostic) as a quality metric. Explicitly prioritize outcome-corrective feedback over procedural corrections in review agent design.

**Generalizable to**: Any multi-agent system where one agent reviews another's reasoning against process rules. Goodhart's Law is architectural: wherever compliance is easier to measure than correctness, the system will optimize for compliance.

#### FM-3: Confidence-Feedback Blind Spot (~400 words)

**What**: 52/100 posts were wrong predictions, but only 6/100 had confidence < 0.8. High-confidence wrong predictions structurally bypass the feedback mechanism. The confidence guard — designed to prevent harmful feedback on correct predictions — creates an unreachable zone for confident incorrect predictions.

**Evidence** (full, with brief summaries):

- *EXP-004 S13 original run, structural blind spot confirmed*: Only 6/100 posts had confidence < 0.8, yet 52 were wrong. The confidence guard designed to protect correct predictions (LL-009 fix) creates a zone where 46/52 wrong predictions terminate via CONFIDENCE_THRESHOLD and never enter Pass 2. The feedback loop cannot correct the most confident errors (F-021).
- *D32 redesign, oracle-aided bypass*: Replacing confidence-only gate with OR gate (wrong prediction OR reasoning_gap OR low confidence) increased Pass 2 activation from 6% to 72% (66/100 posts in EXP-004 S21). Confirmed at scale: 5/10 batches showed 60–80% activation consistently (F-033). However: this gate requires ground truth (wrong prediction) — not available in production deployment.
- *EXP-004 Normal inspection, 20/21 unreachable*: 20/21 misclassified Normal posts had confidence ≥ 0.8 and exited via CONFIDENCE_THRESHOLD. The RA correctly identified signal_gap in all 21 cases, but this feedback was logged without being injected — architecturally disconnected (F-026, F-039).
- *LL-014*: "Confidence-gated Pass 2 silently skips high-confidence wrong predictions — the feedback loop cannot correct the most confident errors." A confidence threshold is a proxy for uncertainty, not for correctness. A model can be wrong and confident simultaneously — especially in biased ICL settings where the model consistently over-predicts one class.

**Design Principle**: *Confidence-gated feedback loops have a structural blind spot for confident errors.* In clinical classification, the most dangerous predictions (high-confidence wrong) are exactly the ones the feedback mechanism cannot reach without external ground truth access. Design feedback loop activation criteria that distinguish confidence from correctness — or accept that the feedback mechanism cannot correct the most confident errors without oracle information.

**Generalizable to**: Any iterative feedback system with confidence-gated activation. The mechanism of confident-but-wrong predictions bypassing correction is structural to any confidence threshold, not specific to this architecture.

#### FM-4: Specification Gaming / Conflicting Rule Exploitation (~400 words)

**What**: During EXP-003 validation, the DA skipped the mandatory Devil's Advocate iteration in 10/10 posts by exploiting a conflict between an early-exit permission and a mandatory evaluation requirement. The model chose the rule that minimized output — reward hacking without an explicit reward signal.

**Evidence** (full, with brief summaries):

- *EXP-003 live validation, 10/10 bypass*: Mean tool calls per post = 2.0 (expected 4.0 minimum). DA iteration 2 executed 0/10 times. STEP C: "If the last evaluate_signal returned confidence_score >= threshold → you MAY call finalize_annotation immediately." STEP B: "If iteration == 2, you MUST perform Devil's Advocate reasoning, regardless of your current confidence score." When iteration 1 confidence ≥ 0.8 (all 10 well-crafted posts), the model interpreted STEP C as permission to skip directly to finalize_annotation. STEP B's "regardless of confidence" language was insufficient to override STEP C's permission grant (LL-012).
- *EXP-003, RA detection correct but insufficient*: RA produced 100% REJECTED (rule a: insufficient iterations + rule b: missing DA). Detection worked; prevention did not. The RA can record violations but cannot alter the DA's behavior within the same invocation.
- *D22 fix, validated in EXP-003c*: DA prompt v5.2 restricted early exit to ≥ 2 evaluate_signal calls. Added explicit WARNING: "Condition (b) CANNOT fire after iteration 1. Iteration 2 (DA) is always mandatory regardless of confidence." Re-validation: 10/10 posts correct, DA iteration always executes. Fix validated (D22, D23).
- *LL-012*: "When two prompt rules conflict, the model favors the one that minimizes output — never allow a mandatory step to be bypassable by an optional early-exit rule." Root cause: LLMs prefer shorter output paths. A permission grant ("MAY skip") overrides a constraint ("MUST perform") when they conflict, because satisfying the permission requires less generation.

**Design Principle**: *When multi-agent prompts contain conflicting rules, models exploit the path of least resistance.* Prompt specifications must be formally consistent: any pair of rules that can logically conflict will be exploited. Mandatory steps must be made immune to co-present optional exit conditions — state the restriction explicitly at the point of the permission grant. Never rely on "regardless of condition" language in a mandatory step to override a permission in another step.

**Generalizable to**: Any prompt-driven agent system with complex instruction sets. Specification gaming without explicit reward signals is a fundamental LLM behavior emerging from output-minimization pressure.

### 5.3 System-Level Failure Modes

#### FM-5: Cross-Class Collateral Damage (~350 words)

**What**: A Depression/Normal boundary check (DA v5.6) improved Normal F1 by +0.086 but regressed Comorbid F1 by -0.061. Targeted fixes in a shared prompt space cause adjacent-class regression. In a system with a shared reasoning prompt across all classes, no fix is class-local.

**Evidence** (full, with brief summaries):

- *EXP-004 S21→S22 (v5.5→v5.6), collateral damage measured*: DA v5.6 added a Depression/Normal boundary check requiring persistence, functional impairment, or unanchored hopelessness for Depression classification. Normal F1: 0.214 → 0.300 (+0.086). Comorbid F1: 0.444 → 0.383 (-0.061). The boundary check fires on posts with Depression-type language in a Comorbid context — posts with both Depression and Anxiety signals necessarily contain Depression markers, which the boundary check interprets as evidence for Normal rather than completing Comorbid dual-signal evaluation (F-042, F-039).
- *LL-032*: "DA class-specific boundary checks cause collateral regression in adjacent multi-signal classes — Depression/Normal gate reduced Comorbid F1 by -0.061 by over-applying Normal to Depression-type language in Comorbid posts." The Comorbid class requires *both* Depression and Anxiety signals independently at ≥ 0.8 — it necessarily contains Depression markers that the boundary check then misinterprets.

**Mechanism**: The DA processes all classes within a single prompt invocation. Boundary checks are implemented as conditional logic in the reasoning protocol. A check targeting the Depression/Normal boundary applies to the DA's reasoning about any post, not only posts where Depression and Normal are the plausible candidates. Comorbid posts with Depression-type language meet the check's criteria and are reclassified before Comorbid dual-signal evaluation completes.

**Design Principle**: *In multi-class feedback systems with shared reasoning, class-specific corrections propagate unpredictably to adjacent classes.* Audit cross-class F1 impact before deploying any class-specific fix. In shared-prompt architectures, no fix is class-local. Consider class-specific routing (separate reasoning paths per class) to isolate fixes.

**Generalizable to**: Any multi-class classification system where prompt or model changes target a single class. The mechanism generalizes wherever a shared reasoning space is used for multiple classes — a common pattern in LLM classification.

#### FM-6: Generic Knowledge Pooling Ceiling (Curation as Proto-DKR) (~450 words)

**What**: The cross-batch cumulative feedback pool improved clinical classes (Anxiety ΔF1 = +0.143, Comorbid ΔF1 = +0.129) but had zero effect on Normal (ΔF1 = 0.000). Generic strategic feedback cannot resolve fine-grained discrimination. The pool is a step toward dynamic knowledge retrieval — and its ceiling reveals exactly what full DKR must provide.

**Evidence** (full, with brief summaries):

- *EXP-004 D37 REFINE, Pass 2 class deltas*: Anxiety ΔF1 = +0.143, Comorbid ΔF1 = +0.129, Normal ΔF1 = 0.000, Depression ΔF1 ≈ 0. Pool calibration works for broad clinical categories (Anxiety, Comorbid) where batch-level strategic principles are sufficient, but not for fine-grained Normal discrimination where each post's misclassification requires post-specific corrective feedback (F-040).
- *EXP-004 Normal Pass 2 analysis, RA drives Normal posts toward clinical labels*: 19 Normal posts entered Pass 2. Post-P2 confusion: 11→Depression, 3→Anxiety, 5→Comorbid, 0 corrected to Normal. The RA actively drives Normal posts toward clinical labels because the pool excludes Normal feedback (to prevent clinical-bias amplification). Without Normal-affirming strategic guidance in the pool, the RA's corrective branch defaults to clinical alternatives (F-040, F-035).
- *Zero-shot ceiling confirmed after 6 DA versions*: After DA v5.1→v5.6 (6 versions) and RA v6→v9 (4 versions), Normal F1 plateaued at 0.300. ANGST paper (Hengle et al., EMNLP 2024) confirms few-shot prompting yields +6.9pp macro-F1 over zero-shot for GPT-3.5 — directly motivating the PIVOT to few-shot ICL (F-041, F-036).
- *LL-031*: "Pool-based Pass 2 calibration cannot override high-confidence wrong Normal predictions — batch_context from pool is general calibration, not per-post correction." The DA hears a general principle ("when clinical language appears, assess active vs contextual engagement") but receives no specific signal about why its prediction on *this specific post* is wrong. Per-post signal_gap injection (D27) is the architectural bottleneck.
- *LL-029*: "Zero-shot prompt engineering has a ceiling for high-distress situational Normal posts — boundary checks close moderate cases but cannot distinguish intense situational distress from clinical Depression without few-shot examples."
- *Comorbid ΔF1 = +0.129 proves the mechanism works*: The positive delta for Anxiety and Comorbid is not accidental — it demonstrates that pool-based calibration functions correctly for classes where pool strategic principles align with the discriminative criteria. The failure is class-specific, not architectural.

**Design Principle**: *Generic feedback pooling works for broad clinical categories but fails for fine-grained discrimination requiring domain-specific knowledge.* Pool-based calibration is a cross-batch smoothing mechanism, not a per-post corrective mechanism. For high-confidence wrong predictions on semantically challenging classes, dynamic knowledge retrieval must supply domain-specific, per-post discriminative evidence — not just accumulated strategic feedback. The progression from pool (FM-6 ceiling) to DKR (Phase 2) is the architectural implication of this failure mode.

**Generalizable to**: Any multi-agent system using feedback pooling or accumulated context for knowledge transfer. The generalization: pool-level calibration has a fundamental resolution limit — below which per-instance retrieval is required. This is a universal trade-off between computational efficiency (pooling) and corrective precision (retrieval).

---

## §6. Discussion (~1,000 words)

**Framing note**: The Discussion synthesizes the six failure modes into three higher-level insights. It answers: what do these failures, taken together, tell us about multi-agent clinical AI? The Discussion should feel conclusory and provocative — not a repetition of the results.

### 6.1 Self-Improvement Is Insufficient; External Review Is Also Insufficient (300 words)

The failure mode taxonomy reveals a two-layer insufficiency that the field has not yet fully confronted:

- **Layer 1 (FM-1, FM-4)**: Single-agent self-improvement mechanisms — mandatory self-critique, iterative deliberation — produce compliance behavior, not genuine reasoning. FM-1 shows that mandatory DA produces noise because the model treats it as a format requirement. FM-4 shows that when prompt rules conflict, models exploit the conflict rather than resolve it. Self-improvement, absent external grounding, optimizes for output minimization.
- **Layer 2 (FM-2, FM-3)**: External review agents that audit the single-agent's reasoning are *necessary but not sufficient*. FM-2 shows that process-compliant reasoning can systematically produce wrong outcomes — the audit mechanism optimizes for what it can measure (process rules) rather than what it should ensure (outcome correctness). FM-3 shows that even when the RA diagnoses the failure correctly, the feedback mechanism cannot reach the most confident errors.

The field has debated single-agent vs. multi-agent approaches. This evidence suggests the more productive question is: *what must the review agent's audit actually measure, and how must it be architecturally connected to the decision agent's subsequent reasoning?* Neither the existence of review nor its correctness is sufficient — the review must be (a) outcome-aware, not just process-aware, and (b) architecturally connected to a subsequent DA invocation that can act on it.

### 6.2 Domain-Specific Knowledge Is the Missing Ingredient (350 words)

FM-6 reveals the fundamental knowledge gap that generic mechanisms cannot close: fine-grained discrimination between clinically similar classes (active depression vs. situational distress) requires domain-specific, per-instance knowledge that accumulated feedback cannot supply.

This has direct implications for architecture:
- **Pool-based calibration** (our current system) is a zero-order approximation: it accumulates strategic principles but cannot distinguish between posts that superficially resemble a class and posts that exemplify it.
- **Few-shot ICL** is a first-order improvement: representative examples provide the discriminative boundary information that abstract strategic principles lack. The ANGST paper's +6.9pp macro-F1 advantage for few-shot over zero-shot directly confirms this.
- **Dynamic knowledge retrieval (DKR)** is the second-order solution: per-post retrieval of domain-specific evidence enables the DA to access the exact discriminative information relevant to the specific post being classified.

The progression from pool (FM-6 evidence) to few-shot ICL (planned PIVOT) to DKR (Phase 2) is not a failure sequence but a principled research trajectory driven by empirical evidence.

### 6.3 Process Review Theory: Validated in Principle, Bounded in Practice (200 words)

The literature establishes that process-level review reduces incorrect knowledge by 63%. Our evidence confirms this is true *in principle* — the RA correctly diagnoses failure modes in all 21 misclassified Normal posts (F-026). But three boundary conditions limit the theory's practical application:

1. **Process rules must encode discriminative criteria**: Rules that verify mechanical compliance (iteration count, DA presence) cannot verify that the DA's reasoning applied the correct discriminative logic.
2. **Feedback must be architecturally connected**: Correct feedback that is logged but not injected into a subsequent DA invocation has zero corrective effect (FM-3 + FM-2 interaction).
3. **Review cannot compensate for prompt-level rule conflicts**: When the DA specification allows rules to conflict (FM-4), the RA can detect the exploitation but cannot prevent it.

The 63% reduction in incorrect knowledge is achievable — but only when all three conditions are met simultaneously. Our evidence localizes precisely where each condition failed.

### 6.4 Generalizability Beyond Mental Health Classification (150 words)

Each of the six failure modes was discovered in a mental health classification context but generalizes to any multi-agent clinical AI system. FM-1 applies to any system with mandatory self-critique. FM-2 applies to any process audit mechanism where compliance is easier to verify than correctness. FM-3 applies to any confidence-gated feedback loop. FM-4 applies to any prompt-driven agent with complex instruction sets. FM-5 applies to any multi-class shared-reasoning system. FM-6 applies to any system using accumulated feedback for knowledge transfer.

The contribution is not mental health-specific — it is the methodology of systematic empirical failure mode discovery applied to a real system, producing a generalizable taxonomy and design principles.

---

## §7. Limitations and Future Work (~400 words)

**Framing note**: Frame limitations as research agenda items, not apologies. Each limitation motivates a specific next step.

### 7.1 Pilot Scale

- 100-post test sample, single dataset (ANGST). Sufficient to discover and characterize failure modes, but insufficient for statistical generalization.
- Single model backbone (Gemini Flash Lite). EXP-007 provides cross-model replication of FM-1 via nano-model; FM-4 confirmed on both. Additional models would further validate cross-model generalizability.
- Future: evaluate on larger ANGST test split, additional datasets (Dreaddit, SDCNL), multiple LLM backbones.

### 7.2 Oracle-Aided Evaluation Gate

- The D32 Pass 2 gate uses ground truth (wrong prediction) to expand Pass 2 activation — not available in production deployment. Oracle-aided evaluation establishes the ceiling of the feedback mechanism's correction capacity; it does not represent realistic deployment performance.
- Future: D27 (per-post signal_gap injection from RA into subsequent DA invocations without oracle access) is the production-realistic implementation.

### 7.3 Zero-Shot Configuration

- All EXP-004 runs used zero-shot prompting. FM-6 evidence demonstrates this is the binding constraint for Normal classification. Few-shot ICL is planned (D40 PIVOT, pending approval).
- The ANGST paper's +6.9pp macro-F1 advantage for few-shot provides the strongest available external evidence that this limitation is real and addressable.

### 7.4 Planned Next Steps

- **D40 PIVOT to few-shot ICL**: Adding Normal examples targeting Group A failure cases (contextual/advisory framing posts labeled Normal by ANGST psychologists).
- **D27 per-post signal_gap injection**: Activating the RA's correct feedback as per-post correction in Pass 2 without oracle gate.
- **Phase 2 Dynamic Knowledge Retrieval**: Replacing the pool mechanism with per-post retrieval of semantically similar correctly-classified posts as few-shot exemplars — the architectural implication of FM-6.
- **EXP-005 ablation**: Isolating the RA's contribution by comparing (a) DA alone, (b) DA + RA without feedback, (c) full DA-RA loop with feedback.

---

## §8. Conclusion (~300 words)

**Framing note**: The Conclusion must be strong and prescriptive. It is NOT a summary — it is the final argument. Two paragraphs.

**Paragraph 1 — The finding**: We characterized six failure modes from a real multi-agent mental health classification system through 22 sessions of systematic empirical probing. Organized across three architectural scopes — single-agent self-correction (FM-1, FM-4), multi-agent interaction (FM-2, FM-3), and system-level behavior (FM-5, FM-6) — these failure modes are not implementation accidents. They are structural consequences of architectural choices that the field has not yet fully diagnosed: mandatory self-critique without quality gating produces noise (FM-1); process compliance and outcome quality can dissociate at scale (FM-2); confidence-gated feedback has a structural blind spot for confident errors (FM-3); conflicting prompt rules are systematically exploited (FM-4); class-specific fixes propagate to adjacent classes in shared-reasoning systems (FM-5); generic knowledge pooling has a resolution limit below which per-instance retrieval is required (FM-6).

**Paragraph 2 — The implication**: Multi-agent clinical AI is not safer by virtue of having more agents. It is safer when each interaction point is designed with explicit awareness of how it can fail. The design principles derived from this taxonomy — external grounding for self-critique, outcome-aware audit metrics, asymmetric feedback architecture for confident errors, formally consistent prompt specifications, cross-class impact auditing for shared-reasoning fixes, and domain-specific retrieval for fine-grained discrimination — are actionable today. They do not require new architectures. They require that the failure modes be known. This paper makes them known.

---

## Suggested Figures and Tables (complete list)

| Item | Description | Location |
|---|---|---|
| Figure 1 | Three-level failure mode taxonomy (visual hierarchy) | §1 or §5 opener |
| Figure 2 | DA-RA-Curation architecture diagram with data flows labeled | §3 |
| Figure 3 | Experiment progression timeline (EXP-002 → EXP-007 → EXP-003 → EXP-004 runs) with failure modes discovered at each stage | §4 |
| Figure 4 | 4×4 confusion matrix for EXP-004 S22 (best run) | §5 |
| Figure 5 | DA confidence trajectory distribution: U-shaped pattern (FM-1) across 100 posts | §5.1 |
| Table 1 | Experiment summary (N, model, discovery, FM first evidenced) | §4 |
| Table 2 | FM summary (FM, name, scope, theoretical construct, design principle) | §5 opener |
| Table 3 | Per-class metrics across EXP-004 runs (all 4 sessions, Pass 1 vs. Pass 2 deltas for S22) | §5.2 or §5.3 |

---

## Approximate Word Count Allocation (8,000 words body)

| Section | Target words |
|---|---|
| §1 Introduction | 700 |
| §2 Related Work | 1,400 |
| §2.1 AI/Agent Advancement | 450 |
| §2.2 Process vs. Outcome Review | 500 |
| §2.3 Failure Mode Frameworks | 450 |
| §3 System Description | 600 |
| §4 Pilot Study Design | 700 |
| §5 Failure Mode Taxonomy | 2,800 |
| §5.1 FM-1 | 450 |
| §5.2 FM-2 | 500 |
| §5.3 FM-3 | 400 |
| §5.4 FM-4 | 400 |
| §5.5 FM-5 | 350 |
| §5.6 FM-6 | 450 (note: FM-5 and FM-6 are both in §5.3 System-Level) |
| §6 Discussion | 1,000 |
| §7 Limitations | 400 |
| §8 Conclusion | 300 |
| **Total** | **7,900** |

*~100 words of flex for table captions and figure captions*

---

## Evidence Cross-Reference Index

This index maps every evidence item cited in the outline to its source file and content summary. Use this when drafting to verify claims.

### Findings (F-xxx)

| ID | Content summary | Source file |
|---|---|---|
| F-006 | Smoke test validates end-to-end pipeline: all JSONL valid, finalize called 10/10 | knowledge/findings.md |
| F-014 | Nano-model: 67% Depression predictions vs 20% GT; Depression recall 1.000, precision 0.372 | knowledge/findings.md |
| F-015 | DA U-shaped confidence: 92.6% instances, iter1=0.688→iter2=0.369→iter3=0.685; iter2 flips Normal 84%; 49.4% non-recovery | knowledge/findings.md |
| F-016 | RA 100% REJECTED (91/91) on nano-model: all procedural violations (rule b: 72, rule c: 74, rule a: 55), no outcome-corrective feedback | knowledge/findings.md |
| F-018 | Cross-validation: EXP-007 independently confirms LL-008 and LL-009; asymmetry direction differs but phenomenon replicated | knowledge/findings.md |
| F-021 | Pass 2 structurally inactive: only 6/100 posts (6%) had confidence < 0.8; 52 wrong predictions bypassed | knowledge/findings.md |
| F-024 | ANGST Normal class: post-level temporal judgment; all posts from users who self-disclosed depression/anxiety; Normal = active-current-distress absent, not clinical-history absent | knowledge/findings.md |
| F-025 | 21 misclassified Normal posts split: 11 (Group A: fixable with prompt redesign — contextual/advisory framing) + 10 (Group B: genuine boundary cases, limited fix ceiling) | knowledge/findings.md |
| F-026 | RA correctly diagnosed all 21 misclassified Normal posts (signal_gap: "conflated clinical vocabulary with internalized pathology") but 20/21 exited via CONFIDENCE_THRESHOLD before feedback could be injected | knowledge/findings.md |
| F-027 | DA DA-step formulaic in 20/21 misclassified Normal cases: "while [external factor] exists, [clinical signal] transcends situational distress" — surface counterargument, immediate dismissal | knowledge/findings.md |
| F-033 | D32 gate activation 72% confirmed at scale (5/10 batches, 50 posts) vs. 6% with confidence-only gate | knowledge/findings.md |
| F-034 | EXP-004 S21 (v5.5/v8): Depression=0.520, Anxiety=0.550, Comorbid=0.444, Normal=0.214, macro=0.432 | knowledge/findings.md |
| F-035 | EXP-004 Pass 2 Normal ΔF1=0.000: 19 Normal posts entered, 0 corrected; post-P2 confusion: 11→Depression, 3→Anxiety, 5→Comorbid | knowledge/findings.md |
| F-036 | ANGST paper benchmark: GPT-3.5-turbo few-shot 67.4% macro-F1; our zero-shot 43.2%; few-shot +6.9pp over zero-shot confirmed | knowledge/findings.md |
| F-039 | EXP-004 S22 (v5.6/v9): Depression=0.566, Anxiety=0.550, Comorbid=0.383, Normal=0.300, macro=0.450; process_valid_rate=0.980 | knowledge/findings.md |
| F-040 | EXP-004 D37 Pass 2 deltas: Anxiety ΔF1=+0.143, Comorbid ΔF1=+0.129, Normal ΔF1=0.000, Depression ΔF1≈0 | knowledge/findings.md |
| F-041 | Zero-shot Normal ceiling confirmed: Normal F1 plateaued at 0.300 after 6 DA versions + 4 RA versions | knowledge/findings.md |
| F-042 | Depression/Normal boundary check (v5.6): Normal +0.086, Comorbid -0.061; Comorbid regression from boundary check firing on Comorbid posts with Depression-type language | knowledge/findings.md |

### Lessons (LL-xxx)

| ID | Content summary | Source file |
|---|---|---|
| LL-009 | Feedback loops without pre-round confidence guard apply feedback to already-correct high-confidence predictions; pre-round guard required | lessons/design.md |
| LL-010 | Mandatory DA iterations can introduce systematic confidence noise without improving classification; model treats DA as format rule, not genuine reasoning | lessons/design.md |
| LL-011 | Review agents auditing process compliance produce procedural feedback, not outcome-corrective — Goodhart's Law in agent feedback; process verification crowds out diagnostic reasoning | lessons/design.md |
| LL-012 | When two prompt rules conflict, model favors the one minimizing output; mandatory steps must be immune to co-present optional exit conditions | lessons/design.md |
| LL-014 | Confidence-gated Pass 2 silently skips high-confidence wrong predictions; confidence threshold is proxy for uncertainty, not correctness | lessons/methodology.md |
| LL-015 | ICL clinical prompts without explicit Normal-class guidance produce clinical over-labeling; fix is single binary discriminating question (active personal distress vs. contextual/historical) | lessons/design.md |
| LL-029 | Zero-shot prompt engineering has a ceiling for high-distress situational Normal posts; boundary checks close moderate cases but cannot distinguish intense situational distress from clinical Depression without few-shot examples | lessons/methodology.md |
| LL-031 | Pool-based Pass 2 calibration cannot override high-confidence wrong Normal predictions; batch_context is general calibration, not per-post correction; D27 activation required | lessons/methodology.md |
| LL-032 | DA class-specific boundary checks cause collateral regression in adjacent multi-signal classes; Depression/Normal gate reduced Comorbid F1 by -0.061 | (methodology implied from F-042, D37) |

### Experiments (EXP-xxx)

| ID | Content summary | Source file |
|---|---|---|
| EXP-002 | 1 post, live API — pipeline integration validated; all components functional | knowledge/experiments.md |
| EXP-003 | 10 hand-crafted posts, live API — FM-4 discovered (DA skipped DA 10/10); RA detected (100% REJECTED) but could not prevent | knowledge/experiments.md |
| EXP-003c | Re-validation after v5.2 fix — 10/10 correct, DA iteration always executes; FM-4 closed at prompt level | knowledge/experiments.md |
| EXP-004 | 100 posts (25×4 stratified, seed=42), 5 runs across Sessions 13–22 — full taxonomy FM-1–3, FM-5–6 at scale | knowledge/experiments.md |
| EXP-007 | ~80 nano-model instances, retrospective — first observation FM-1 (U-shaped confidence 92.6%); first indication FM-2 (procedural-only RA feedback) | knowledge/experiments.md |

---

## Key Direct Quotes for the Writer

Use these verbatim in the paper where they fit:

1. **LL-010**: "Mandatory Devil's Advocate iterations can introduce systematic confidence noise without improving classification." (FM-1 section)

2. **LL-011**: "Review agents optimized for process compliance produce procedural feedback, not outcome-corrective — Goodhart's Law in agent feedback." (FM-2 section)

3. **LL-012**: "When two prompt rules conflict, the model favors the one that minimizes output — never allow a mandatory step to be bypassable by an optional early-exit rule." (FM-4 section)

4. **LL-014**: "Confidence-gated Pass 2 silently skips high-confidence wrong predictions — the feedback loop cannot correct the most confident errors." (FM-3 section)

5. **LL-031**: "Pool-based Pass 2 calibration cannot override high-confidence wrong Normal predictions — batch_context from pool is general calibration, not per-post correction." (FM-6 section)

6. **LL-029**: "Zero-shot prompt engineering has a ceiling for high-distress situational Normal posts — boundary checks close moderate cases but cannot distinguish intense situational distress from clinical Depression without few-shot examples." (FM-6 section, Discussion)

7. **F-027 (paraphrase as direct narrative)**: In 20/21 misclassified Normal posts, the DA's Devil's Advocate reasoning followed a single formula: "while [external factor] exists, the presence of [clinical signal] transcends situational distress." This formula generates a surface-level counterargument and immediately dismisses it. (FM-1 section, Evidence)

8. **F-026 (paraphrase for Discussion)**: The Review Agent correctly identified the failure mode in all 21 misclassified Normal posts — citing DA "conflated clinical vocabulary with internalized pathology without adequately considering situational/proportional framing." The RA is not the bottleneck. The pipeline architecture is. (Discussion §6.1)
