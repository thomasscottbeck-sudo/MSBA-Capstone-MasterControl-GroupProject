# MasterControl: Lead Scoring & Conversion Optimization
**Thomas Beck | MSBA IS 6813 Capstone | Spring 2026 | University of Utah**

---

## Business Problem

MasterControl sells quality management software to regulated industries through two product lines: Mx (manufacturing execution) and Qx (quality management). Leads enter the pipeline as Qualified Activity Leads (QALs) and progress toward SQL, SQO, or Won status.

The company's conversion rates diverge significantly by product: Mx converts at roughly 12.7% while Qx converts at 19.7%. With a $50 cost per outbound sales call and $6,000 value per SQL conversion, the ability to identify which leads are most likely to convert is a direct revenue lever. The current approach works the pipeline uniformly, which means high-value leads get no more attention than low-value ones.

**Project objective:** Build a lead scoring model that ranks incoming QALs by conversion probability, enabling the sales team to prioritize effort toward the highest-value opportunities.

---

## Our Solution

We built a binary classification model on 16,816 QAL records from MasterControl's Salesforce CRM. The target variable is whether a lead converts to SQL or beyond (approximately 15% positive class). Because accuracy is meaningless with 85/15 class imbalance, we optimized for AUC-ROC.

The solution has three components:

**1. Feature Engineering**

Raw CRM fields (industry, title, channel, account tier, territory) have limited predictive power on their own. I designed six domain-informed engineered features that encode business logic:

- *Intent Strength:* Composite score from channel type and priority label, mapping each combination to a calibrated conversion signal
- *Channel Efficiency Tier:* Categorical grouping of channels by historical conversion rate (Premium, Standard, Low-Value) derived from the data
- *Hidden Gem Flag:* Binary flag identifying accounts in high-fit industry segments that arrive via low-touch channels (underserved by the current sales motion)
- *Capital Density Score:* Continuous industry-level proxy for budget availability, distinguishing pharma/biotech from general manufacturing
- *Role-Product Match:* Continuous score mapping contact titles to product line fit based on domain knowledge of who buys Mx vs. Qx
- *Title NLP:* TF-IDF on contact titles reduced to 20 LSA components, capturing title signal beyond simple keyword matching

Additional features: ordinal seniority encoding, temporal decay (recency-weighted cohort signal), seniority-by-decision-relevance interaction, and industry-channel synergy terms.

**2. Model Tournament**

Five gradient boosting models (CatBoost, LightGBM, XGBoost, Gradient Boosting, Random Forest) plus a logistic regression benchmark, each tuned via randomized search (100-150 iterations, 5-fold stratified CV). SMOTE oversampling applied inside each CV fold via `imblearn.Pipeline` to prevent synthetic-sample leakage across folds — an important methodological detail that earlier versions got wrong.

Top performers combined into a stacking ensemble (LightGBM meta-learner) and a soft-voting ensemble. Champion selected based on validation AUC, with generalization gap (train vs. test AUC) as a secondary criterion.

**3. Profit Curve**

Predicted probabilities translated into a dollar-denominated profit curve ($50/call cost, $6,000/SQL value), identifying the exact score threshold that maximizes cumulative profit. This converts a statistical result into an operational recommendation the sales team can act on directly.

---

## My Contribution

I was the primary contributor to this project's technical work. Specifically, I built:

- The full feature engineering pipeline (all six engineered features plus interactions and temporal features)
- The modeling infrastructure: preprocessing pipeline, CV framework, SMOTE-inside-CV implementation, randomized hyperparameter search for all five candidate models
- The ensemble methods: stacking with LightGBM meta-learner and soft-voting ensemble
- SHAP explainability analysis using TreeExplainer
- The profit curve and business impact analysis
- The EDA notebook: funnel analysis, channel breakdown, seniority-function heatmap, feature distributions, conversion gap analysis
- All notebook versions from v1 through v13 (52 commits, full git history in the group repo)

---

## Business Value

The model demonstrates that contacting only the top-scored leads captures the majority of conversions at a fraction of the call volume. The profit curve shows a clear optimal threshold where incremental calls stop generating positive return.

Concrete recommendations that fall directly out of the analysis:

- Route high-probability leads (top decile) to closers immediately
- Deprioritize External Demand Gen and Email channels — both show poor conversion economics relative to cost
- Differentiate P1 intent: "Contact Us" is a strong signal and should go to senior reps; "Webinar" is weak and should enter nurture
- Target outbound prospecting toward Directors and VPs in Pharma & BioTech — this segment converts at a substantial multiple of the overall baseline
- Flag hidden gem accounts for dedicated outreach

---

## Difficulties

**Class imbalance.** At 85/15, a naive classifier achieves 85% accuracy by predicting "no conversion" for every lead. Standard accuracy metrics are misleading. We used AUC-ROC as the primary metric and SMOTE to address class imbalance during training.

**SMOTE leakage.** Early model versions applied SMOTE before cross-validation, which inflated CV scores by exposing the model to synthetic samples derived from the validation fold. Moving SMOTE inside the CV loop via `imblearn.Pipeline` corrected this and produced more honest generalization estimates. This was a meaningful methodological error that required rebuilding the training pipeline.

**Feature sparsity in CRM data.** Many records had missing or sentinel values in key fields (manufacturing model "Unknown," territory "Other," account tier gaps). Imputation choices had downstream effects on feature quality. Extensive exploratory analysis was required to distinguish true nulls from informative missingness.

**Sparse title data.** Contact titles vary widely in format and specificity. Parsing seniority (C-suite, VP, Director, Manager, Individual Contributor) from free-text required a multi-pass regex approach, and role-product matching required manual domain knowledge mapping rather than a clean categorical field.

---

## What I Learned

**Feature engineering matters more than model selection.** The ablation studies show that domain-informed features drive more AUC improvement than switching between gradient boosting implementations. All five candidate models performed within a narrow band once given the same feature set. The real lift came from encoding business logic.

**Methodological precision compounds.** The SMOTE leakage issue is a good example: a seemingly minor implementation detail (where in the pipeline SMOTE runs) produced meaningfully different and more honest model estimates. Getting this right required understanding the CV data flow precisely.

**Ensembles provide marginal but consistent improvement.** Stacking and voting ensembles added a small AUC increment over the best individual model. Whether that justifies added deployment complexity depends on the use case — for this project, a single CatBoost or LightGBM model is easier to explain to a non-technical sponsor.

**Profit curves are more useful than AUC for business audiences.** AUC tells you the model can rank leads; a profit curve tells you how many leads to call and what the dollar outcome looks like. Translating statistical metrics into operational terms was the most important communication step.

---

## Notebooks

| Notebook | Description |
|----------|-------------|
| [`MasterControl_EDA_vFinal.qmd`](notebooks/02_EDA/Thomas/MasterControl_EDA_vFinal.qmd) | Exploratory data analysis: funnel structure, channel efficiency, seniority heatmap, conversion gap analysis, feature distributions (1,564 lines) |
| [`mastercontrol_model_v13.qmd`](notebooks/03_Modeling/Thomas/mastercontrol_model_v13.qmd) | Full modeling pipeline: feature engineering, preprocessing, model tournament, ensembles, SHAP, profit curve, sponsor Q&A validation (3,201 lines) |

Both notebooks are self-contained and reproducible. All dependencies auto-install on first run. Data not included per course requirements.
