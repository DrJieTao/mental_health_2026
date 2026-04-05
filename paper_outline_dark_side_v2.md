# Seven Failure Modes of Multi-Agent LLM Systems for Mental Health Classification
## An Empirical Taxonomy from Decision-Review Architectures

**Target**: HICSS 2027 — "The Dark Side of Human-Agent Collaboration and Collaborative Workflow"
**Format**: 10-page (~8,000 words body, excluding references)
**Primary backbone**: Gemma 4 E4B (local, quantized Q6_K). Cross-backbone validation: Gemini Flash Lite, Qwen3.5-2B.
**Central claim**: Multi-agent systems introduce structural failure modes that cannot be diagnosed from outcome metrics alone, and that process-level auditing — while necessary — is also insufficient. Seven failure modes, each with a design principle, derived from 30 sessions of systematic probing.

---

## §1. Introduction (~700 words)

**Paragraph 1 — The Reasoning Paradox** (~100 words): Open with: structured reasoning (CoT, ToT) improves accuracy in 86.3% of LLM-clinical task pairs but also introduces hallucination, omission, and reasoning drift (arXiv:2509.21933). Reasoning models improve accuracy ~3% but reduce consistency — 84% vs 91% (Springer, 2026). Multi-agent review was proposed as the answer. We show it introduces its own failure modes.

**Paragraph 2 — The Multi-Agent Promise and the Empirical Gap** (~200 words): Process-level review reduces incorrect knowledge by 63% (Process vs. Outcome Review, 2025). Agent-based mental health systems are "emerging but underexplored" (MDPI 2026, Ge et al. 2025). No prior work has systematically characterized what goes wrong inside a real multi-agent clinical AI system. Gap: the field lacks empirical failure mode catalogs from deployed systems.

**Paragraph 3 — This Paper** (~200 words): Seven failure modes from 30 sessions of systematic probing of a Decision-Review architecture for mental health classification on social media (ANGST dataset, 100 posts, 5 replicate runs per condition). Three architectural scopes: single-agent (FM-1, FM-4, FM-7), multi-agent interaction (FM-2, FM-3), system-level (FM-5, FM-6). Each yields a design principle. The principles converge on a common requirement: externalized, structured clinical knowledge.

**Paragraph 4 — Paper Organization** (~100 words): Standard roadmap. Organizing principle: from failure observation to generalizable principle.

---

## §2. Theoretical Background (~1,400 words)

### 2.1 AI/Agent Advancement in Mental Health Detection (~450 words)
Evolution: lexicon → fine-tuned transformers (MentalBERT, Sánchez Rodríguez 2026) → LLM prompting (Cognitive-Mental-LLM 2026) → agent era (nascent). Closest multi-agent work: Mao et al. 2026 (CFD debate for data enrichment), AgentMental 2025 (interview simulation). Neither does direct classification with process auditing. Gap: no empirical failure mode catalog from a real multi-agent clinical system.

Key citations: Ge et al. 2025, MDPI 2026, Sánchez Rodríguez 2026, Mao et al. 2026, AgentMental 2025, ANGST (Hengle et al. EMNLP 2024)

### 2.2 Process vs. Outcome Review in Clinical AI (~500 words)
Reasoning paradox (arXiv:2509.21933, Springer 2026). Process review theory: 63% reduction in incorrect knowledge. Implementation analogs: CMVA, Process Reward Models That Think, CriticBench (saturation of review — inverse of our FM-3). Agent-as-a-Judge. Boundary condition our paper reveals: process_valid_rate = 0.980 while Normal F1 = 0.300.

Key citations: "Why CoT Fails" 2025, Reasoning LLMs 2026, Process vs. Outcome Review 2025, Pathology-CoT 2025, CriticBench, Agent-as-a-Judge 2025

### 2.3 Failure Mode Frameworks for Multi-Agent Clinical Systems (~450 words)
TED Red Teaming (arXiv:2602.19948) — structural analog to our systematic probing. PAMAS (arXiv:2602.03158) — information drowning. AgentOps RCA (arXiv:2508.02121) — origin taxonomy. Constrained Process Maps / MDP (arXiv:2602.02034). FAITA-MH, READI, MIND-SAFE. Neural Howlround / RISM (arXiv:2504.07992). Sycophantic validation (Stanford 2025). Gap: frameworks exist but empirical evidence from real systems does not.

---

## §3. Artifact Design (~600 words)

