# Research Question & Confirmed Lane

**Status:** Confirmed. Lane locked to CTR / Engagement Opportunity Scoring (official lane 4 of 4).

## Lane

**CTR / Engagement Opportunity Scoring** — official lane 4 of 4: "find visible pages that
under-capture clicks or engagement and recommend metadata, content, or monitoring review."

## The question

Among pages already ranking for a given query/topic, which ones are earning fewer clicks
than their position and content characteristics would predict — and what is the single
highest-leverage fix for each?

## Unit of analysis

One row per (content item × query-topic cluster), aggregated over a rolling observation
window (e.g. 90 days) — not per raw query, and not per raw event.

## Output

A ranked list of content items, each with:
- an opportunity score (predicted CTR gap × traffic volume at stake)
- one specific recommended action (title rewrite, meta expansion, schema addition, or
  freshness/relevance review)

## The decision and action someone could take

A content or SEO team lead uses the ranked list as a prioritized backlog: work top-down,
applying the recommended fix to the highest-opportunity items first, instead of guessing
which of hundreds of pages to review.

## Cost of a wrong recommendation

- **False positive** (flagging a page that's actually fine): wastes a content editor's time
  on a rewrite that likely won't move the needle — low-severity but adds up at scale.
- **False negative** (missing a genuinely under-earning page): a real opportunity for
  traffic/click recovery goes unaddressed — opportunity cost, not direct harm.
- Neither error is safety-critical, which makes this a reasonable lane for a first model:
  mistakes are cheap to make and cheap to correct on the next review cycle.

## Why this isn't just "train a model"

- The hard part is the **baseline**: CTR is already well known to follow position in a
  predictable curve. A model only earns its place if it beats that curve meaningfully —
  so the project has to build and honestly report the baseline before any model result
  means anything.
- The hard part is **honest validation**: because content items can appear multiple times
  across query clusters, a naive random split leaks information (the model can "memorize"
  an item). This requires a grouped validation strategy (e.g. GroupKFold on content_id),
  not just calling `.fit()` and reporting a score.
- The hard part is **turning a number into an action**: a predicted-CTR-gap score is not
  itself useful to a content team. It has to be converted into a ranked, opportunity-weighted
  queue with a concrete recommended action per item — that translation step is a design
  choice, not a model output.
- The hard part is **staying inside safe, cautious claims**: it would be easy to overstate
  results as "this improves ranking" or "this explains Google's algorithm." The project has
  to consistently frame results as observed/directional, decision-support only.

## Why data/ML helps here (vs. a fixed rule)

A hand-written rule (e.g. "flag anything below position 10 with low CTR") is transparent but
blunt — it can't weigh multiple interacting signals (content age, title length, schema
presence, content type) at once. A model can combine these signals and be benchmarked
directly against the hand-rule baseline, so its value is measurable rather than assumed —
matching the same "rule vs. learned model, compared honestly" pattern already demonstrated in
the Week 1 starter notebooks.

## Public-safety compliance

No client names, domains, URLs, raw queries, or credentials appear in this document. All
identifiers referenced are pseudonymous, used only for grouping/joining, never as model
features. No claim in this document asserts proof of Google's ranking algorithm or a causal
refresh effect.
