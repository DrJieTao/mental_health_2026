# Seven Failure Modes of Multi-Agent LLM Systems for Mental Health Classification
## An Empirical Taxonomy from Decision-Review Architectures

**Target**: HICSS 2027 — "The Dark Side of Human-Agent Collaboration and Collaborative Workflow"
**Format**: 10-page (~8,000 words body, excluding references)
**Primary backbone**: Gemma 4 E4B (local, quantized Q6_K). Cross-backbone validation: Gemini Flash Lite, Qwen3.5-2B.
**Central claim**: Multi-agent systems introduce structural failure modes that cannot be diagnosed from outcome metrics alone, and that process-level auditing — while necessary — is also insufficient. Seven failure modes, each with a design principle, derived from 30 sessions of systematic probing.

---

## §1. Introduction (~700 words)

**Paragraph 1 — The Reasoning Paradox** (~100 words): Open with: structured reasoning (CoT, ToT) improves accuracy in 86.3% of LLM-clinical task pairs but also introduces hallucination, omission, and reasoning drift (arXiv:2509.21933). Reasoning models improve accuracy ~3% but reduce consistency — 84% vs 91% (Springer, 2026). Multi-agent review was proposed as the answer. We show it introduces its own failure modes.

**Paragraph 2 — The Multi-Agent Promise and Its Limits** (~200 words): Process-level review reduces incorrect knowledge by 63% (Process vs. Outcome Review, 2025) — strong motivation for multi-agent designs. MAST (Cemri et al., NeurIPS 2025) provides the best existing failure taxonomy: 14 modes from 1,600+ traces across 7 frameworks. But MAST catalogs execution-level failures (did agents follow instructions?); it does not address reasoning-level failures (does following instructions produce correct outcomes?) or system-level emergent effects. Four of our seven failure modes fall outside MAST's scope.

**Paragraph 3 — This Paper** (~200 words): Seven failure modes from 30 sessions of systematic probing of a Decision-Review architecture for mental health classification on social media (ANGST dataset, 100 posts, 5 replicate runs per condition). Three architectural scopes: single-agent (FM-1, FM-4, FM-7), multi-agent interaction (FM-2, FM-3), system-level (FM-5, FM-6). Our contribution fills three compounding gaps: the coverage gap (4/7 FMs outside MAST), the theory-evidence gap (Goodhart, calibration, scaffolding interference — studied in isolation, here shown interacting in one system), and the domain gap (no failure catalog for clinical multi-agent AI). Each FM yields a design principle; the principles converge on externalized, structured clinical knowledge.

**Paragraph 4 — Paper Organization** (~100 words): Standard roadmap. Organizing principle: from failure observation to generalizable principle.

---

## §2. Theoretical Background (~1,400 words)

### 2.1 The Reliability Problem: Failure Taxonomies for Multi-Agent AI (~500 words)

Opens with the reliability problem: multi-agent LLM systems are deployed in high-stakes domains but systematic failure characterization lags behind deployment. MAST as the sustained anchor.

- **MAST** (Cemri et al., NeurIPS 2025): 14 failure modes, 3 categories (Specification/Design, Inter-Agent Misalignment, Task Verification/Termination), 1,600+ annotated traces, Grounded Theory methodology (κ=0.88). Show coverage: FM-4 ≈ FC1.1, partial FM-2/FM-7. Show gaps: FM-1 (no confidence variable), FM-3 (no architectural gating), FM-5 (trace-level not label-level), FM-6 (no knowledge management dimension).
- **AgentOps RCA** (arXiv:2508.02121): origin-level classification (system/model/orchestration-centric). Maps our FMs across origins.
- **Microsoft Taxonomy** (2025 whitepaper): industry validation that the problem space is active.
- **Gregor & Hevner** (MISQ 2013): our contribution extends MAST from Level 1 empirical catalog → Level 2 nascent design theory (design principles). **ISR 2024 editorial**: legitimizes failure characterization as DSR contribution in novel AI problem spaces.

**Gap**: Existing taxonomies catalog execution-level failures (did the agent follow instructions?) but not reasoning-level failures (does following instructions produce correct outcomes?) or system-level emergent effects (do architectural choices create failures invisible at the component level?). Four of our seven failure modes fall outside the scope of the best existing taxonomy.

### 2.2 Beyond Execution Failures: Metric Gaming, Calibration, and Reasoning Interference (~500 words)

Theoretical constructs that explain the mechanisms MAST does not address. Each tied to specific FMs — a toolkit for §5, not a survey.

