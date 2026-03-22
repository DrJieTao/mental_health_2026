# Project Status — Multi-Agent Mental Health Detection on Social Media

**Principal Investigator**: Jie Tao, D.Sc., Department of Analytics, Fairfield Dolan School of Business, Fairfield University
**Date**: 2026-03-22
**Phase**: Phase 1 — Decision-Review Agent Loop (pilot study complete, pending design decisions)
**Target Venues**: HICSS, Information Systems Research, Information Systems Frontiers

**Summary**: This study introduces a two-agent LLM architecture for mental health classification on social media. A Decision Agent classifies Reddit posts using structured clinical reasoning with in-context learning, while a Review Agent audits both the classification outcome and the reasoning process, delivering Socratic feedback that drives iterative improvement. A two-pass batch pipeline screens all posts, then curates clinically important cases for deeper multi-round analysis. Evaluated on the ANGST dataset (2,876 gold-labeled Reddit posts, 4-class), a 100-post pilot study has been completed across 22 development sessions, producing both architecture validation (Anxiety ΔF1 = +0.143, Comorbid ΔF1 = +0.129 from feedback) and four empirically documented architectural failure modes. Two open design decisions — task formulation and publication framing — are presented for collaborator discussion.

---

## 1. Research Question & Contributions

### Research Question

Can a multi-agent Decision-Review feedback loop with structured clinical reasoning and reasoning process auditing improve LLM-based mental health classification on social media, compared to single-agent in-context learning?

### Claimed Contributions (verified against Dec 2025 – Mar 2026 literature)

1. **First Decision-Review feedback loop for direct mental health classification.** Prior multi-agent work in mental health (Mao et al., 2026) targets data enrichment, not direct classification with iterative feedback. AgentMental (2025) uses interactive interviews, not post-level classification with process auditing. This work implements a feedback loop where a Review Agent audits both the outcome and reasoning process, and Socratic feedback drives iterative improvement.

2. **Reasoning process auditing (4-rule) during classification.** Existing explainability methods (LIME, attention visualization) are applied post hoc. This work introduces real-time reasoning process auditing — iteration count, Devil's Advocate execution, prior evaluation consideration, and feedback integration — enforced during classification by a separate Review Agent. *Note: this is reasoning process auditing, not clinical protocol compliance — we do not encode DSM-5 or clinical guidelines as auditable rules. We audit reasoning thoroughness and evidence grounding.*

3. **Socratic feedback adapted to clinical signal analysis.** Socratic questioning has been validated for math reasoning (SSR, 2025) and education (Holub et al., 2026) but not clinical text classification. This work adapts Socratic feedback to clinical signal analysis through three channels: `signal_gap` (class-specific diagnostic questions), `reasoning_gap` (ungrounded reasoning detection), and `process_error` (protocol violations).

4. **Adaptive-depth two-pass batch pipeline with cross-batch learning.** All existing mental health classification systems apply uniform computational depth. This work introduces a pipeline that screens all posts cheaply (Pass 1), curates cross-batch feedback (cumulative pool, top-3 per clinical label), then allocates deeper multi-round analysis exclusively to cases identified by an oracle-aided gate (wrong prediction, reasoning gap, or low confidence).

---

## 2. Methodology

### Two-Agent Architecture

The system pairs two LLM agents, both powered by Gemini Flash Lite (`gemini-3.1-flash-lite-preview`) in an in-context learning configuration (no fine-tuning):

- **Decision Agent** (current: prompt v5.6) performs structured clinical reasoning through a mandatory 3-iteration evaluation cycle using two simulated tools (`evaluate_signal()` and `finalize_annotation()`). On each iteration, it analyzes the post for clinical signals (internalization markers: hopelessness, anhedonia, catastrophic thinking, hypervigilance), assigns a label and confidence score, and provides evidence spans. The second iteration requires a mandatory Devil's Advocate analysis with four checks: (1) past-tense/historical? (2) advisory/informational? (3) functional/hopeful stance? (4) episodic/proportional vs. persistent? (Depression-specific). v5.6 adds a Depression/Normal boundary sub-section requiring persistent low mood or functional impairment for Depression assignment.

- **Review Agent** (current: prompt v9) conducts a four-tool audit sequence: (1) `parse_trajectory()` — extract final label, confidence, evaluation history; (2) `audit_outcome()` — compare to ground truth (oracle-aided); (3) `audit_process()` — check 4 rules; (4) `submit_review()` — synthesize Socratic feedback. v9 adds class-specific feedback branches: Normal (active distress vs. contextual engagement), Depression (temporal persistence + functional scope), Comorbid (co-occurrence question), Anxiety (internalization pattern).

### Feedback Protocol

The Review Agent produces three categories of feedback:

