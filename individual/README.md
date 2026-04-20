<div class="hero" align="center">
  <h1>MasterControl: Lead Scoring &amp; Conversion Optimization</h1>
  <div class="meta"><strong>Individual Portfolio &nbsp;|&nbsp; Thomas Beck &nbsp;|&nbsp; MSBA IS 6813 Capstone &nbsp;|&nbsp; Spring 2026</strong></div>
  <div class="badges">
    <a href="modeling/mastercontrol_model_v13.pdf"><img src="https://img.shields.io/badge/AUC--ROC-0.9147-00534B?style=flat-square" alt="AUC-ROC"></a>
    <a href="modeling/mastercontrol_model_v13.pdf"><img src="https://img.shields.io/badge/Top%20Decile%20Conv.-77.5%25-2980b9?style=flat-square" alt="Top Decile"></a>
    <a href="eda/MasterControl_EDA_vFinal.pdf"><img src="https://img.shields.io/badge/Baseline%20Conv.-17.9%25-95a5a6?style=flat-square" alt="Baseline"></a>
  </div>
</div>

## Business Problem

MasterControl sells quality management software through two product lines: Mx (manufacturing execution) and Qx (quality management). Leads enter as QALs and progress to SQL, SQO, or Won.

Mx converts at 12.6%, Qx at 19.7%. Overall baseline is 17.9%. At $50 per SDR call, uniform pipeline coverage is expensive and leaves the highest-intent leads getting the same attention as the lowest.

**Objective:** rank every incoming QAL by conversion probability so the sales team can allocate effort toward the leads most likely to close.

---

## Our Solution

Binary classification on 16,816 QAL records from Salesforce CRM. Positive class is 17.9%, so accuracy is meaningless. Primary metric is AUC-ROC.

Three components:

**1. Feature engineering.** Raw CRM fields (industry, title, channel, tier, territory) are thin signal on their own. Six domain features encode business logic:

- *Intent Strength.* Composite of channel type and priority label.
- *Channel Efficiency Tier.* Premium / Standard / Low-Value grouping by historical conversion.
- *Hidden Gem Flag.* High-fit accounts arriving via low-touch channels.
- *Capital Density.* Industry-level budget proxy (Pharma/Biotech vs. general mfg).
- *Role-Product Match.* Title-to-product-line fit, scored continuously.
- *Title NLP.* TF-IDF reduced to 20 LSA components.

Plus ordinal seniority encoding, temporal decay, and seniority × decision-relevance interactions.

**2. Model tournament.** Five gradient boosting models (CatBoost, LightGBM, XGBoost, Gradient Boosting, Random Forest) plus a logistic regression benchmark. Randomized search, 5-fold stratified CV, SMOTE inside the pipeline. Top performers combined into a stacking ensemble (LightGBM meta-learner) and a soft-voting ensemble.

**3. Explainability.** SHAP TreeExplainer on the voting ensemble. The sales team sees the reason each lead is scored, not just the score.

---

## Results

| Metric | Value |
|--------|------|
| Test AUC-ROC | **0.9147** |
| Top decile conversion | **77.5%** |
| Baseline conversion | 17.9% |
| Top decile lift vs. baseline | **4.3x** |
| Champion model | VotingEnsemble |

Top SHAP predictors: intent_strength, industry × model interaction, account primary site function, manufacturing model, territory rollup, channel tier, lead age decay.

---

## My Contribution

I did the technical build end to end.

- **EDA:** the full `MasterControl_EDA_vFinal` notebook (1,564 lines). Funnel analysis, channel breakdown, seniority × function heatmap, conversion gap, feature distributions.
- **Modeling:** feature engineering pipeline (all six engineered features plus temporal and interaction terms), preprocessing pipeline, CV framework, SMOTE-inside-Pipeline implementation, randomized hyperparameter search across all five candidate models.
- **Ensembles:** stacking with LightGBM meta-learner and soft-voting ensemble.
- **Explainability:** SHAP analysis with TreeExplainer.
- **Notebook build:** compilation, versioning through v13, final write-up.

52 commits on `main`, full history in the repo. Max and Astha were heavily involved in ideation, brainstorming, and synthesis of modeling direction throughout the project. Max ran the modeling notebook end-to-end on his machine, debugged the CatBoost wrapper, and rewrote the SHAP block to run reliably. Astha built and delivered the final sponsor presentation, ran final review on each deliverable draft, and kept us on milestone checkpoints.

---

## Business Value

Contacting the top-scored decile captures the majority of conversions at a fraction of the call volume. The model lets the sales team stop treating every QAL the same and route based on evidence.

Concrete moves that fall out of the analysis:

- Score and tier every incoming QAL before SDR assignment
- Deprioritize External Demand Gen and Email, both of which show poor conversion economics
- Differentiate P1 intent: "Contact Us" to closers, "Webinar" to nurture
- Target Pharma & Biotech Directors and VPs, the highest-converting segment
- Flag hidden-gem accounts (high fit, low touch) for dedicated outreach