**Metric gaming and process-outcome dissociation** (FM-2, FM-4):
- Formal Goodhart typology (ICLR 2024) — specifically *Causal Goodhart*: process compliance is downstream of quality, not causally upstream.
- Thomas & Uminsky (Patterns 2022) — IS-accessible metric reliance framework.
- Ojewale et al. (CHI 2025) — "compliance theater" in AI audit infrastructure.
- Process vs. Outcome Review (2025) — 63% reduction in incorrect knowledge. The theoretical promise; our paper reveals the boundary conditions.

**Confidence calibration failure** (FM-1, FM-3):
- Chhikara (TMLR 2025) — structured, non-random "confidence gap." RLHF-tuned models more miscalibrated on easier tasks.
- CriticBench — saturation of review (inverse of FM-3).

**Reasoning scaffold interference** (FM-7):
- Sprague et al. (2024) — CoT reduces accuracy up to 36.3% when deliberation is counterproductive.
- NeurIPS 2024 noisy CoT — external reasoning degrades native reasoning when introducing noise.
- Gemma 4 as native reasoning model (Google 2026). XAI explainability-performance tradeoff at prompt architecture level.

**Gap**: These constructs have been studied in isolation. No prior work examines how they interact within a single multi-agent system — where calibration failure feeds audit paradox, compounded by confidence gating, within a shared prompt space.

### 2.3 Mental Health AI at the Agent Frontier (~400 words)

The domain where these theoretical mechanisms are most consequential.

- Evolution: lexicon → fine-tuned transformers (MentalBERT, Sánchez Rodríguez 2026) → LLM prompting (Cognitive-Mental-LLM 2026) → agent era (nascent).
- Closest multi-agent: Mao et al. 2026 (CFD debate — enrichment, not classification), AgentMental 2025 (interview simulation, not post-level process auditing).
- Domain frameworks: TED (systematic probing — methodological analog, single-agent safety focus), PAMAS (information drowning — FM-6 reveals ceiling), MIND-SAFE (ethical safeguards, not reasoning audit).
- ANGST (Hengle et al. EMNLP 2024): benchmark + "Normal" = post-level temporal judgment, not population-level health claim.
- Why mental health amplifies the failures: (1) clinical vocabulary overlap → FM-5 cross-class interference; (2) Normal/clinical boundary ambiguity → FM-1 and FM-3 most damaging; (3) privacy mandates local models → FM-7 scaffolding tax exposed.

**Gap**: The domain has frameworks, benchmarks, and single-model baselines. It lacks the diagnostic vocabulary for what goes wrong when these are assembled into multi-agent architectures.

**Three gaps compose into the contribution**: coverage gap (2.1) + theory-evidence gap (2.2) + domain gap (2.3) = "We extend the best existing failure taxonomy by grounding previously theoretical constructs in the first empirical evidence from a clinical multi-agent system."

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

### 4.2 Ablation Design (500 words)

Brief context: The seven failure modes were first identified through iterative development across four prior experiment phases on Gemini Flash Lite (mechanical validation → nano-model structural analysis → live specification gaming discovery → full-scale taxonomy). This development history informed the ablation design described here; details are available in the supplementary materials.

The primary evaluation is a three-condition ablation isolating each architectural layer's contribution. All conditions share: Gemma 4 E4B backbone, temp=0.3, 100 posts (25/class, ANGST test split, seed=42), 5 replicate runs per condition.

**Table 1**: Ablation Conditions

