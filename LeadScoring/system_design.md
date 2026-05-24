# Lead Scoring & Ranking System — System Design

## 1. System Thesis

The system was designed around a core operational constraint:

> Sales teams cannot contact every lead.

This transformed the problem from:
- binary classification
- probability estimation

into:
- constrained ranking optimization
- resource allocation under extreme class imbalance

The objective became:

> maximize conversion capture within a constrained Top-K operational budget.

Rather than optimizing for:
- global classification accuracy
- thresholded predictions
- generic probability estimation

the system optimized for:
- ranking concentration
- lead prioritization quality
- operational outreach efficiency

under realistic business constraints.

---

# 2. Problem Reformulation

## Traditional Framing

Conventional lead scoring systems typically attempt to answer:

> “Will this user convert?”

However, this framing becomes problematic under:
- extreme class imbalance
- unstable threshold selection
- limited outreach bandwidth
- sparse behavioral signals

In this environment:
- most leads never convert
- sales teams can only contact a limited subset of users
- threshold-based classification creates unstable operational behavior

---

## Ranking-Based Reformulation

The system instead reframed the task as:

> “Which users should be prioritized first?”

This allowed optimization around:
- Precision@K
- Recall@K
- capture concentration
- operational prioritization quality

rather than:
- global classification accuracy
- ROC-AUC optimization
- threshold tuning

This architectural shift became one of the most important design decisions in the system.

---

# 3. Data Modeling Strategy

## Conversion Definition

The positive label represented:
- successful full-payment conversion

The system focused on identifying:
- high-intent users
- early conversion signals
- pre-sales engagement quality

rather than long-term lifecycle prediction.

---

## Prediction Window

Behavioral signals were generated using:
- user activity occurring within approximately 30 minutes after lead creation
- strictly pre-contact interaction windows

This design simulated:
- realistic deployment-time signal availability
- operational scoring constraints
- sales-ready prioritization timing

The 30-minute aggregation window emerged as the strongest predictive period during experimentation.

---

## Leakage Prevention

Leakage prevention became a critical system constraint.

Behavioral systems are highly vulnerable to:
- future interaction contamination
- post-sales-contact events
- delayed funnel activity leakage

To avoid this:
- feature generation was strictly bounded before first sales interaction
- only deployment-available events were included
- aggregation windows enforced temporal cutoff boundaries

This ensured:
- forward-safe feature generation
- realistic offline evaluation
- operationally valid ranking behavior

---

# 4. Identity & Session Aggregation

User behavior frequently spanned:
- multiple sessions
- repeated visits
- fragmented engagement windows

Session aggregation was performed using:
- `user_id`-based identity tracking
- interaction count aggregation
- temporal grouping windows
- pre-contact behavioral merging

The aggregation strategy prioritized:
- preserving temporal intent patterns
- reducing fragmented behavioral representation
- improving signal continuity before ranking

---

# 5. Feature Engineering System

## Design Philosophy

Feature engineering prioritized:
- explainability
- temporal relevance
- sparse-signal robustness
- operational interpretability
- deployment-oriented transparency

The system intentionally avoided:
- opaque representations
- difficult-to-debug feature transformations
- overly complex embedding pipelines

during initial experimentation phases.

---

# 6. Feature Categories

## Behavioral Signals

Examples:
- page interactions
- clicks
- session activity
- engagement density
- repeat interactions
- form progression

These features captured:
- user intent intensity
- behavioral momentum
- interaction consistency

---

## Funnel Signals

Examples:
- OTP completion
- partial funnel progression
- drop-offs
- repeated attempts
- engagement completion patterns

These became some of the strongest indicators of downstream conversion intent.

---

## Traffic & Acquisition Signals

Examples:
- referral source
- campaign source
- paid vs direct traffic
- acquisition channel quality

These features helped contextualize:
- user quality
- acquisition intent
- campaign-driven behavioral variance

---

# 7. Temporal Aggregation System

Behavioral signals were aggregated across multiple temporal windows:

| Window | Purpose |
|--------|---------|
| 5 min | immediate behavioral intent |
| 10 min | short-term engagement stabilization |
| 15 min | interaction consistency |
| 30 min | strongest predictive aggregation window |

Key observation:
- the majority of meaningful behavioral intent emerged within the first ~30 minutes after lead creation.

This insight significantly influenced:
- feature design
- scoring cadence
- operational ranking assumptions

---

# 8. Hybrid Activity Mapping System

A hybrid behavioral grouping pipeline was introduced to reduce sparse event fragmentation.

The system combined:
- KNN-based clustering
- semantic feature grouping
- manual behavioral abstraction

Implementation approach:
- SentencePiece-style encoding representations were used for behavioral feature grouping
- KNN clustering generated approximately 11–12 activity clusters
- original sparse activities were mapped into generalized semantic groupings