---

## Difficulties

**Sponsor pushback on the profit curve.** Early versions of the model output leaned hard on a dollar-denominated profit curve using $50/call and $6,000/SQL as fixed economics. When we presented to MasterControl, they pushed back. The unit economics were not that clean in practice, and a "max profit" figure computed on a test-set slice does not translate to annual revenue. I kept the score ranking and the top-decile framing, which are defensible, and dropped the dollar claims from the sponsor-facing narrative. Lesson: if you don't have verified unit economics from the sponsor, don't build them into the headline.

**Class imbalance at 82/18.** A null model predicting "no conversion" hits 82% accuracy and zero business value. Switched the North Star to AUC-ROC early and used SMOTE for training-time balance.

**SMOTE leakage.** Early notebook versions applied SMOTE before the CV split, which inflated CV scores by exposing the model to synthetic samples derived from the validation fold. Moving SMOTE inside the `imblearn.Pipeline` fixed it and tightened the generalization gap. Small implementation detail with a meaningful impact on honesty of the estimates.

**Feature sparsity.** "Unknown" manufacturing model, "Other" territory, gaps in tier rollup. Distinguishing true nulls from informative missingness took several passes. Ended up treating several sentinel values as their own category rather than imputing, which captured signal that imputation would have erased.

**Free-text titles.** Parsing seniority from raw title strings required multi-pass regex. Role-to-product mapping was manual domain knowledge, not a clean field. Title NLP helped but didn't eliminate the need for structure.

---

## What I Learned

**Feature engineering beats model selection.** All five candidate models landed inside a narrow AUC band once given the same feature set. Domain-informed features drove the lift.

**Methodological honesty compounds.** The SMOTE-before-CV mistake produced a model that looked better than it was. Once corrected, the CV and validation AUC converged and I trusted the numbers. Shortcuts on methodology always show up in the generalization gap.

**Sponsor trust beats technical polish.** The profit curve was technically correct and operationally useless because we didn't have verified inputs. The SHAP analysis and seniority × function heatmap were less flashy and more useful because they pointed at specific segments the sales team could act on. When the sponsor pushes back, the thing they're pushing back on is usually the thing you made up.

**Ensembles add marginal AUC.** Stacking and soft voting added a small increment over the best single model. Whether it's worth the deployment complexity depends on the stakes. For this project, a single CatBoost or LightGBM would ship faster and be easier to explain.

---

## Deliverables

| Artifact | Contents |
|----------|----------|
| [Business Problem Statement](bps/MasterControl_Business_Problem_Statement.pdf) | Graded Feb 1 submission. Scoping, objective, success criteria, analytic approach. |
| [`MasterControl_EDA_vFinal`](eda/MasterControl_EDA_vFinal.pdf) | Funnel, conversion gap, channel breakdown, seniority × function heatmap, feature distributions. 1,564 lines. |
| [`mastercontrol_model_v13`](modeling/mastercontrol_model_v13.pdf) | Full pipeline: feature engineering, preprocessing, model tournament, ensembles, SHAP, score distribution, sponsor Q&A validation. 3,201 lines. |
| [Final Presentation](presentation/MasterControl_Final_Presentation.pdf) | Sponsor-facing deck delivered to MasterControl. |

Notebooks are self-contained. Dependencies auto-install. Raw data not included per course requirements.

### Version History

The path from a first pass to the submitted v13 is worth seeing. Earlier iterations live in [`archive/notebooks/03_Modeling/Thomas/`](../archive/notebooks/03_Modeling/Thomas/):

- **v7 – v10:** initial feature builds, first SMOTE attempts, pre-leak-fix CV
- **v11 – v12:** SMOTE-inside-pipeline fix, expanded model tournament, first SHAP pass
- **Overfit Check:** `04_Model_Validation_Suite.qmd`, a generalization-gap audit
- **v13 (final):** benchmark logistic, five-model tournament with 150-iteration randomized search, stacking + voting ensembles, ablation studies, sponsor Q&A validation

EDA iterations are in [`archive/notebooks/02_EDA/Thomas/`](../archive/notebooks/02_EDA/Thomas/).

### Key Sections in the Modeling Notebook

Quick links into [`mastercontrol_model_v13.pdf`](modeling/mastercontrol_model_v13.pdf):

1. Introduction + business problem
2. Feature engineering (six domain features, decay, interactions)
3. Data at a glance: target distribution, channel tier, seniority × function heatmap, feature distributions, correlation matrix
4. Benchmark + model tournament
5. Stacking and voting ensembles, ablation studies
6. Test set evaluation: ROC, PR curves, calibration, confusion matrix
7. SHAP explainability + dependence plots
8. Profit analysis (kept for completeness; caveats in Difficulties above)
9. Sponsor Q&A validation
10. Results and conclusion
