---
name: brainstorming-ml
description: "Use when brainstorming a machine learning or data science project — defining the prediction target, choosing between classification/regression/clustering/forecasting, scoping training data, planning EDA, feature engineering, or model selection — before writing any code, notebook, or spec."
---

# Brainstorming ML & Data Science Projects

Help turn a machine learning or data science idea into a fully-specified design through staged, collaborative dialogue.

**REQUIRED BACKGROUND:** You MUST follow superpowers:brainstorming for the overall process — context exploration, one question at a time, the design doc, the spec self-review, the approval gates, and the handoff to writing-plans. This skill does not replace that process; it replaces the *content* of its "ask clarifying questions" and "propose approaches" steps with an ML-specific sequence, and adds ML-specific sections to the design doc.

<HARD-GATE>
Do NOT propose a model, a library, or an algorithm until the prediction target (Y) and problem type have been established with the user. Do NOT write any training code, notebook, or pipeline until the design has been presented and approved. This applies even when the user already names a model or technique in their first message.
</HARD-GATE>

## Anti-Pattern: "I Already Know Which Model To Use"

A named model in the request ("let's build an XGBoost model") is not a design. Jumping straight to model choice skips the decisions that actually determine success: what Y looks like, whether the model type even fits the problem, what data is available, and whether the metric matches the business goal. Ask the framing questions first — the answer may still be XGBoost, but now for a reason you can both defend.

## The ML Question Sequence

Work through these four stages in order, one question at a time, as the content for superpowers:brainstorming's "ask clarifying questions" step. Ask the user about what they observe and want — never ask them to name a problem type, a test, or a method themselves. You classify and recommend from their answers, then state it back for confirmation. Prefer multiple choice where the reference tables give you concrete options. Pull rows from `reference.md` into the conversation as they become relevant — don't dump full tables on the user unprompted.

**① Target & Data Scoping**
- What are you trying to predict or discover? Get them to describe it in plain terms — a yes/no outcome, a number, groups nobody's labeled yet, next period's value, something else.
- Do you have historical examples where the right answer is already known for each row (labels)? Roughly how many, and where did they come from?
- What data do you have access to today: type (tabular/text/image/time series/graph) and structure (one row per entity, per event, per time step, or spread across multiple tables)?
- From the answers, state back the problem type (classification/regression/clustering/ranking/forecasting/anomaly detection) and whether this is supervised/unsupervised/semi-supervised/self-supervised (`reference.md` → problem types, decision questions), and confirm it's right before moving on. Then suggest the feature families typical for this kind of problem (`reference.md` → data requirements) and ask what's actually available.

**② Exploratory Data Analysis**
- Ask whether the data is time-indexed, and whether they already know of data quality issues (missingness, duplicates, known outliers).
- From the answers, propose the EDA plan by name: structure checks, target distribution, univariate/bivariate views, variance, linearity, correlation/multicollinearity, and the specific hypothesis test(s) that fit the questions being asked of the data — e.g. "a chi-square test for X vs. the label," not just "hypothesis tests" (`reference.md` → EDA checklist). If time-indexed, name the specific seasonality, stationarity, trend, and autocorrelation tests too (e.g. STL decomposition, ADF, Mann-Kendall).
- Confirm the plan covers what they actually need to know before moving on.

**③ Feature Engineering**
- Ask whether the model needs to be interpretable (e.g., a regulatory or stakeholder requirement) or a black box is acceptable.
- Ask, as a separate question, what the compute/latency budget looks like (e.g. real-time scoring vs. periodic batch).
- From the answers and the data shape established in ①, recommend a dimensionality-reduction approach if one is warranted, and a feature-selection strategy (filter/wrapper/embedded), explaining the trade-off (`reference.md` → dimensionality reduction, feature selection).

**④ Data Pipeline**
- Ask whether multiple rows belong to the same entity (e.g., repeated customers) and whether there's a natural time cutoff in the data.
- From the answers, propose the split strategy that avoids leakage — random, stratified, group, or time-based — and the preprocessing order (`reference.md` → pipeline design). Flag any leakage risk you spot (target-derived features, transforms fit on the full dataset).

## Proposing Approaches

This is superpowers:brainstorming's "propose 2-3 approaches" step — do it after the question sequence above, not during it.

- Ask what matters more if there's a trade-off: catching more true positives, avoiding false alarms, or something else specific to the business goal. Use that, plus the problem type from ①, to pick the metric (`reference.md` → metric selection notes) — don't default to accuracy or RMSE without checking.
- Propose 2-3 candidate models. For each: how it works at a glance, its key hyperparameters, and a tuning approach (grid, random, Bayesian/Optuna, early stopping) (`reference.md` → candidate models, tuning approaches). Lead with your recommendation and why, per superpowers:brainstorming.

## Presenting the Design

Structure the design doc's sections around the stages above: Problem Framing (Y, problem type, supervised/unsupervised), Data Requirements, EDA Plan, Feature Engineering Plan, Pipeline Design, and Modeling & Evaluation Plan. Follow superpowers:brainstorming's rule of scaling each section to its complexity and getting approval section by section.

## Common Mistakes

| Mistake | Why it bites |
|---|---|
| Choosing a model before defining Y | The model choice depends entirely on Y's shape; picking first means re-litigating later |
| Using accuracy on an imbalanced classification problem | Accuracy stays high even when the model just predicts the majority class |
| Random-splitting time series or grouped data | Leaks future/entity information into training, inflating validation scores |
| Fitting scalers/encoders on the full dataset before splitting | Leaks test distribution into training, inflating validation scores |
| Running EDA without checking whether the data is time series | Skips stationarity/seasonality tests that change what "linear" and "correlated" even mean |
| Treating clustering metrics as ground truth | Silhouette/Davies-Bouldin score the geometry, not whether clusters mean anything to the business — always sanity-check with a stakeholder |