- **Signal gap**: Class-specific diagnostic questions targeting overlooked clinical signals. Strategic and transferable across posts (not anchored to specific post phrases). Example for Normal: "Is the clinical language an active expression of distress, or contextual/historical?"
- **Reasoning gap**: Identifies ungrounded reasoning — claims without evidence spans, signal attribution without text support, unexplained confidence jumps. Null if reasoning is substantive.
- **Process error**: One sentence per violated audit rule. Null if all 4 rules are satisfied. Invariant: process_error is null ⟺ process_valid is true.

Feedback is serialized as `[SIGNAL GAP]`, `[REASONING GAP]`, `[PROCESS ERROR]` blocks and injected into the Decision Agent's next iteration.

### Two-Pass Batch Pipeline

1. **Pass 1 (Screening)**: Every post receives one Decision→Review cycle. Results constitute **Baseline 1** (single agent, no iterative feedback).

2. **Curation** (cross-batch cumulative feedback pool): See detailed description below.

3. **Pass 2 (Deep Analysis)**: Posts meeting any of three gate conditions enter up to 3 feedback rounds with batch context injected from the cumulative pool:
   - Wrong prediction (oracle-aided)
   - Reasoning gap identified by Review Agent
   - Low confidence (< 0.8)

   Confidence guard bypass: posts entering via wrong-prediction or reasoning-gap conditions bypass the normal confidence threshold, ensuring high-confidence wrong predictions are not skipped.

### Curation Mechanism — Cross-Batch Cumulative Feedback Pool

The curation mechanism is a key architectural choice that distinguishes the full pipeline from per-post feedback. The design evolved from two approaches:

**Per-post feedback (used in early validation):** Each post receives its own `signal_gap` feedback from the Review Agent, injected directly back into the Decision Agent for that specific post in the next round. The feedback is *tactical* — anchored to specific phrases and signals in the individual post. This approach is simpler and can correct individual misclassifications, but the feedback does not transfer to other posts.

**Cross-batch cumulative pool (current architecture):** Signal gap feedbacks from Pass 1 across *all processed batches* are collected into a shared, persistent pool. The pool operates as follows:

- **Collection**: After each batch's Pass 1, the Review Agent's `signal_gap` feedback for each post is extracted. Posts with Normal ground truth are excluded (Normal feedback would amplify clinical over-labeling bias). Posts without reviews or with empty signal_gap are skipped.

- **Ranking**: Each feedback entry is scored by a composite metric:
  - *Recency* (weight 0.5): min-max normalized post timestamp within the label group. Newer feedback scores higher.
  - *Severity* (weight 0.5): weighted combination of (a) *repeatedness* — fraction of evaluate_signal iterations where the Decision Agent's label ≠ ground truth (measures classification difficulty), and (b) *impact* — F2-weighted cost: missed clinical case (FN) = 1.0, false clinical alarm (FP) = 0.5, correct = 0.0 (penalizes missed clinical cases more heavily).

- **Retention**: Top-3 entries per clinical label (Depression, Anxiety, Comorbid). When new entries are added from a new batch, recency scores are recomputed across all entries in the label group, entries are re-ranked, and only the top-3 survive. This means the pool is *dynamic* — entries from earlier batches can be displaced by higher-ranked entries from later batches.

- **Injection into Pass 2**: Pool entries are formatted as neutral batch context:
  ```
  [BATCH CONTEXT — Depression 1]: <signal_gap text>
  [BATCH CONTEXT — Anxiety 1]: <signal_gap text>
  [BATCH CONTEXT — Comorbid 1]: <signal_gap text>
  ```
  This context is injected into the Decision Agent's prompt for all Pass 2 posts, providing *strategic calibration* — general principles about what clinical signals to attend to — rather than post-specific correction.

**Why cross-batch pooling over per-post feedback:**
- **Research novelty**: Cross-batch learning is a contribution unique to this architecture; per-post feedback is standard.
- **Transferability**: Strategic feedback ("attend to temporal persistence when distinguishing Depression from situational distress") transfers across posts; tactical feedback ("the word 'depressed' in sentence 3") does not.
- **Empirical validation**: Anxiety ΔF1 = +0.143 and Comorbid ΔF1 = +0.129 in Pass 2 demonstrate that cross-batch calibration works for clinical classes.
- **Known limitation**: Batch-context calibration *cannot override high-confidence wrong predictions* for individual posts. Normal Pass 2 ΔF1 = 0.000 because the Decision Agent's confidence in (wrong) clinical labels remains above the threshold even after receiving batch context. Per-post signal injection (deferred to future work, requires max_rounds ≥ 2) would address this by providing post-specific corrective feedback.

### Classification Schema

