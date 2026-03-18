# Project Status — Multi-Agent Mental Health Detection on Social Media

**Principal Investigator**: Jie Tao, D.Sc., Department of Analytics, Fairfield Dolan School of Business, Fairfield University
**Date**: 2026-03-18
**Phase**: Phase 1 — Decision-Review Agent Loop (in progress)
**Target Venues**: HICSS, Information Systems Research, Information Systems Frontiers
**Funding**: Robert E. Wall Faculty Award (2025–2026)

**Summary**: This study introduces a two-agent LLM architecture for mental health classification on social media. A Decision Agent classifies Reddit posts using structured clinical reasoning, while a Review Agent audits both the classification outcome and the reasoning process, delivering Socratic feedback that drives iterative improvement. A two-pass batch pipeline screens all posts cheaply, then curates the most clinically important cases for deeper multi-round analysis. Evaluated on the ANGST dataset (2,876 gold-labeled Reddit posts, 4-class), the system is designed to demonstrate that structured inter-agent feedback with process compliance enforcement improves classification accuracy and consistency over single-agent baselines — without fine-tuning.

---

## 1. Research Question & Contributions

### Research Question

Can a multi-agent Decision-Review feedback loop with structured clinical reasoning and process compliance auditing improve LLM-based mental health classification on social media, compared to single-agent in-context learning?

### Claimed Contributions

1. **First Decision-Review feedback loop for direct mental health classification.** Prior multi-agent work in mental health (Mao et al., 2026) targets data enrichment, not direct classification with iterative feedback. This work implements a feedback loop where a Review Agent audits both the outcome and reasoning process, and Socratic feedback drives iterative improvement of the Decision Agent's classifications.

2. **Four-rule process compliance auditing during classification.** Existing explainability methods (LIME, attention visualization) are applied post hoc. This work introduces real-time process auditing — iteration count, Devil's Advocate execution, prior evaluation consideration, and feedback integration — enforced during classification by a separate Review Agent.

3. **Socratic feedback adapted to clinical signal analysis.** Socratic questioning has been validated for math reasoning (SSR, 2025) and education (Holub et al., 2026) but not clinical text classification. This work adapts Socratic feedback to clinical signal analysis: feedback is blinded to ground truth labels and expressed in clinical terms (hopelessness, hypervigilance, catastrophizing, social withdrawal) to guide the Decision Agent toward overlooked signals.

4. **Adaptive-depth two-pass batch pipeline.** All existing mental health classification systems apply uniform computational depth. This work introduces severity-weighted curation (recency + F2-scored impact) that screens all posts cheaply, then allocates deeper multi-round analysis exclusively to clinically important or difficult cases.

---

## 2. Methodology

### Two-Agent Architecture

The system pairs two LLM agents, both powered by Gemini Flash Lite in a zero-shot in-context learning configuration (no fine-tuning):

- **Decision Agent** performs structured clinical reasoning through a mandatory 3-iteration evaluation cycle. On each iteration, it analyzes the post for clinical signals (linguistic markers, emotional indicators, behavioral patterns), assigns a label and confidence score, and provides reasoning. The second iteration requires a mandatory Devil's Advocate analysis — the agent must argue against its own initial classification and consider alternative interpretations. After three iterations, the agent produces a final annotation with a consolidated label and confidence score.

- **Review Agent** conducts a four-step audit of the Decision Agent's work: (1) parse the full reasoning trajectory, (2) audit the classification outcome against ground truth, (3) audit the reasoning process against four compliance rules, and (4) issue a final verdict (APPROVED or REJECTED) with structured feedback. The four process rules are: completion of all required iterations, execution of Devil's Advocate analysis, consideration of prior evaluations when shifting labels, and integration of any prior feedback.

### Feedback Protocol

When the Review Agent rejects a classification, it produces two categories of feedback:

- **Signal gap**: Identifies clinical signals the Decision Agent overlooked or underweighted. Expressed in clinical terms (e.g., "hopelessness language in sentences 2 and 5 suggests depressive ideation") rather than revealing the ground truth label. The Decision Agent is blinded to ground truth throughout.
- **Process error** (when applicable): Identifies specific protocol violations (e.g., "Devil's Advocate analysis was superficial — alternative label was dismissed without engaging counter-evidence").

Feedback is serialized and injected into the Decision Agent's next iteration, where it must explicitly address each feedback point.

### Two-Pass Batch Pipeline

The system processes posts in batches through a screening–curation–deep analysis pipeline:

1. **Pass 1 (Screening)**: Every post in the batch receives exactly one Decision→Review cycle. This produces initial classifications, reasoning trajectories, and review results for all posts.