**Figure 1**: Three-level failure mode taxonomy (pyramid). Base: single-agent failures (FM-1, FM-4, FM-7). Middle: multi-agent interaction failures (FM-2, FM-3). Top: system-level failures (FM-5, FM-6). Place at the top of §3 to anchor the architecture description in the failure landscape — the reader sees what will go wrong before seeing how the system is designed.

### 3.1 Architecture Overview (200 words)
Decision Agent (DA): 3-iteration evaluate_signal with mandatory Devil's Advocate at iteration 2, finalize_annotation. Simulated tool-use protocol. Review Agent (RA): flat JSON review with signal_gap (pattern-typed) and process_error. Two-pass pipeline: Pass 1 screens all posts; Pass 2 gives eligible posts a second DA call with RA feedback. Cross-batch curation pool.

**Figure 2**: Architecture diagram — DA → RA → Pass 2 loop with curation pool.

### 3.2 Design Rationale (200 words)
Ground each choice in literature: mandatory DA (ChatEval), external RA audit (Process vs. Outcome Review), two-pass adaptive depth (CriticBench saturation), cross-batch curation (PAMAS information drowning).

### 3.3 The Curation Pipeline as Proto-Knowledge-Retrieval (200 words)
Frame pool as a precursor to structured knowledge retrieval. FM-6 reveals where it succeeds and where it fails.

---

## §4. Systematic Probing Methodology (~700 words)

### 4.1 Local Model Selection and Evaluation Protocol (250 words)
**Why local models**: (1) Privacy — sensitive clinical text stays local; (2) Reproducibility — fixed weights, no silent API updates; (3) Experimental control — full parameter transparency, enables sensitivity analysis. Primary: Gemma 4 E4B (Q6_K, Apple Silicon 64 GB). Cross-backbone validation in §6.

Dataset: ANGST (Hengle et al. EMNLP 2024), 4-class, 100-post stratified test sample (25/class, seed=42). Important: "Normal" is post-level temporal judgment, not population-level health claim — all users self-disclosed clinical history. Metrics: per-class F1, macro F1, Pass 2 correction rate, parse failure rate.

### 4.2 Experiment Progression as Systematic Probing (500 words)
Four-phase escalation. Each phase: fix the dominant failure → test → reveal the next layer.

| Phase | Experiment | N | Model | FM Discovered |
|-------|-----------|---|-------|---------------|
| 0 | EXP-002 | 1 | Gemini | None (mechanical validation) |
| 1 | EXP-007 | ~80 | Nano-model | FM-1 (first obs), FM-2 (first indication) |
| 2 | EXP-003/003c | 10 | Gemini | FM-4 (discovered and closed) |
| 3 | EXP-004 (5 runs) | 100 | Gemini | FM-1–3, FM-5–6 confirmed at scale |
| 4 | EXP-008 (15 runs) | 100 | Gemma 4 E4B | FM-1–3, FM-5–7 cross-backbone; FM-7 new |

**Figure 3**: Experiment progression timeline with FM discovery points.

---

## §5. Failure Mode Analysis (~3,200 words)

**Table 2**: FM Summary

| FM | Name | Scope | Theoretical Construct | Design Principle |
|----|------|-------|-----------------------|-----------------|
| FM-1 | Overconfident Single-Pass Classification | Single-agent | Reflexion / self-correction | External knowledge grounding required |
| FM-4 | Specification Gaming | Single-agent | Rule conflict exploitation | Mandatory steps must be immune to optional exits |
| FM-7 | Architectural Complexity Overhead | Single-agent | Scaffolding tax / XAI tradeoff | Externalize reasoning into knowledge stores |
| FM-2 | Process-Outcome Audit Paradox | Multi-agent | Goodhart's Law | Process rules must encode discriminative criteria |
| FM-3 | Confidence-Feedback Blind Spot | Multi-agent | Confidence calibration | Feedback activation must decouple confidence from correctness |
| FM-5 | Cross-Class Collateral Damage | System-level | Shared prompt coupling | Class-specific reasoning paths required |
| FM-6 | Generic Knowledge Pooling Ceiling | System-level | Knowledge retrieval limits | Per-instance retrieval beyond batch calibration |

### 5.1 Where Self-Correction Fails (~1,000 words)