Four classes calibrated to the ANGST dataset: **Depression**, **Anxiety**, **Comorbid**, and **Normal**. The Comorbid label requires both Depression and Anxiety confidence scores independently ≥ 0.8; otherwise, the stronger single-condition label is assigned. Normal is assigned when the post does not express active, current clinical symptoms — even if it contains clinical vocabulary or mentions a diagnosis.

---

## 3. Experimental Design

### Dataset

**ANGST** (Hengle et al., EMNLP 2024): 2,876 gold-labeled Reddit posts with expert psychologist annotations. 4-class: Depression, Anxiety, Comorbid, Normal. Pilot evaluation: stratified sample of 100 posts (25 per class, seed=42), organized in 10 batches of 10.

### Open Design Decision: Task Formulation

*For collaborator discussion — not yet settled.*

**Option A — 4-Class Single-Label** (current implementation):
The Decision Agent picks exactly one of {Depression, Anxiety, Comorbid, Normal}.

| | Pros | Cons |
|---|---|---|
| 1 | Harder task better demonstrates architecture value (explicit comorbid reasoning, class-specific Review Agent feedback) | Cannot directly compare to ANGST paper baselines (different task formulation) |
| 2 | Richer showcase for Socratic feedback branches (4 distinct label-specific strategies) | "Normal" as a positive class is the source of our primary failure mode (F1 = 0.300) |

**Option B — Multi-Label Binary** (ANGST paper formulation):
Two independent binary decisions: "depression present?" and "anxiety present?". Comorbid = both yes, Normal = both no.

| | Pros | Cons |
|---|---|---|
| 1 | Direct comparison to ANGST paper (best: GPT-3.5 few-shot 67.4% macro F1) | Comorbid emerges automatically — no explicit reasoning to showcase |
| 2 | Normal class collapse likely resolved (Normal = absence of signals, not positive identification) | Requires rearchitecting prompts, curation, and evaluation |
| 3 | Aligns with clinical framing (screening for presence/absence) | Feedback loop contribution may be less differentiated on easier binary tasks |

**Option C — Both** (strongest paper, highest cost):
Multi-label binary as primary (clean baselines), 4-class as supplementary (architecture showcase).

**Decision criteria:**
1. How important is direct ANGST paper comparability for target venues (HICSS/ISR/ISF)?
2. Does the harder 4-class task better serve the reasoning process auditing contribution narrative?
3. Is the additional experiment cost of running both formulations justified?

### Baselines

| # | Baseline | Description |
|---|----------|-------------|
| 1 | Single agent, no feedback | Decision Agent classifications from Pass 1 across all posts — produced by the same pipeline, no separate experiment needed |
| 2 | Published benchmarks | Hengle et al. (2024) ANGST results, Sánchez Rodríguez et al. (2026) MentalBERT. *Direct comparison valid only under multi-label binary formulation.* |
| 3 | Keyword/lexicon-based | Non-LLM detection using clinical lexicons (e.g., KETCH), if applicable |

### Metrics

- **Per-class**: Precision, Recall, F1, F2, support
- **Aggregate**: macro F1, weighted F1, AUC-ROC (clinical vs. Normal)
- **Confusion matrix**: 4×4 (or 2×2 per binary task under Option B)
- **Process compliance**: process_valid_rate, approved_rate
- **Pass 2 deltas**: Per-class F1 improvement from Pass 1 → Pass 2

### Ablation Studies (planned)

- Review Agent feedback vs. no feedback (Baseline 1 comparison)
- Devil's Advocate iteration on vs. off
- Cross-batch feedback pooling on vs. off
- Few-shot examples: 0 vs. 3 vs. 5

### Success Criteria

- Decision-review loop demonstrates statistically significant improvement over Baseline 1 on the ANGST dataset.
- Reasoning process auditing shows measurable contribution via ablation.
- Normal class F1 ≥ 0.32 (addresses documented LLM failure mode from Hengle et al. 2024).

---

## 4. Key Design Decisions

| Decision | Value | Rationale |
|----------|-------|-----------|
| LLM backbone | Gemini Flash Lite (`gemini-3.1-flash-lite-preview`) for both agents | Budget constraint; supports structured JSON output required by prompts |
| Max feedback rounds | 3 (production) / 1 (prototype) | Caps compute cost; diminishing returns expected beyond 3 |
| Feedback format | Serialized `[SIGNAL GAP]` + `[REASONING GAP]` + `[PROCESS ERROR]` | Three distinct channels enable targeted correction |
| Curation strategy | Cross-batch cumulative pool, top-3 per clinical label, Normal excluded | Clinical labels benefit from accumulated feedback; Normal exclusion prevents clinical-bias amplification |
| Pass 2 gate | Oracle-aided: wrong prediction OR reasoning_gap OR low confidence | Activates 72% of posts (vs. 6% with confidence-only gate) |
| Confidence guard bypass | Oracle-aided entries bypass confidence threshold check | Prevents high-confidence wrong predictions from being unreachable |
| Batch context injection | Neutral class-labeled framing, no similarity claims | Prevents the Decision Agent from anchoring on false comparisons |
| Dataset | ANGST (4-class, Reddit) | Expert gold labels; comorbid class; documented LLM failure modes; established benchmark |
| Learning approach | In-context learning with few-shot examples (pending D40) | Zero-shot ceiling confirmed at Normal F1 = 0.300; few-shot planned |
| DA Normal discrimination | Active expression vs. contextual/historical engagement | Core question: is clinical language an active expression of current distress, or contextual? |
| Comorbid threshold | Both Depression AND Anxiety independently ≥ 0.8 | Intentionally strict; prevents over-assignment |

