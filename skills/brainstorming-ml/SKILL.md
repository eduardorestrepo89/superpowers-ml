---
name: brainstorming-ml
description: "Use when brainstorming a machine learning or data science project — defining the prediction target, choosing between classification/regression/clustering/forecasting, scoping training data, planning EDA, feature engineering, or model selection — before writing any code, notebook, or spec."
---

# Brainstorming ML & Data Science Projects

Help turn a machine learning or data science idea into a fully-specified design through staged, collaborative dialogue.

**REQUIRED BACKGROUND:** You MUST follow superpowers:brainstorming for the overall process — context exploration, one question at a time, the design doc, the spec self-review, and the approval gates. This skill does not replace that process; it replaces the *content* of its "ask clarifying questions" and "propose approaches" steps with an ML-specific sequence, and adds ML-specific sections to the design doc.

**Terminal handoff override:** superpowers:brainstorming states its terminal rule as "the ONLY skill you invoke after brainstorming is writing-plans." For ML projects, that rule is replaced by this one: **the next step after this skill is always superpowers:running-eda-ml, unless the user wants to refine or modify the spec first.** The spec's EDA Plan, Feature Engineering Plan, and Pipeline Design sections are still proposals at this point — running-eda-ml runs the EDA plan against real data and finalizes those sections before writing-plans is invoked.

**User Review Gate override:** superpowers:brainstorming's review-gate message asks whether the user wants changes "before we start writing out the implementation plan." For ML projects, ask about EDA readiness instead: "Spec written and committed to `<path>`. Is this ready to run the EDA against, or is there more to refine first?" If they want more refinement, keep looping — revise the spec and re-ask — rather than handing off early with an unclear or incomplete EDA Plan. Only once the user confirms it's ready, invoke superpowers:running-eda-ml — do not invoke writing-plans or any other skill. Running the EDA notebook is always the first thing running-eda-ml does — it doesn't do anything else first.

<HARD-GATE>
Do NOT propose a model, a library, or an algorithm until the problem type has been established and confirmed with the user — and, if the problem is supervised (or semi-/self-supervised), until the prediction target (Y) has also been established. Not every problem has a Y: clustering, dimensionality reduction, and unlabeled anomaly detection are unsupervised and are framed by problem type and data alone. Do NOT write any training code, notebook, or pipeline until the design has been presented and approved. This applies even when the user already names a model or technique in their first message.
</HARD-GATE>

## Anti-Pattern: "I Already Know Which Model To Use"

A named model in the request ("let's build an XGBoost model" or "let's cluster our customers") is not a design. Jumping straight to model choice skips the decisions that actually determine success: what the problem type is, what Y looks like if there's a Y to predict, whether the model type even fits the problem, what data is available, and whether the metric matches the business goal. Ask the framing questions first — the answer may still be XGBoost (or k-means), but now for a reason you can both defend.

## Beyond reference.md

`reference.md` is a fixed snapshot, not the ceiling on what you know or can find. Once the problem type is confirmed at the end of stage ①, and again at each later stage right before you'd pull a recommendation from `reference.md` (an EDA test, a dimensionality-reduction method, a candidate model, a metric), do one quick, narrowly-scoped web search for that specific problem type and topic — looking for anything newer or complementary that `reference.md` doesn't cover.

- **Search surfaces something genuinely complementary** (a newer technique, a metric better suited to this specific case, a library worth naming): fold it into what you tell the user alongside the `reference.md` content, so the recommendation reflects it.
- **Search only confirms what's already in `reference.md`**: say nothing about having searched, don't quote results, and don't let them linger in context — proceed with `reference.md`'s framing as-is. A search that adds no information shouldn't take up space in the conversation.

## The ML Question Sequence

Work through these four stages in order, one question at a time, as the content for superpowers:brainstorming's "ask clarifying questions" step. Ask the user about what they observe and want — never ask them to name a problem type, a test, or a method themselves. You classify and recommend from their answers, then state it back for confirmation. Prefer multiple choice where the reference tables give you concrete options. Pull rows from `reference.md` into the conversation as they become relevant — don't dump full tables on the user unprompted. Apply the "Beyond reference.md" check (above) whenever you're about to pull one of these rows in.

**① Target & Data Scoping**
- What are you trying to predict or discover? Get them to describe it in plain terms — a yes/no outcome, a number, groups nobody's labeled yet, next period's value, something else.
- Do you have historical examples where the right answer is already known for each row (labels)? Roughly how many, and where did they come from?
- What data do you have access to today: type (tabular/text/image/time series/graph) and structure (one row per entity, per event, per time step, or spread across multiple tables)?
- From the answers, state back the problem type (classification/regression/clustering/ranking/forecasting/anomaly detection) and whether this is supervised/unsupervised/semi-supervised/self-supervised (`reference.md` → problem types, decision questions), and confirm it's right before moving on. Then suggest the feature families typical for this kind of problem (`reference.md` → data requirements) and ask what's actually available.