**FM-1: Overconfident Single-Pass Classification** (~450 words)
- What: 95–99% of posts terminate via CONFIDENCE_THRESHOLD despite accuracy 0.34–0.36. DA performs the form of deliberation without the substance.
- Evidence: EXP-008 C1 ablation (5 runs, Gemma): mean F1=0.368, SD=0.025. U-shape rate 2–5% (confidence guard prevents original pattern). EXP-007 nano-model: 91.9% U-shaped confidence. Cross-backbone confirmed (F-043, F-050, F-057).
- Design Principle (DP-1): Single-agent self-correction is insufficient. External knowledge grounding required.

**FM-7: Architectural Complexity Overhead** (~400 words)
- What: Tool-use scaffold imposes overhead that can interfere with native reasoning. Baseline F1=0.643 vs DA F1=0.368 on Gemma (27.5pp gap); negligible on Gemini (-0.006).
- Evidence: Cross-backbone interaction effect (F-056, F-057, F-044). RA v9→v10 redesign: 97%→0% parse failure (F-055). Gemma 4 is a native reasoning model with `<|think|>` mode (Google 2026). Sprague et al. 2024: CoT reduces accuracy up to 36.3% when deliberation is counterproductive.
- XAI tradeoff: Scaffold provides interpretability at the cost of performance — same tradeoff at prompt architecture level. Open question: how to design scaffolds that complement rather than compete with native reasoning.
- Design Principle (DP-7): Externalize reasoning complexity into knowledge stores. Separate reasoning from memory.

**FM-4: Specification Gaming** (~300 words, Identified/Closed)
- What: DA exploited conflicting prompt rules to skip mandatory iterations (10/10 posts).
- Evidence: EXP-003. Resolved with DA v5.2 (D22), validated in EXP-003c.
- Design Principle: Mandatory steps must be immune to co-present optional exits.

### 5.2 Where Review Fails to Correct (~900 words)

**FM-2: Process-Outcome Audit Paradox** (~500 words)
- What: process_valid_rate=0.980 while Normal F1=0.300. RA optimizes for compliance over correctness.
- Evidence: EXP-008 C2 ablation: 66 posts enter Pass 2, 5 corrections (7.6%), 0 regressions. RA correctly diagnosed all 21 misclassified Normal posts (F-026) but feedback had minimal corrective effect. Class-asymmetric: Comorbid correction 45% vs Normal 7% (LL-036).
- Design Principle (DP-2): Process rules must encode discriminative criteria, not generic compliance checks.

**FM-3: Confidence-Feedback Blind Spot** (~400 words)
- What: High-confidence wrong predictions structurally bypass correction.
- Evidence: Structural blind spot addressed by D32 bypass_confidence_guard (Pass 2 activation 63%). Residual: DA receives feedback but maintains prediction in 92.4% of Pass 2 posts. max_rounds=1 → 0% correction; max_rounds=2 → 7.6% correction (round-dependent resistance, LL-037).
- Design Principle (DP-3): Feedback activation must decouple confidence from correctness.

### 5.3 Where Architecture Fails to Scale (~800 words)

**FM-5: Cross-Class Collateral Damage** (~350 words)
- What: Depression/Normal boundary check improved Normal +0.086 but regressed Comorbid -0.061.
- Evidence: EXP-004 S22 (F-042, LL-032). C1 ablation per-class variance: Normal consistently lowest (mean 0.187), Comorbid depressed relative to other clinical classes.
- Design Principle (DP-5): In shared-reasoning systems, class-specific corrections propagate to adjacent classes. Class-specific reasoning paths required.

**FM-6: Generic Knowledge Pooling Ceiling** (~450 words)
- What: Pool improved Anxiety ΔF1=+0.143, Comorbid ΔF1=+0.129, Normal ΔF1=0.000.
- Evidence: EXP-004 D37 (F-040). C3 probe: no accumulation effect (first half 22.2% ≈ second half 21.4%, F-054). Zero-shot ceiling confirmed after 6 DA + 4 RA versions (F-041).
- Design Principle (DP-6): Generic pooling has a resolution limit below which per-instance retrieval is required.

### 5.4 Ablation Summary and Design Principles (~500 words)

**Table 3**: EXP-008 Ablation Results (Gemma 4 E4B, temp=0.3, 5 runs × 100 posts)

| Condition | Macro F1 (mean ± SD) | Accuracy | Pass 2 | Corrections | What it tests |
|-----------|----------------------|----------|--------|-------------|---------------|
| Baseline | 0.643 | 0.643 | — | — | Native model calibration |
| C1 (DA-only) | 0.368 ± 0.025 | 0.358 | 0 | — | FM-1, FM-7 |
| C2 (DA+RA, no pool) | [pending] | [pending] | ~66 | ~7.6% | FM-2, FM-3 |
| C3 (DA+RA+pool) | [pending] | [pending] | [pending] | [pending] | FM-6 |