The final system used:
- hybrid automated clustering
- manually validated semantic mappings

This improved:
- behavioral generalization
- sparse-event robustness
- feature consistency across users

while preserving:
- interpretability
- debugging visibility
- operational reasoning capability

---

# 9. Encoding Strategy

One-hot encoding was intentionally selected despite increased dimensionality.

Reasons:
- strong interpretability
- SHAP compatibility
- feature-level debugging visibility
- operational transparency

This decision traded:
- compact representation efficiency

for:
- explainability
- operational trust
- easier failure analysis

The system prioritized:
- understandable ranking behavior
over:
- maximally compressed feature representations

---

# 10. Explainability System

Operational explainability was treated as a first-class design requirement.

The explainability layer included:
- SHAP analysis
- feature importance ranking
- behavioral contribution inspection
- Top-K reasoning validation

This was necessary for:
- debugging ranking quality
- validating behavioral assumptions
- operational trust
- stakeholder interpretability

[Add examples of strongest behavioral signals discovered during analysis.]

---

# 11. Modeling Strategy

## Why Standard Classification Failed

Initial classification-oriented approaches suffered from:
- unstable thresholds
- weak operational alignment
- imbalance sensitivity
- inconsistent prioritization behavior

Even models with acceptable:
- ROC-AUC
- classification accuracy

often produced:
- weak Top-K concentration
- unstable operational rankings

This revealed that:
- ranking quality mattered substantially more than threshold accuracy.

---

# 12. Model Benchmarking

## Model Strategy Decision

| Model Type | Why It Was Considered | Why It Was Rejected / Limited | Final Outcome |
|------------|----------------------|-------------------------------|----------------|
| Logistic Regression | Strong interpretability, stable baseline, easy probability inspection, operational transparency | Linear decision boundary struggled to capture complex behavioral interactions and temporal engagement patterns | Useful baseline and debugging model, but insufficient ranking quality for final deployment-oriented system |
| XGBoost | Strong non-linear modeling capability and high predictive power on sparse behavioral data | Showed instability under severe class imbalance, inconsistent Top-K behavior across varying K ranges, and higher sensitivity to overfitting/drift | Strong offline performance but operationally less stable |
| Ensemble Models (Voting/Stacking) | Attempted to reduce variance and improve robustness through model combination | Added significant complexity, reduced interpretability, and produced only marginal ranking improvements | Operational cost/complexity outweighed gains |
| Balanced Random Forest | Naturally suited for imbalance handling through balanced sampling, stable ranking behavior, explainability compatibility | Slightly lower theoretical complexity/performance ceiling compared to advanced boosting methods | Selected due to stable Top-K performance, operational simplicity, explainability, and robustness across multiple ranking thresholds |

---

# 13. Balanced Random Forest Selection

Balanced Random Forest (BRF) was ultimately selected because it consistently demonstrated:

- stable ranking behavior
- reliable Top-K concentration
- robustness under severe imbalance
- lower operational complexity
- explainability compatibility

A key observation was:

> BRF maintained relatively stable performance across multiple K thresholds.

This mattered more operationally than:
- maximizing benchmark metrics
- achieving marginal offline score improvements

because the real deployment objective depended on:
- stable prioritization quality
- consistent operational behavior

rather than:
- theoretical modeling complexity.

---

# 14. Why Learning-to-Rank Was Not Used

Learning-to-Rank (LTR) architectures were intentionally avoided during initial system development.

Reasons included:
- pairwise ranking computational cost
- operational complexity
- reduced explainability
- deployment simplicity constraints
- infrastructure overhead

The project prioritized:
- reproducibility
- interpretability
- stable operational integration
- maintainable ranking logic

rather than:
- maximizing theoretical ranking performance.

This aligned better with:
- deployment-readiness goals
- operational debugging requirements
- stakeholder trust considerations

---

# 15. Ranking System Design

## Ranking Objective

The final objective was:

> maximize conversion capture within constrained operational outreach capacity.

The system optimized for:
- ranking concentration
- high-intent prioritization
- constrained resource allocation

rather than:
- global prediction accuracy.

---

## Top-K Selection Strategy

Top-K selection was determined using:
- Precision@K
- Recall@K
- capture concentration analysis

The selected K balanced:
- conversion density
- operational sales bandwidth
- outreach efficiency
- ranking precision stability

This ensured:
- prioritization quality remained operationally useful
- sales effort remained concentrated on high-intent users

rather than maximizing:
- total outreach volume.

---

# 16. Evaluation Framework

## Primary Metrics