**② Exploratory Data Analysis**
- Ask whether the data is time-indexed.
- Ask, as a separate question, whether they already know of data quality issues (missingness, duplicates, known outliers).
- From the answers, propose the EDA plan by name: structure checks, target distribution, univariate/bivariate views, variance, linearity, correlation/multicollinearity, and the specific hypothesis test(s) that fit the questions being asked of the data — e.g. "a chi-square test for X vs. the label," not just "hypothesis tests" (`reference.md` → EDA checklist). If time-indexed, name the specific seasonality, stationarity, trend, and autocorrelation tests too (e.g. STL decomposition, ADF, Mann-Kendall).
- Confirm the plan covers what they actually need to know before moving on.

**③ Feature Engineering**
- Ask whether the model needs to be interpretable (e.g., a regulatory or stakeholder requirement) or a black box is acceptable.
- Ask, as a separate question, what the compute/latency budget looks like (e.g. real-time scoring vs. periodic batch).
- From the answers and the data shape established in ①, recommend a dimensionality-reduction approach if one is warranted, and a feature-selection strategy (filter/wrapper/embedded), explaining the trade-off (`reference.md` → dimensionality reduction, feature selection).
- If there are categorical features, recommend an encoding approach (`reference.md` → encoding categorical features) — the right choice depends on model family (tree-based vs. linear/distance-based vs. neural) and cardinality, not on the feature alone. If the model hasn't been chosen yet, note which encoding fits which candidate family and revisit once Proposing Approaches settles on one.

**④ Data Pipeline**
- Ask whether multiple rows belong to the same entity (e.g., repeated customers).
- Ask, as a separate question, whether there's a natural time cutoff in the data.
- From the answers, propose the split strategy that avoids leakage — random, stratified, group, or time-based — and the preprocessing order (`reference.md` → pipeline design). Flag any leakage risk you spot (target-derived features, transforms fit on the full dataset).

## Proposing Approaches

This is superpowers:brainstorming's "propose 2-3 approaches" step — do it after the question sequence above, not during it. Run the "Beyond reference.md" check before finalizing the metric and the candidate models — this is the stage where a stale reference costs the most, since model families and tuning tools move fast.

- If ① established a Y (supervised/semi-supervised problem): ask what matters more if there's a trade-off — catching more true positives, avoiding false alarms, or something else specific to the business goal. Use that, plus the problem type, to pick the metric (`reference.md` → metric selection notes) — don't default to accuracy or RMSE without checking.
- If ① established there's no Y (unsupervised problem, e.g. clustering): pick metrics that don't require ground truth (`reference.md` → metric selection notes) and ask what would make the output actually useful to them — e.g. does a cluster or grouping need to make sense to a stakeholder, not just score well geometrically.
- Before naming any models, ask: do they want to see multiple candidate models compared against each other (a bake-off), or would they rather pool your knowledge and theirs about what tends to work best for this kind of problem and go straight to one recommendation? Their answer shapes what you propose next — don't decide this for them.
- **If they want a comparison:** propose 2-3 candidate models to benchmark. For each: how it works at a glance, its key hyperparameters, and a tuning approach (grid, random, Bayesian/Optuna, early stopping) (`reference.md` → candidate models, tuning approaches). Say which one you'd expect to win and why, but let the comparison settle it, per superpowers:brainstorming's lead-with-your-recommendation rule.
- **If they want the single best fit:** combine what you know about this model family's track record on similar problems with anything domain-specific they know (e.g., "we tried X before and it struggled with Y"), and reason to the one model that best matches the problem type, data shape, and the interpretability/compute constraints from ③. Propose that one model, with its key hyperparameters and tuning approach, and explain why you ruled out the obvious alternatives rather than listing them as live options.

## Presenting the Design

Structure the design doc's sections around the stages above: Problem Framing (problem type, supervised/unsupervised, Y if applicable), Data Requirements, EDA Plan, Feature Engineering Plan, Pipeline Design, and Modeling & Evaluation Plan. Follow superpowers:brainstorming's rule of scaling each section to its complexity and getting approval section by section.

## Common Mistakes

| Mistake | Why it bites |
|---|---|
| Choosing a model before defining the problem type (and Y, if one exists) | The model choice depends entirely on the problem type and Y's shape (or lack of one); picking first means re-litigating later |
| Using accuracy on an imbalanced classification problem | Accuracy stays high even when the model just predicts the majority class |
| Random-splitting time series or grouped data | Leaks future/entity information into training, inflating validation scores |
| Fitting scalers/encoders on the full dataset before splitting | Leaks test distribution into training, inflating validation scores |
| One-hot encoding a high-cardinality categorical for a tree-based model | Fragments splits across many dummy columns, dilutes feature importance, and forgoes the native categorical handling tree models are built for |
| Running EDA without checking whether the data is time series | Skips stationarity/seasonality tests that change what "linear" and "correlated" even mean |
| Treating clustering metrics as ground truth | Silhouette/Davies-Bouldin score the geometry, not whether clusters mean anything to the business — always sanity-check with a stakeholder |