2. **Curation**: Using Pass 1 results, posts are ranked by a composite score combining recency (min-max normalized timestamps) and severity (classification difficulty and clinical impact, F2-weighted to penalize missed clinical cases more heavily than false alarms). The top-K posts (default K=3) are selected for deeper analysis.

3. **Pass 2 (Deep Analysis)**: Selected posts that were not already approved in Pass 1 enter the full feedback loop for up to 3 additional Decision→Review rounds. Posts approved in Pass 1 are skipped.

This design resolves the chicken-and-egg problem — severity scoring requires classification results, which the screening pass provides — while concentrating computational resources on the cases that need them most.

### Classification Schema

Four classes calibrated to the ANGST dataset: **Depression**, **Anxiety**, **Comorbid**, and **Normal**. The Comorbid label requires both Depression and Anxiety confidence scores independently ≥ 0.8; otherwise, the stronger single-condition label is assigned.

### Approach

Zero-shot in-context learning with structured prompts — no fine-tuning, no few-shot demonstrations. Both agents share identical clinical definitions and a target confidence threshold of 0.8. Prompts employ simulated tool use: the LLM generates structured JSON describing a sequence of analytical steps, which the system parses programmatically.

---

## 3. Experimental Design

### Dataset

**ANGST** (`ameyhengle/ANGST`): 2,876 gold-labeled Reddit posts with 4-class annotations (Depression, Anxiety, Comorbid, Normal). The label space maps directly to the system's classification schema. Approximately 10% (~290 posts) reserved for evaluation; remainder reserved for additional experiments.

### Baselines

| # | Baseline | Description |
|---|----------|-------------|
| 1 | Single agent, no feedback | Decision Agent classifications from Pass 1 across all posts — produced by the same pipeline, no separate experiment needed |
| 2 | Keyword/lexicon-based | Non-LLM detection using clinical lexicons (e.g., KETCH) |
| 3 | Published benchmarks | MentalBERT (Sánchez Rodríguez et al., 2026), KETCH results where applicable |

### Metrics

- **Per-class**: Precision, Recall, F1, F2 for each of Depression, Anxiety, Comorbid, Normal
- **Aggregate**: Macro-average and weighted-average across classes
- **AUC-ROC**: Binary collapse — clinical (Depression | Anxiety | Comorbid) vs. Normal — using final confidence as decision score
- **Process compliance**: `process_valid` rate (fraction of reviews with no protocol violations), first-pass APPROVED rate
- **Improvement trajectory**: Per-round deltas in outcome match and confidence for posts receiving multiple feedback rounds

### Success Criterion

Statistically significant improvement of the Decision-Review loop (Pass 2 results) over Baseline 1 (Pass 1 results) on the ANGST dataset. Cross-lingual results (if pursued) are exploratory and not required for Phase 1 success.

---

## 4. Key Design Decisions

| Decision | Value | Rationale |
|----------|-------|-----------|
| LLM backbone | Gemini Flash Lite (`gemini-3.1-flash-lite-preview`) for both agents | Budget constraint; supports structured JSON output required by prompts |
| Max feedback rounds | 3 (production) / 1 (prototype) | Caps compute cost; diminishing returns expected beyond 3 |
| Feedback format | Structured JSON between agents; serialized `[SIGNAL GAP]` + `[PROCESS ERROR]` for Decision Agent injection | Parseable programmatically; enforces consistent schema |
| Context window strategy | Curation — rank posts, keep Top-3 per batch | Fits within context window; forces signal prioritization |
| Post ranking | Recency + severity, equal weights (0.5/0.5) | Balances temporal relevance with clinical salience; weights are tunable |
| Severity scoring | F2-weighted impact (FN = 2× FP cost) + repeatedness | Clinical NLP default; recall-favoring without being extreme |
| Recency normalization | Min-max across the batch | Oldest→0, newest→1; simplest prototype default |
| Dataset | ANGST (4-class, Reddit) | Label space maps directly to prompt schema; 2,876 gold-labeled posts |
| Learning approach | Zero-shot ICL, no fine-tuning | Simplifies initial implementation; baseline before adding demonstrations |
| Processing unit | One post at a time; curation at batch level | Each post runs an independent Decision→Review loop |
| Early stopping | Allowed when `final_confidence ≥ 0.8` | Avoids unnecessary rounds for high-confidence classifications |

---

## 5. Literature Positioning

### Three Research Dimensions

This work sits at the intersection of:

1. **LLM-based mental health detection on social media** — where single-model approaches (MentalBERT, GPT-4 prompting) dominate, and agent-based systems are identified as the "next frontier" but remain underexplored (Ge et al., 2025; MDPI Systematic Review, 2026).