| Metric | Purpose |
|--------|---------|
| Precision@K | Measures converter density inside ranked leads |
| Recall@K | Measures coverage of total converters captured |
| Capture Rate@K | Measures operational ranking effectiveness |

---

## Why ROC-AUC Was Insufficient

ROC-AUC failed to capture:
- constrained execution behavior
- operational ranking concentration
- Top-K prioritization quality

This became one of the major reasons the system transitioned toward:
- ranking-first evaluation

instead of:
- threshold classification evaluation.

---

## Ranking Evaluation Strategy

The system evaluated:
- Top-K concentration
- ranking stability
- conversion density
- prioritization consistency

across:
- multiple K ranges
- varying operational capacities
- imbalance-sensitive evaluation settings

[Add:
- decile lift analysis
- cumulative gain evaluation
- ranking curve analysis if available.]

---

# 17. Operational System Design

## Intended Deployment Model

The system was designed as:
- a deployment-oriented ranking framework
- operational lead prioritization infrastructure
- CRM-ready scoring architecture

The project primarily validated:
- architectural feasibility
- ranking quality
- operational alignment
- deployment-readiness assumptions

rather than full production rollout.

---

## Batch vs Real-Time Tradeoff

The system intentionally adopted:
- batch-oriented ranking workflows

instead of:
- real-time streaming inference

Reasons included:
- operational simplicity
- lower infrastructure complexity
- sales workflow alignment
- easier reproducibility
- simplified debugging and ranking validation

The architecture prioritized:
- reliability
- maintainability
- deployment simplicity

during initial system design phases.

---

# 18. Key Tradeoffs

| Tradeoff | Decision | Reason |
|----------|----------|--------|
| Accuracy vs Explainability | Explainability prioritized | Sales and operational workflows required interpretable lead prioritization and debugging visibility |
| Complexity vs Stability | Stability prioritized | Stable ranking concentration across Top-K ranges was more valuable than maximizing benchmark metrics |
| Boosted Models vs BRF | BRF selected | BRF demonstrated more stable ranking behavior and lower operational complexity under extreme imbalance |
| Real-time vs Batch | Batch selected | Sales workflow operated in scheduled prioritization cycles, making streaming infrastructure unnecessary initially |
| Ranking vs Classification | Ranking prioritized | Operational objective depended on constrained outreach prioritization rather than global classification accuracy |
| Dense Encoding vs One-Hot Encoding | One-hot encoding selected | Preserved interpretability, SHAP compatibility, and feature-level debugging capability |
| Learning-to-Rank vs Pointwise Ranking | Pointwise ranking approach selected | LTR introduced higher computational complexity and reduced explainability with limited operational benefit initially |
| Model Complexity vs Operational Usability | Simpler architecture preferred | Easier debugging, reproducibility, maintenance, and deployment integration |

---

# 19. Failure Cases & Lessons

## Threshold Instability

Threshold-based systems produced:
- inconsistent prioritization behavior
- unstable operational outputs
- weak Top-K concentration

This became one of the major drivers behind:
- ranking-first reformulation.

---

## Complexity Without Operational Gain

Complex ensemble systems:
- increased engineering complexity
- reduced interpretability
- complicated debugging workflows

while providing:
- limited practical ranking improvement.

Key insight:
> operational stability mattered more than benchmark maximization.

---

## Leakage Sensitivity

Behavioral systems proved highly sensitive to:
- future-event contamination
- post-contact leakage
- temporal misalignment

This reinforced the importance of:
- strict cutoff enforcement
- deployment-realistic feature generation
- forward-safe aggregation logic

---

## Explainability as a System Constraint

Interpretability evolved from:
- a “nice-to-have”

into:
- a core operational requirement.

This influenced:
- feature encoding
- model selection
- ranking design
- evaluation methodology

Explainability became necessary for:
- debugging
- operational trust
- ranking validation
- stakeholder reasoning

---

# 20. Future Improvements

Potential future redesign directions include:

- Learning-to-Rank architectures
- online ranking adaptation
- streaming aggregation systems
- feature store integration
- drift monitoring pipelines
- adaptive ranking refresh systems
- retrieval-style behavioral similarity ranking
- CRM-native scoring APIs
- online feedback learning loops

---

# 21. Closing Insight

The central lesson from the project was:

> the best production ranking system is not necessarily the most complex model, but the system most aligned with operational constraints, explainability requirements, and real-world decision-making workflows.

The project demonstrated how:
- ranking reformulation
- operational alignment
- explainability-aware design
- imbalance-aware modeling

can outperform more theoretically complex approaches in practical ML system design.

---

## Disclaimer

This repository represents a generalized and sanitized reconstruction of production-inspired ML system design work.

Sensitive datasets, infrastructure details, proprietary logic, deployment systems, and organization-specific implementations have been removed or abstracted.