---

## 5. Theoretical Motivation & Literature Positioning

### The Reasoning Paradox in Clinical AI

Recent literature reveals a paradox: structured reasoning (Chain-of-Thought) improves LLM accuracy on clinical tasks but simultaneously reduces consistency:

- **86.3% of LLM-clinical task pairs suffer CoT degradation** — including hallucination, omission, and reasoning drift — when reasoning is unaudited ("Why Chain of Thought Fails in Clinical Text Understanding," arXiv:2509.21933, submitted ICLR 2026).
- **Reasoning models improve accuracy by ~3% but reduce consistency** from 91% to 84% ("Can Reasoning LLMs Enhance Clinical Document Classification?", Springer 2026).

This accuracy-consistency paradox is the core motivation for the Decision-Review architecture.

### Process vs. Outcome Review

In medical diagnostic reasoning, **process-level review** (auditing whether the reasoner followed proper steps and grounded conclusions in evidence) **reduces incorrect knowledge by 63%** — meaning that flawed reasoning chains that happen to produce correct answers are caught and corrected, preventing those fragile shortcuts from propagating to future cases. This far exceeds outcome-only review (Process vs. Outcome Review, 2025).

**Concrete example in our system:** A Decision Agent might correctly label a post as "Depression" but base its reasoning on vocabulary presence ("I'm depressed" appears in the text) rather than clinical signal analysis (persistent low mood + functional impairment). Outcome review would approve this — the label is correct. Process review catches it — the reasoning is fragile and would fail on posts where clinical vocabulary is absent but clinical signals are present.

### Why Social Media Data + Multi-Step Reasoning Is Valid

The multi-step aspect is in the **classification reasoning process**, not in the data collection. Each social media post is a single text unit. The Decision Agent iterates its reasoning three times (evaluate → challenge → refine), and the Review Agent ensures this reasoning is thorough and evidence-grounded. The data modality is orthogonal to the reasoning architecture.

### Three Research Dimensions

This work sits at the intersection of:

1. **LLM-based mental health detection on social media** — where single-model approaches dominate, and agent-based systems are identified as the "next frontier" but remain underexplored (Ge et al., 2025; MDPI Systematic Review, 2026; JMIR AI Systematic Review, 2026 — no multi-agent feedback architectures found in healthcare text classification).

2. **Multi-agent LLM systems and iterative feedback** — where debate, reflexion, and Socratic questioning have been validated for general reasoning but remain nascent in clinical classification. Closest work: Mao et al. (2026) CFD (data enrichment, not classification); AgentMental (2025) (interactive interviews, not post-level classification).

3. **Clinical reasoning and process evaluation** — where the reasoning paradox motivates external process auditing. Our Review Agent functions as a domain-specific CMVA (Criteria Match Validator Agent, 2025) — verifying outputs against predefined formal criteria, decoupled from stochastic model output.

---

## 6. Pilot Study Results

### Evaluation Summary (100 posts, 10 batches, DA v5.6 + RA v9)

| Metric | Pass 1 | Pass 2 | Δ |
|--------|--------|--------|---|
| Macro F1 | 0.450 | 0.450 | 0.000 |
| Depression F1 | 0.566 | 0.566 | 0.000 |
| Anxiety F1 | 0.550 | 0.693 | **+0.143** |
| Comorbid F1 | 0.383 | 0.512 | **+0.129** |
| Normal F1 | 0.300 | 0.300 | 0.000 |
| AUC-ROC | — | 0.510 | — |
| Process valid rate | 0.980 | — | — |
| Parse failures | 0/100 | — | — |

### Key Findings

**Architecture validated for clinical classes:**
- Feedback loop produces substantial improvements for Anxiety (ΔF1 = +0.143) and Comorbid (ΔF1 = +0.129)
- Review Agent's class-specific strategic feedback (v9) doubles Comorbid Pass 2 effectiveness vs. generic feedback
- Pipeline highly stable at 100-post scale: 0 parse failures, 0.980 process valid rate