2. **Multi-agent LLM systems and iterative feedback** — where debate, reflexion, and Socratic questioning have been validated for general reasoning (math, QA, code) but remain nascent in clinical classification. The closest work (Mao et al., 2026, CFD) applies multi-agent debate to mental health data enrichment, not direct classification.

3. **Clinical reasoning and process compliance** — where a "reasoning paradox" has emerged: structured reasoning (Chain-of-Thought) improves accuracy but degrades consistency in 86.3% of LLM-clinical task pairs ("Why CoT Fails," 2025). This motivates the Review Agent's process auditing role.

### Key Insight

The reasoning paradox — CoT helps accuracy but hurts consistency — is precisely what the Decision-Review architecture addresses. By combining a Decision Agent that follows a structured clinical reasoning protocol with a Review Agent that audits both outcomes and process compliance, the system targets accuracy *and* consistency simultaneously.

### Closest Competitor

Mao et al. (2026) Confidence-Aware Fine-Grained Debate (CFD): multi-agent debate on 350 Reddit posts for mental health data enrichment. Differs from this work in three key ways: (1) targets label generation for unlabeled data, not direct classification with iterative feedback; (2) lacks structured clinical reasoning protocols; (3) does not audit the reasoning process.

> Full literature analysis and citation groups: see `literature_positioning.md`

---

## 6. Current Status & Next Steps

### Completed

- **Research design**: Two-agent architecture, feedback protocol, two-pass pipeline, classification schema — all decisions finalized (D1–D16)
- **Prompt engineering**: Decision Agent prompt v5.1 (3-iteration evaluation with mandatory Devil's Advocate) and Review Agent prompt v6 (4-tool audit sequence with Socratic feedback)
- **System architecture**: Complete prototype design with component specifications, data model, pipeline flow, configuration management, and testing strategy
- **Literature positioning**: Gap analysis identifying four unique contributions against Dec 2025 – Mar 2026 literature; positioning statement drafted

### Next Steps

- **Prototype implementation**: Build orchestrator, API client, prompt renderer, response parser, curation module, evaluation module, and logger
- **Smoke testing**: Single post through the full Decision→Review pipeline (validates API round-trip, prompt rendering, response parsing)
- **Validation testing**: One full batch (10 posts) through the two-pass pipeline (validates curation, Pass 2, evaluation, logging)
- **Full evaluation**: Run against ANGST evaluation set; compute all metrics; compare against baselines

### Open Questions

- **Feedback actionability metric**: Dimensions identified (process similarity to clinical standards; performance impact across rounds) but operationalization is pending — scoring rubric vs. embedding similarity vs. checklist-based approach still to be determined
- **Severity score extensibility**: Architecture supports additional metrics under both severity and recency dimensions; exact formula for additional recency signals to be determined when needed

---

## 7. Scope Boundaries

### In Scope (Phase 1)

- Decision-Review agent loop design, implementation, and evaluation
- Prompt engineering for both agents (ICL with clinical lexicons)
- Two-pass batch pipeline with severity-weighted curation
- Evaluation on ANGST dataset with three baselines
- Ablation studies isolating the Review Agent's contribution
- Publication preparation for HICSS / ISR / ISF

### Out of Scope

- **Phase 2 — Dynamic Knowledge Retrieval**: Planned extension to enrich Decision Agent context with clinical guidelines and psychiatric literature in near-real-time. Phase 1 results serve as baselines.
- **Fine-tuning**: All experiments use zero-shot ICL
- **Cross-lingual analysis**: Sina Weibo / Chinese data is exploratory only, not required for Phase 1 success
- **Clinical deployment**: System outputs are decision-support tools, not clinical diagnoses. All data is publicly available and de-identified; IRB consultation planned for exemption determination.

---

## 8. Ethical Considerations

- All datasets are publicly available, de-identified, and used for secondary analysis only
- IRB consultation planned for exemption determination
- Compliance with GDPR and HIPAA where applicable
- System outputs are framed as decision-support tools, not clinical diagnoses — this framing must be maintained in all publications

---

## 9. Document Map

| Document | Role |
|----------|------|
| `project_status.md` | **This document** — current research design snapshot |
| `project_brief.md` | Original proposal-level specification (goals, datasets, tasks, ethics) |
| `research_state.md` | Chronological decision log (D1–D16), open questions, session history |
| `prototype_design.md` | System architecture and implementation specifications (code-focused) |
| `literature_positioning.md` | Literature landscape, gap analysis, citation groups, related work outline |
| `decision_agent_prompt_v5.1.md` | Decision Agent system prompt (active, v5.1) |
| `review_prompt_v6.md` | Review Agent system prompt (active, v6) |