**Table 4**: Seven Design Principles and Their Convergence

| DP | From FM | Requirement | Convergent direction |
|----|---------|-------------|---------------------|
| DP-1 | FM-1 | External knowledge grounding | Externalized clinical knowledge |
| DP-2 | FM-2 | Outcome-aware audit criteria | Evidence-based review |
| DP-3 | FM-3 | Confidence-independent correction | Per-instance retrieval |
| DP-4 | FM-4 | Formally consistent specifications | (Prompt-level, closed) |
| DP-5 | FM-5 | Class-specific reasoning paths | Structured knowledge routing |
| DP-6 | FM-6 | Per-instance knowledge retrieval | Domain knowledge stores |
| DP-7 | FM-7 | Externalized reasoning complexity | Reasoning–memory separation |

---

## §6. Discussion (~1,250 words)

### 6.1 Research Implications (~500 words)
- Cross-backbone validation: FM-1 confirmed on both (Gemini, Gemma). FM-7 requires cross-backbone comparison. Qwen3.5-2B rejection confirms capability floor.
- Generalizability beyond mental health: each FM generalizes to any multi-agent clinical AI.
- Reasoning–memory separation: FM-7/DP-7 connects to the emerging paradigm. Clinical knowledge should not live in prompts; it should live in external structured stores.

### 6.2 Practitioner Implications (~500 words)
- Reliability: parse failure stochasticity, per-post error isolation, silent output truncation.
- Prompt complexity cliff: hard threshold per model tier (RA v9→v10).
- Feedback loops: round resistance (LL-037), class-asymmetric correction (LL-036), confidence guard design tension (F-051).
- Architecture portability: migration required full prompt redesign, parser hardening, parameter adjustment.
- Temperature as experimental precondition: temp=0.0 → deterministic; temp=0.3 → valid variance.

### 6.3 Convergent Design Requirements for External Clinical Knowledge (~250 words)
Seven DPs converge on: externalized, structured clinical knowledge, retrievable per-instance, auditable independently. Specific artifact form is an open research question. Directions: clinical ontology structuring for retrieval, class-specific subgraph traversal, scaffold-free knowledge injection. Our FMs provide concrete evaluation criteria for any proposed architecture.

---

## §7. Conclusion (~400 words)

**Paragraph 1 — Contributions**: Seven failure modes, three levels, 30 sessions. Each yields a design principle converging on externalized clinical knowledge. "Not safer by virtue of more agents — safer when each interaction point is designed with awareness of how it can fail."

**Paragraph 2 — Limitations**: Pilot study. 100 posts, single dataset, 4B local model (privacy-motivated). Cross-backbone validation on two additional models. Oracle-aided Pass 2 gate. Zero-shot configuration. Boundaries define where next investigation should push. FMs and DPs offered as testable hypotheses.

---

## Figures and Tables

| Item | Description | Section |
|------|-------------|---------|
| Figure 1 | Three-level FM taxonomy pyramid (FM-1/4/7 → FM-2/3 → FM-5/6) | §3 (leads section) |
| Figure 2 | Architecture diagram: DA → RA → Pass 2 → curation pool | §3 |
| Figure 3 | Experiment progression timeline with FM discovery points | §4 |
| Figure 4 | Confusion matrix (EXP-008 Gemma, best run) | §5 |
| Figure 5 | Ablation box plot: F1 across C1/C2/C3 conditions | §5.4 |
| Table 1 | Experiment summary (phase, N, model, FM discovered) | §4 |
| Table 2 | FM summary (name, scope, construct, DP) | §5 opener |
| Table 3 | EXP-008 ablation results | §5.4 |
| Table 4 | Seven DPs and convergent direction | §5.4 |

---

## Word Count

| Section | Words |
|---------|-------|
| §1 Introduction | 700 |
| §2 Theoretical Background | 1,400 |
| §3 Artifact Design | 600 |
| §4 Systematic Probing Methodology | 700 |
| §5 Failure Mode Analysis | 3,200 |
| §6 Discussion | 1,250 |
| §7 Conclusion | 400 |
| **Total** | **~8,250** |

*Trim ~250 words during drafting to fit 8,000 target.*