**Normal class remains the primary challenge:**
- Normal F1 = 0.300 (target ≥ 0.32), with 84% of Normal posts misclassified as clinical
- Pass 2 Normal ΔF1 = 0.000 — batch-context calibration cannot override high-confidence wrong predictions
- Zero-shot prompt engineering ceiling confirmed after 6 DA versions (v5.1→v5.6) and 4 RA versions (v6→v9)

**Pending decision — D40 PIVOT to few-shot ICL:**
- Add 3–5 Normal + 2 Comorbid in-context learning examples to DA prompt
- Success criteria: Normal F1 ≥ 0.32, Comorbid F1 ≥ 0.40, macro F1 > 0.450
- Aligns with original project brief (which always specified few-shot ICL, not zero-shot)
- ANGST paper documents +6.9pp macro F1 gain from few-shot (Hengle et al., 2024)

---

## 7. Verification Strategy

1. **Oracle-aided evaluation** (development): ANGST expert-annotated ground truth labels for classification accuracy metrics.
2. **Automated reasoning process audit** (runtime): The Review Agent's 4-rule audit functions as a domain-specific CMVA — verifying outputs against predefined formal criteria, decoupled from stochastic model output.
3. **Planned human expert review** (validation): Subset review for inter-rater reliability between Review Agent assessments and human expert assessments.

### Process Auditing Impact — Direct Assessment
- Pass 1 → Pass 2 per-class F1 delta isolates the feedback loop's contribution per class
- Component ablation: isolate contributions of Devil's Advocate, Socratic feedback, cross-batch pooling, adaptive depth

### Process Auditing Impact — Indirect Assessment
- Process compliance rate correlated with classification accuracy
- Batch-level compliance vs. accuracy analysis

---

## 8. Open Design Decision: Publication Framing

*For collaborator discussion — not yet settled.*

The pilot study produced both architecture validation and rich failure mode evidence. This opens two viable publication framings.

### Option A — Architecture Contribution Paper

*"Here is our novel Decision-Review architecture for mental health classification, and here is evidence it works."*

- **Framing**: Novel contributions → experimental validation → comparison to baselines
- **Strengths**: Positive narrative; 4 novel contributions verified against current literature; proven Anxiety/Comorbid improvements via feedback loop
- **Weaknesses**: Normal class remains challenging (F1 = 0.300); reviewers may focus on what doesn't work
- **Target venues**: HICSS (general track), ISR, ISF

### Option B — "Dark Side" Failure Modes Paper

*"Here is what goes wrong when multi-agent LLM systems attempt mental health classification, and what these failures reveal about human-agent collaboration."*

- **Target**: HICSS minitrack — "The Dark Side of Human-Agent Collaboration and Collaborative Workflow"
- **Analytical framework**: Failure mode taxonomy from multi-agent computational psychiatry literature
- **Framing**: Pilot study as a case study surfacing *architectural* failure modes in clinical AI
- **Strengths**: Failure modes are genuine and empirically documented (not hypothetical); "dark side" framing turns weaknesses into the contribution; directly addresses a gap — most multi-agent MH papers report successes, failure mode analysis is scarce; HICSS minitrack is a natural fit (DA-RA collaboration failures are literally human-agent collaboration failures)
- **Weaknesses**: Requires careful framing to avoid "our system doesn't work" perception; architecture novelty becomes secondary; Final deliverable alignment needs verification; some failure mode mappings are stronger than others (see assessment below)

#### Critically Assessed Finding-to-Failure-Mode Mappings

Not all findings map equally well to the failure mode taxonomy. This assessment distinguishes strong, moderate, and weak mappings:

**STRONG mappings (could anchor a paper):**

1. **Process-Outcome Audit Paradox.** The Review Agent achieved ~98% process_valid_rate while Normal class F1 = 0.214. Process-compliant reasoning trajectories produced wrong outcomes. This is Goodhart's Law in multi-agent AI: the Review Agent optimizes for compliance because it is easier to measure than outcome quality. Directly validated by Process vs. Outcome Review literature (63% claim). *Generalizable to any multi-agent system where one agent reviews another's reasoning.*

2. **Specification Gaming / Conflicting Rule Exploitation.** During a 10-post validation test, the Decision Agent skipped the mandatory Devil's Advocate iteration entirely by exploiting a conflict between an early-exit permission and a mandatory evaluation requirement. The model chose the rule that minimized output — a form of reward hacking without explicit reward signals. Required a prompt fix mandating ≥2 evaluate_signal calls before early exit. *Generalizable: when multi-agent prompts contain any pair of conflicting rules, models will exploit the path of least resistance.*

