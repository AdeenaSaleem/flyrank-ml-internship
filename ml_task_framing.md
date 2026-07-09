# ML Task Framing

**Lane (provisional):** CTR / Position Modeling — see `research_question_framing.md` (ML-02).
Framing below will be re-checked once official lane names are confirmed (target: Week 4 lock).

## Task type

**Regression, feeding a ranking/scoring output.**

The core prediction is a continuous value — predicted CTR for a content item given its
position and page-level features. That's regression, not classification: there's no fixed
category being predicted, just a rate on a 0-1 (or 0-100%) scale.

The regression output is then *used* for ranking: content items are sorted by
`predicted_ctr − actual_ctr`, weighted by traffic volume, to produce the final ranked action
list. So the task is regression at the modeling step, ranking at the decision step — the
model doesn't need to directly optimize a ranking loss, because the thing we're ranking by
(the gap) is a simple downstream transform of the regression output.

Considered and rejected:
- **Classification** ("is this page under-performing: yes/no") — throws away the magnitude
  information (a page that's 2% off baseline vs. 15% off baseline) that the ranking step
  needs to prioritize correctly.
- **Clustering** — useful for grouping similar content items or queries (and may be a
  reasonable *supporting* step for query-topic bucketing per the data contract), but doesn't
  by itself answer "which pages should we fix first."

## Target / proxy

**Target:** observed CTR for a content item over the observation window (`clicks / impressions`).

This is a direct target, not a proxy — CTR is exactly the quantity we care about, not a stand-in
for something unmeasurable. The proxy risk in this lane is elsewhere: using CTR gap as a proxy
for "content quality" or "ranking-worthiness" would be overreaching. The target stays narrowly
defined as CTR itself; the *interpretation* of a large gap (title issue vs. meta issue vs.
genuine relevance problem) is a separate, rule-based step layered on top of the model output,
not learned by the model.

## Success metric

**Primary:** Mean Absolute Error (MAE) of predicted vs. actual CTR, measured out-of-fold
(grouped cross-validation by content item), compared against a position-only baseline curve.
A model that doesn't beat the baseline MAE by a meaningful margin isn't worth shipping over
just using position.

**Secondary:** Spearman rank correlation between predicted and actual CTR. Because the final
output is a ranked list, getting the *order* right matters as much as getting each point's
exact value right — a model could have a mediocre MAE but still order pages correctly, which
is what the downstream action queue actually needs.

**Not used as primary:** R² alone, since it's harder to communicate to a non-technical
stakeholder than "the model's average error dropped from X to Y."

## What action the output supports

A content/SEO team lead gets a ranked backlog: for each flagged content item, an opportunity
score and one recommended action (title rewrite, meta expansion, schema addition, or
freshness/relevance review). They work the list top-down instead of manually reviewing
pages with no prioritization.

## Why ML (not a fixed rule)

A fixed rule — e.g. "flag pages below position 15 with CTR under 2%" — only conditions on one
or two signals at a time and can't learn *interactions* between them (e.g. a short title only
hurts CTR when position is already borderline; a missing schema tag matters more for certain
content types). A regression model can weigh several signals jointly and its marginal value
over the rule/baseline is directly measurable (the baseline-vs-model MAE comparison above),
rather than assumed. This mirrors the Week 1 starter notebook exercise (hand rule vs. decision
tree, compared honestly on Precision@K) — same logic, applied to a continuous target instead
of a binary label.

## Public-safety compliance

No client names, domains, URLs, raw queries, or credentials appear in this document. No claim
here asserts proof of Google's ranking algorithm or a causal refresh effect — CTR is modeled
as a function of observable features, not as evidence about how search ranking itself works.
