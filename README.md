# Credit Risk Decision System
### Stage 1 — Business Context

---

## The Problem

Non-Banking Financial Companies rely heavily on income and credit score as primary loan approval criteria. These indicators are standardized and auditable — but they have a fundamental resolution problem: borrowers with near-identical financial profiles often exhibit very different repayment behaviour.

This creates two compounding failures:

**Bad Approvals** — High-risk borrowers clear income-based thresholds and receive credit they will not repay. The lender absorbs the loss: unrecovered principal, collection costs, and capital provisioned against default. This is the visible, painful failure.

**Bad Rejections** — Creditworthy borrowers are screened out because blunt indicators cannot distinguish them from riskier applicants with similar surface profiles. The revenue on those loans is permanently foregone — and unlike defaults, this failure produces no visible signal. It compounds quietly at scale.

Both failures share a root cause — and a reason. Most traditional systems are explicitly calibrated to minimize Bad Approvals, because defaults are visible and measurable. Bad Rejections produce no signal, so they are systematically underweighted. The result is a bias baked into the decision framework itself.

That bias is compounded by the instruments used. Income reflects earning capacity. It does not capture employment stability, financial discipline, or how a borrower appears in external credit assessments. Systems built on income alone are structurally blind to the signals that actually predict repayment behaviour.

---

## What This System Does Differently

This project replaces the binary income/score heuristic with a probability-based decision system built on richer behavioural signals — particularly external credit assessment scores (EXT_SOURCE variables) and employment stability indicators.

The model outputs a default probability for each borrower. That probability maps to a risk tier, and each tier carries a defined policy action: approve, conditional approval, or reject.

The result is not just a model. It is an operational decision tool with three layers:

1. **Individual layer** — For any new applicant, the system produces a risk tier and a lending decision
2. **Portfolio layer** — Across the full borrower population, the system quantifies what each policy costs in defaults prevented vs. revenue foregone
3. **Explanation layer** — For each decision, the system generates a plain-language rationale a credit officer can read, audit, and act on

---

## Objective

Design a credit risk decision system that identifies the true behavioural drivers of default, segments borrowers into actionable risk tiers, and quantifies the tradeoff between risk reduction and business volume — producing decisions that are neither recklessly permissive nor unnecessarily restrictive.