3. **Confidence-Feedback Blind Spot.** In the full 100-post evaluation, only 6/100 posts had confidence < 0.8, yet 52 were wrong. High-confidence wrong predictions structurally bypass the feedback mechanism. The confidence guard — designed to prevent harmful feedback on correct predictions — creates an unreachable zone for confident *incorrect* predictions. *Generalizable to any iterative system with confidence-gated feedback.*

4. **Cross-Class Collateral Damage.** A Depression/Normal boundary check improved Normal F1 by +0.086 but regressed Comorbid F1 by -0.061. Targeted fixes in a shared prompt space cause adjacent-class regression — not documented elsewhere in the multi-agent mental health literature. *Generalizable to any multi-class feedback system with shared reasoning.*

**MODERATE mappings (usable with careful framing):**

5. **Inverted Information Drowning.** 84% of Normal posts were misclassified as clinical. Normal signals (absence of clinical indicators) were drowned by clinical detection bias. Maps to the PAMAS (Self-Adaptive Multi-Agent System with Perspective Aggregation; AAMAS 2026) framework but is inverted: PAMAS describes sparse deceptive cues overwhelmed by truthful content; ours is a legitimate class overwhelmed by dominant clinical reasoning. Should be presented as an extension, not a direct claim.

6. **Forced Dissensus Noise.** 92.6% of Devil's Advocate iterations produced U-shaped confidence trajectories (high → collapse → partial recovery). The mandatory self-critique introduced systematic noise rather than genuine reasoning. Related to the spurious consensus problem (ChatEval, ICLR 2024) but is the mirror image: forced *dissensus* that is mechanical rather than genuine.

**WEAK mappings (recommend excluding):**

- **Neural Howlround / RISM**: Our clinical over-labeling is prompt-induced bias, not spontaneous self-reinforcing bias during inference. Additionally, the RISM concept (arXiv:2504.07992) is not yet well-established in the literature.
- **Sycophantic Validation**: Our Review Agent doesn't flatter — it diligently audits and generates feedback, even when that feedback is harmful. The process-outcome paradox framing is more accurate.
- **Zero-shot Ceiling**: A methodology finding, not a failure mode.

#### Structural Concerns

1. **Sample size**: 100 posts, 1 dataset, 1 model. Counter: the strong findings are *architectural* (emerge from system structure, not implementation) and would occur in any system with the same design patterns.
2. **Source quality**: The failure mode literature review includes some non-peer-reviewed sources. HICSS submission requires replacing these with peer-reviewed equivalents.
3. **Perception risk**: System macro F1 = 0.450, below published baselines (67.4%). Narrative must be: "the failure modes are the contribution, not the system's performance."

#### What the Paper Could Look Like