| Condition | What it does | What it isolates |
|-----------|-------------|-----------------|
| Baseline | Flat single-pass prompt (no tool-use, no RA) — same clinical knowledge as DA, simpler format | Native model capability; FM-7 overhead measurement |
| C1: DA-only | DA with full tool-use scaffold (3-iteration evaluate_signal + Devil's Advocate + finalize_annotation). No RA, no Pass 2 | FM-1 (overconfidence), FM-7 (scaffold overhead). C1 vs Baseline = cost of tool-use architecture |
| C2: DA+RA, no pool | DA + RA in Pass 1. Pass 2 with RA feedback (max_rounds=2). Pool updated but not injected | FM-2 (process-outcome gap), FM-3 (feedback resistance). C2 vs C1 = contribution of multi-agent review |
| C3: DA+RA+pool | Same as C2, plus cumulative feedback pool injected as batch_context | FM-6 (pooling ceiling). C3 vs C2 = contribution of cross-batch knowledge accumulation |

**Why these conditions**: Each comparison isolates exactly one architectural layer:
- Baseline → C1: Does structured reasoning help or hurt? (FM-7)
- C1 → C2: Does adding a review agent improve classification? (FM-2, FM-3)
- C2 → C3: Does accumulated cross-batch knowledge help? (FM-6)
- FM-5 (cross-class collateral) is observable in per-class breakdowns across all conditions.
- FM-4 (specification gaming) was resolved prior to this ablation and is reported as an identified/closed failure mode.

**Pass 2 design**: Posts enter Pass 2 if the RA identifies them as wrong (`outcome_match=false`) or having reasoning gaps. In Pass 2, the DA receives RA feedback and gets two rounds (max_rounds=2): Round 0 with RA review, Round 1 with RA feedback but no final RA call. This design was determined empirically — single-round Pass 2 (max_rounds=1) produced zero corrections due to the DA's first-round feedback resistance.

**Temperature selection**: temp=0.0 produced fully deterministic output across all 5 replicates on both Gemini and Gemma (byte-identical runs), making the ablation scientifically meaningless. temp=0.3 was selected as the minimum producing measurable cross-run variance (SD=0.025 on macro F1) while maintaining output coherence (1% parse failure rate).

**Figure 3**: Ablation condition diagram showing the four conditions as nested layers: Baseline (innermost) → C1 adds tool-use → C2 adds RA → C3 adds pool.

---

## §5. Failure Mode Analysis (~3,200 words)

MAST (Cemri et al., NeurIPS 2025) provides the most comprehensive existing failure taxonomy: 14 modes across 3 categories, validated on 1,600+ traces. Our Table 2 extends MAST with seven failure modes from clinical multi-agent classification — four entirely outside MAST's scope, three overlapping but refined with domain-specific evidence.

**Table 2**: FM Summary (extends MAST)

| FM | Name | Scope | §2.2 Construct | MAST Relation | Design Principle |
|----|------|-------|----------------|---------------|-----------------|
| FM-1 | Overconfident Single-Pass Classification | Single-agent | Calibration gap (Chhikara 2025) | Outside MAST | External knowledge grounding |
| FM-4 | Specification Gaming | Single-agent | Extremal Goodhart (ICLR 2024) | Maps to FC1.1 | Mandatory steps immune to optional exits |
| FM-7 | Architectural Complexity Overhead | Single-agent | Scaffolding interference (Sprague 2024) | Extends FC2.6 | Externalize reasoning into knowledge stores |
| FM-2 | Process-Outcome Audit Paradox | Multi-agent | Causal Goodhart (ICLR 2024) | Extends FC3.3 | Discriminative criteria in process rules |
| FM-3 | Confidence-Feedback Blind Spot | Multi-agent | Calibration gap (Chhikara 2025) | Outside MAST | Decouple confidence from correctness |
| FM-5 | Cross-Class Collateral Damage | System-level | Domain-specific emergent | Outside MAST | Class-specific reasoning paths |
| FM-6 | Generic Knowledge Pooling Ceiling | System-level | Domain-specific emergent | Outside MAST | Per-instance retrieval |

### 5.1 Where Self-Correction Fails (~1,000 words)

**FM-1: Overconfident Single-Pass Classification** (~450 words)
- What: 95–99% of posts terminate via CONFIDENCE_THRESHOLD despite accuracy 0.34–0.36. DA performs the form of deliberation without the substance.
- §2.2 construct: Instance of the confidence calibration gap (Chhikara, TMLR 2025) — structured, non-random miscalibration operating within a clinical multi-agent pipeline. Outside MAST scope (no confidence variable in MAST's 14 modes).
- Evidence: EXP-008 C1 ablation (5 runs, Gemma): mean F1=0.368, SD=0.025. 0 posts entered Pass 2. U-shape rate 2–5% (confidence guard prevents original pattern). Cross-backbone confirmed (F-043, F-050, F-057).
- Design Principle (DP-1): Single-agent self-correction is insufficient. External knowledge grounding required.

**FM-7: Architectural Complexity Overhead** (~400 words)
- What: Tool-use scaffold imposes overhead that can interfere with native reasoning. Baseline F1=0.643 vs DA F1=0.368 on Gemma (27.5pp gap); negligible on Gemini (-0.006).
- §2.2 construct: Instance of scaffolding interference (Sprague et al. 2024, NeurIPS 2024 noisy CoT). Extends MAST FC2.6 (reasoning-action mismatch) by identifying a deeper mechanism: the scaffold degrades reasoning quality below the unscaffolded baseline.
- Evidence: Cross-backbone interaction effect (F-056, F-057, F-044). RA v9→v10 redesign: 97%→0% parse failure (F-055). Gemma 4 is a native reasoning model with `<|think|>` mode (Google 2026). Sprague et al. 2024: CoT reduces accuracy up to 36.3% when deliberation is counterproductive.
- XAI tradeoff: Scaffold provides interpretability at the cost of performance — same tradeoff at prompt architecture level. Open question: how to design scaffolds that complement rather than compete with native reasoning.
- Design Principle (DP-7): Externalize reasoning complexity into knowledge stores. Separate reasoning from memory.

**FM-4: Specification Gaming** (~300 words, Identified/Closed)
- What: DA exploited conflicting prompt rules to skip mandatory iterations (10/10 posts).
- §2.2 construct: Extremal Goodhart (ICLR 2024). Maps to MAST FC1.1 (disobey task specification) — confirms MAST in clinical domain.
- Evidence: EXP-003. Resolved with DA v5.2 (D22), validated in EXP-003c.
- Design Principle: Mandatory steps must be immune to co-present optional exits.

### 5.2 Where Review Fails to Correct (~900 words)

**FM-2: Process-Outcome Audit Paradox** (~500 words)
- What: process_valid_rate=0.980 while Normal F1=0.300. RA optimizes for compliance over correctness.
- §2.2 construct: Causal Goodhart (ICLR 2024) — process compliance is downstream of quality, not causally upstream. Extends MAST FC3.3 (incorrect verification): our verifier works correctly but verifies the wrong thing.
- Evidence: EXP-008 C2 ablation: 66 posts enter Pass 2, 5 corrections (7.6%), 0 regressions. RA correctly diagnosed all 21 misclassified Normal posts (F-026) but feedback had minimal corrective effect. Class-asymmetric: Comorbid correction 45% vs Normal 7% (LL-036).
- Design Principle (DP-2): Process rules must encode discriminative criteria, not generic compliance checks.

**FM-3: Confidence-Feedback Blind Spot** (~400 words)
- What: High-confidence wrong predictions structurally bypass correction.
- §2.2 construct: Same calibration gap as FM-1 (Chhikara 2025), but at the multi-agent level. FM-1 = calibration prevents self-correction; FM-3 = calibration creates a structural blind spot in the review loop. Outside MAST scope (no conditional review activation in MAST's 14 modes).
- Evidence: Structural blind spot addressed by D32 bypass_confidence_guard (Pass 2 activation 63%). Residual: DA receives feedback but maintains prediction in 92.4% of Pass 2 posts. max_rounds=1 → 0% correction; max_rounds=2 → 7.6% correction (round-dependent resistance, LL-037).
- Design Principle (DP-3): Feedback activation must decouple confidence from correctness.

### 5.3 Where Architecture Fails to Scale (~800 words)

**FM-5: Cross-Class Collateral Damage** (~350 words)
- What: Depression/Normal boundary check improved Normal +0.086 but regressed Comorbid -0.061.
- Outside MAST scope (trace-level analysis cannot detect label-level cross-class coupling). Domain-specific emergent effect from clinical vocabulary overlap (§2.3).
- Evidence: EXP-004 S22 (F-042, LL-032). C1 ablation per-class variance: Normal consistently lowest (mean 0.187), Comorbid depressed relative to other clinical classes.
- Design Principle (DP-5): In shared-reasoning systems, class-specific corrections propagate to adjacent classes. Class-specific reasoning paths required.

**FM-6: Generic Knowledge Pooling Ceiling** (~450 words)
- What: Pool improved Anxiety ΔF1=+0.143, Comorbid ΔF1=+0.129, Normal ΔF1=0.000.
- Outside MAST scope (no knowledge management dimension; MAST's modes are observable within single traces, FM-6 requires cross-batch observation). Domain-specific: Normal/clinical boundary ambiguity (§2.3) is where pooling fails.
- Evidence: EXP-004 D37 (F-040). C3 probe: no accumulation effect (first half 22.2% ≈ second half 21.4%, F-054). Zero-shot ceiling confirmed after 6 DA + 4 RA versions (F-041).
- Design Principle (DP-6): Generic pooling has a resolution limit below which per-instance retrieval is required.

### 5.4 Ablation Summary and Design Principles (~500 words)

The ablation results map onto the three gaps from §2: Baseline→C1 reveals FM-1 and FM-7 (coverage gap — outside MAST). C1→C2 provides evidence for Causal Goodhart and calibration gap operating in one system (theory-evidence gap). FM-5 and FM-6 are observable in per-class breakdowns across all conditions (domain gap — emergent effects of mental health boundary ambiguity).

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
- Cross-backbone validation extends MAST's generalizability: MAST validated on general-purpose frameworks with frontier models; we validate on local clinical models under privacy constraints. FM-1 confirmed backbone-independent. FM-7 shows capability-dependent magnitude (a dimension MAST does not capture). Qwen3.5-2B rejection confirms capability floor.
- Generalizability beyond mental health: each FM generalizes to any multi-agent clinical AI.
- Reasoning–memory separation: FM-7/DP-7 connects to the emerging paradigm. Clinical knowledge should not live in prompts; it should live in external structured stores.

### 6.2 Practitioner Implications (~500 words)
- Execution-level taxonomies like MAST do not capture the operational realities below. These are complementary practitioner-facing findings.
- Reliability: parse failure stochasticity, per-post error isolation, silent output truncation.
- Prompt complexity cliff: hard threshold per model tier (RA v9→v10).
- Feedback loops: round resistance (LL-037), class-asymmetric correction (LL-036), confidence guard design tension (F-051).
- Architecture portability: migration required full prompt redesign, parser hardening, parameter adjustment.
- Temperature as experimental precondition: temp=0.0 → deterministic; temp=0.3 → valid variance.

### 6.3 Convergent Design Requirements for External Clinical Knowledge (~250 words)
Seven DPs converge on: externalized, structured clinical knowledge, retrievable per-instance, auditable independently — addressing both the MAST coverage gap (§2.1) and the theoretical construct gaps (§2.2). Specific artifact form is an open research question. Directions: clinical ontology structuring for retrieval, class-specific subgraph traversal, scaffold-free knowledge injection. Our FMs provide concrete evaluation criteria for any proposed architecture.

---

## §7. Conclusion (~400 words)

**Paragraph 1 — Contributions**: Extends MAST (Cemri et al., NeurIPS 2025) into clinical mental health classification. Seven failure modes, three levels, 30 sessions. Three gaps addressed: coverage (4/7 outside MAST), theory-evidence (Goodhart, calibration, scaffolding interference interacting in one system), domain (first clinical multi-agent failure catalog). Each yields a design principle converging on externalized clinical knowledge. "Not safer by virtue of more agents — safer when each interaction point is designed with awareness of how it can fail."

**Paragraph 2 — Limitations**: Pilot study. 100 posts, single dataset, 4B local model (privacy-motivated). Cross-backbone validation on two additional models. Oracle-aided Pass 2 gate. Zero-shot configuration. Boundaries define where next investigation should push. FMs and DPs offered as testable hypotheses.

---

## Figures and Tables

| Item | Description | Section |
|------|-------------|---------|
| Figure 1 | Three-level FM taxonomy pyramid (FM-1/4/7 → FM-2/3 → FM-5/6) | §3 (leads section) |
| Figure 2 | Architecture diagram: DA → RA → Pass 2 → curation pool | §3 |
| Figure 3 | Ablation condition diagram: nested layers (Baseline → C1 → C2 → C3) | §4 |
| Figure 4 | Confusion matrix (EXP-008 Gemma, best run) | §5 |
| Figure 5 | Ablation box plot: F1 across C1/C2/C3 conditions | §5.4 |
| Table 1 | Ablation conditions (what, how, why, FM isolated) | §4 |
| Table 2 | FM summary (name, scope, construct, DP) | §5 opener |
| Table 3 | EXP-008 ablation results | §5.4 |
| Table 4 | Seven DPs and convergent direction | §5.4 |

---

## Word Count

| Section | Words |
|---------|-------|
| §1 Introduction | 700 |
| §2 Theoretical Background | 1,400 |
| §2.1 The Reliability Problem: Failure Taxonomies for Multi-Agent AI | 500 |
| §2.2 Beyond Execution Failures: Metric Gaming, Calibration, Reasoning Interference | 500 |
| §2.3 Mental Health AI at the Agent Frontier | 400 |
| §3 Artifact Design | 600 |
| §4 Systematic Probing Methodology | 700 |
| §5 Failure Mode Analysis | 3,200 |
| §6 Discussion | 1,250 |
| §7 Conclusion | 400 |
| **Total** | **~8,250** |

*Trim ~250 words during drafting to fit 8,000 target.*