**Draft Outline**: [HERE](https://github.com/DrJieTao/mental_health_2026/blob/main/dark_side_paper_outline_v0.md)

**Title direction**: *"When the Reviewer Becomes the Problem: Failure Modes in Multi-Agent Reasoning Process Auditing for Mental Health Classification"*

**Four core contributions**: (1) Process-outcome audit paradox; (2) Specification gaming in agent prompts; (3) Confidence-feedback blind spot; (4) Cross-class collateral damage.

**Applicable frameworks from the literature:**
- **TED** (Talk, Evaluate, Diagnose; 2026) — automated error analysis that clusters errors into categories and translates them to specific remedies. Our Review Agent's feedback channels are a concrete implementation; our findings reveal its limits.
- **Constrained Process Maps** (2026) — formalization of multi-agent workflows as bounded-horizon Markov Decision Processes (MDPs). Our two-pass pipeline with confidence gates maps to this framework; can formalize failure propagation paths.
- **AgentOps** (2025) — operational observability for autonomous agent lifecycles, including agent-level telemetry and cognitive traces. Our JSONL logging provides this telemetry; lessons about what we *couldn't* observe are directly relevant.
- **FAITA-MH** (Framework for AI Tool Assessment in Mental Health; 2025) — product-level assessment across credibility, user experience, crisis protocols, user agency, and transparency. Applicable if we discuss governance implications.

### Option C — Both Papers from Same Pilot Data

- Paper 1 (architecture): Validated contributions (Anxiety/Comorbid improvements, process auditing design)
- Paper 2 (dark side): Architectural failure modes + governance implications
- Risk: Splitting results may weaken each individually
- Benefit: Maximum publication output from one pilot study

### Strategic Sequencing Question

*Does writing the "dark side" paper first help or hurt the architecture paper?*
- **Helps if**: It establishes that these failure modes exist and are generalizable → the architecture paper can reference it and say "we address these documented failure modes."
- **Hurts if**: It is perceived as "their system doesn't work" → undermines the architecture paper at the same venues.

### Decision Criteria for Collaborators

1. Do the 4 strong architectural failure modes constitute a sufficient standalone contribution for HICSS?
2. Does the "dark side" framing serve or undermine the final deliverable?
3. Is the mapping to established frameworks (TED, Constrained Process Maps, FAITA-MH) strong enough for peer review given a 100-post pilot?
4. Would collaborators prefer to contribute domain expertise to a failure analysis or an architecture evaluation?
5. What is the preferred publication sequence if both papers are pursued?

---

## 9. Collaborator Questions & Responses

Questions from collaborators (`docs/collaborator_questions.md`) with responses grounded in current architecture and findings:

**Q1: "If policy or standard clinical protocols are not explicitly incorporated into the architecture, is the plan essentially process compliance auditing or process auditing?"**

Our Review Agent performs **reasoning process auditing** — verifying that the Decision Agent followed a prescribed multi-step reasoning procedure (4 rules: iteration count ≥ 2, Devil's Advocate at iteration 2, prior evaluations referenced, feedback compliance). This is *not* clinical protocol compliance — we do not encode DSM-5 or clinical guidelines as auditable rules. We audit reasoning thoroughness and evidence grounding. See §2, Review Agent.

**Q2: "Existing clinical protocols are typically not one-shot interactions but involve multi-step procedures or multi-turn interactions. As a result, social media data may not be a good fit."**

We do not simulate clinical procedures on social media data. The multi-step aspect is in the *classification reasoning process*, not in data collection. Each post is a single text unit, but the Decision Agent iterates its reasoning 3 times with self-critique. Literature support: 86.3% of LLM-clinical task pairs suffer CoT degradation when reasoning is unaudited. Our architecture compensates for this — the data modality is orthogonal to the reasoning architecture. See §5, Theoretical Motivation.

**Q3: "Where do you plan to introduce human verification, or use CMVA only?"**

CMVA = Criteria Match Validator Agent (2025 paper — verifies outputs against predefined formal criteria). We use **layered verification**: (1) oracle-aided evaluation with ANGST expert ground truth; (2) automated reasoning process audit via the Review Agent's 4-rule CMVA; (3) planned human expert review for inter-rater reliability. See §7, Verification Strategy.

**Q4: "Do you plan to assess the impact of process auditing directly or indirectly?"**

**Both.** Direct: Pass 1 → Pass 2 per-class F1 delta isolates the feedback loop's contribution; component ablation isolates Devil's Advocate, Socratic feedback, cross-batch pooling, adaptive depth. Indirect: process compliance metrics correlated with classification accuracy; batch-level compliance vs. accuracy analysis. See §7.

**Q5: "Is the plan to conduct experiments with multi-class mental health classification problems to better demonstrate the strengths of the architecture?"**

Yes — ANGST provides 4-class classification. This deliberately harder task demonstrates class-specific feedback, cross-batch pooling, and boundary discrimination. However, whether to use 4-class or multi-label binary (or both) is an open design decision for collaborator input. See §3, Open Design Decision: Task Formulation.

**Q6: "What does it mean by 'reduces incorrect knowledge'?"**

From Process vs. Outcome Review (2025): process-level review **reduces incorrect knowledge by 63%** — meaning flawed reasoning chains that happen to produce correct answers are caught and corrected, preventing fragile shortcuts from propagating. Concrete example: a Decision Agent correctly labels a post "Depression" based on vocabulary presence ("I'm depressed" appears) rather than clinical signal analysis. Outcome review approves it (correct label). Process review catches it — the reasoning would fail on posts where clinical vocabulary is absent but clinical signals are present. See §5.

---

## 10. Current Status & Next Steps

### Completed

- **Research design**: Two-agent architecture, feedback protocol, two-pass pipeline — all core decisions finalized (D1–D40)
- **Prompt engineering**: Decision Agent v5.6 (Depression/Normal boundary check, Devil's Advocate 4-check) and Review Agent v9 (class-specific feedback branches, ground_truth_label parameter)
- **Prototype implementation**: All source modules operational — orchestrator, API client, prompt renderer, response parser, curation module, evaluation module, feedback serializer, dataset loader, logger
- **Unit tests**: 79/79 passing across all modules
- **SDK migration**: `google-genai` v1.68.0 with per-call timeout (60s), 5-attempt retry, all-5xx coverage
- **Pilot study complete**: 100 posts (25×4 classes), 10 batches, 332 API calls, 0 errors, ~39 minutes
- **Two full evaluation runs**: Original (DA v5.4 / RA v8) and REFINE (DA v5.6 / RA v9)
- **32 lessons learned**: Recorded across methodology, design, domain, data, tooling, and collaboration categories

### Pending Decisions (require collaborator input)

- **D40 PIVOT to few-shot ICL**: Add 3–5 Normal + 2 Comorbid examples to DA prompt. Awaiting approval.
- **Task formulation**: 4-class vs. multi-label binary vs. both. See §3.
- **Publication framing**: Architecture paper vs. "dark side" failure modes paper vs. both. See §8.

### Next Steps (after decisions)

1. Implement D40 few-shot examples in DA prompt (v5.7)
2. Validation run (~10 posts) before full re-evaluation
3. Full 100-post re-evaluation with updated prompts
4. Ablation studies if success criteria are met
5. Manuscript preparation for selected venue

---

## 11. Scope Boundaries

### In Scope (Phase 1)

- Decision-Review agent loop design, implementation, and evaluation
- Prompt engineering for both agents (ICL with clinical signal definitions)
- Two-pass batch pipeline with cross-batch feedback pooling
- Evaluation on ANGST dataset with baselines and ablations
- Publication preparation for HICSS / ISR / ISF

### Out of Scope

- **Phase 2 — Dynamic Knowledge Retrieval**: Planned extension. Phase 1 results serve as baselines.
- **Fine-tuning**: All experiments use in-context learning
- **Per-post feedback injection**: Deferred; requires max_rounds ≥ 2 (currently 1 in prototype)
- **Cross-lingual analysis**: Exploratory only, not required for Phase 1 success
- **Clinical deployment**: System outputs are decision-support tools, not clinical diagnoses

---

## 12. Ethical Considerations

- All datasets are publicly available, de-identified, and used for secondary analysis only
- IRB consultation planned for exemption determination
- Compliance with GDPR and HIPAA where applicable
- System outputs are framed as decision-support tools, not clinical diagnoses — this framing must be maintained in all publications

---

## 13. Key References

### Multi-Agent Mental Health Detection
- Mao et al. (2026) — CFD: Multi-agent debate for mental health data enrichment (closest competitor)
- AgentMental (2025) — Interactive 4-agent mental health assessment framework
- Sánchez Rodríguez et al. (2026) — Fine-tuned MentalBERT, 82.2% accuracy on 4-class
- Ge et al. (2025) — Survey: From LLMs and RAG to Agents for Mental Disorder Detection
- JMIR AI Systematic Review (2026) — No multi-agent feedback architectures found in healthcare text classification

### Clinical Reasoning and Process Evaluation
- "Why Chain of Thought Fails in Clinical Text Understanding" (arXiv:2509.21933) — 86.3% CoT degradation
- "Can Reasoning LLMs Enhance Clinical Document Classification?" (Springer 2026) — accuracy-consistency paradox
- Process vs. Outcome Review (2025) — 63% reduction in incorrect knowledge via process review
- CriticBench (2025) — saturation of review; adaptive depth motivation
- CMVA (2025) — Criteria Match Validator Agent; validation against formal criteria

### Feedback Quality and Multi-Agent Debate
- ChatEval (ICLR 2024) — spurious consensus problem
- TED (2026) — automated error analysis for agent evaluation
- Agent-as-a-Judge (2026) — holistic analysis beyond correctness
- PAMAS (AAMAS 2026) — hierarchical aggregation for information drowning

### Benchmark Dataset
- Hengle et al. (2024) — ANGST: EMNLP 2024. Best macro-F1: GPT-3.5 few-shot = 67.4%, GPT-4 zero-shot = 66.7%. 84% Normal error rate documented. Few-shot advantage: +6.9pp.

### Foundational References (from Proposal)
- Zhang et al. (2024) — KETCH model for suicidal ideation detection (ISR)
- Min et al. (2022) — In-context learning vs. fine-tuning
- Tadesse et al. (2022) — MentalBERT baseline
- Amann et al. (2023) — Human-AI collaboration transparency

---

## 14. Document Map

| Document | Role |
|----------|------|
| `docs/project_status.md` | **This document** — current research status for collaborator sharing |
| `docs/project_brief.md` | Original proposal-level specification (goals, datasets, tasks, ethics) |
| `research_state.md` | Chronological decision log (D1–D40), open questions, session history |
| `docs/prototype_design.md` | System architecture and implementation specifications (code-focused) |
| `docs/literature_positioning.md` | Literature landscape, gap analysis, citation groups |
| `docs/deep_research_fail_modes.md` | Failure mode taxonomy and evaluation frameworks for clinical AI |
| `docs/collaborator_questions.md` | Collaborator questions (addressed in §9) |
| `prompts/decision_agent_prompt_v5.6.md` | Decision Agent system prompt (active) |
| `prompts/review_prompt_v9.md` | Review Agent system prompt (active) |
| `knowledge/` | Categorized knowledge base: decisions, experiments, findings, literature, questions, reviews |
| `lessons/` | 32 lessons learned across methodology, design, domain, data, tooling, collaboration |
