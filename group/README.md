<div class="hero" align="center">
  <h1>MasterControl: QAL Lead Scoring &amp; Conversion Optimization</h1>
  <div class="meta"><strong>Group Deliverable &nbsp;|&nbsp; MSBA IS 6813 Capstone &nbsp;|&nbsp; Spring 2026</strong></div>
  <div class="badges">
    <a href="modeling/mastercontrol_model_v13-group.pdf"><img src="https://img.shields.io/badge/AUC--ROC-0.9147-00534B?style=flat-square" alt="AUC-ROC"></a>
    <a href="modeling/mastercontrol_model_v13-group.pdf"><img src="https://img.shields.io/badge/Top%20Decile%20Conv.-77.5%25-2980b9?style=flat-square" alt="Top Decile"></a>
    <a href="eda/MasterControl_EDA_vFinal.pdf"><img src="https://img.shields.io/badge/Baseline%20Conv.-17.9%25-95a5a6?style=flat-square" alt="Baseline"></a>
  </div>
  <div class="authors">Thomas Beck &nbsp;·&nbsp; Max Ridgeway &nbsp;·&nbsp; Astha KC</div>
</div>

## The Problem

MasterControl's SDRs work the QAL pipeline uniformly. Every lead gets the same attention regardless of how likely it is to convert. Mx converts at 12.6%, Qx at 19.7%, baseline 17.9%. The data contains clear signals that separate high-probability leads from low ones. The sales team is not using them.

---

## What We Recommend

Five actions that fall directly out of the analysis, ordered by confidence and ease of implementation.

**1. Score and tier every incoming QAL before SDR assignment.**
The voting ensemble ranks leads with a test AUC of 0.9147. Top-decile leads convert at 77.5%, a 4.3x lift over the 17.9% baseline. Operationalizing this requires a scored export from the model into the CRM, nothing more.

**2. Prioritize Pharma & Biotech accounts.**
Industry and manufacturing model are the second and third strongest predictors in SHAP. Pharma and Biotech organizations convert higher across seniority levels because regulatory compliance creates non-discretionary demand. Concentrating SDR effort here compounds.

**3. Engage senior contacts in Quality and Regulatory functions first.**
Seniority matters, but function matters as much as level. C-suite Regulatory converts at 43%, C-suite R&D at 50%, C-suite Quality at 21%. VP Quality converts at 31%. "Unknown" title records convert at 11.8%, well below baseline. Cleaning and enriching title data is a no-model improvement available today.

**4. Shift channel investment toward high-intent touchpoints.**
Intent strength is the single most predictive feature in the model. Webinars, personalized demos, and direct executive outreach outperform organic and email channels by a wide margin. Low-value channels are not just underperforming. They are introducing noise into the pipeline.

**5. Re-engage high-scoring recycled leads, filtered by score.**
Recycled leads make up the majority of the pipeline by volume and convert at roughly 13% overall, lower than first-touch at 19%. But the model scores recycled and active leads on the same scale. High-scoring recycled contacts represent recoverable pipeline at no incremental sourcing cost. Blanket re-engagement is not the recommendation. Filtered re-engagement is.

---

## What the Data Shows

| | |
|:---:|:---:|
| ![SHAP Feature Importance](../output/shap_importance.png) | ![Channel Conversion by Tier](../output/channel_tier_conversion.png) |
| **Top SHAP predictors (VotingEnsemble)** | **Conversion rate by channel tier** |

**Seniority × Function conversion rates (from EDA):**

| Function | C-Suite | VP | Director | Manager |
|----------|---------|-----|----------|---------|
| Quality | 21% | 31% | 24% | 27% |
| Regulatory | 43% | 29% | 18% | 12% |
| R&D | 50% | 25% | 17% | 30% |
| PMO | 33% | . | 24% | 16% |
| Mfg/Ops | 20% | . | 12% | 20% |

VP and C-suite contacts in Regulatory and R&D are the highest-converting segments. Quality at VP (31%) is the most reliable high-volume opportunity.

---

## How the Model Works

16,816 QAL records from Salesforce CRM. Binary target: SQL / SQO / Won vs. Recycled / Disqualified. Positive class 17.9%. Primary metric is AUC-ROC, chosen because accuracy is meaningless at that imbalance.

Five gradient boosting models tuned via randomized search with 5-fold stratified cross-validation. SMOTE inside the CV pipeline, not before, to prevent synthetic-sample leakage. Top candidates combined into a soft-voting ensemble. SHAP values explain each prediction in terms of the original features, so the sales team sees the reason, not just the score.

**Champion model:** VotingEnsemble. Test AUC 0.9147.

---

## Deliverables

| Notebook | Contents |
|----------|----------|
| [EDA (`MasterControl_EDA_vFinal`)](eda/MasterControl_EDA_vFinal.pdf) | Funnel analysis, conversion gap, channel breakdown, seniority × function heatmap |
| [Modeling (`mastercontrol_model_v13-group`)](modeling/mastercontrol_model_v13-group.pdf) | Feature engineering, model tournament, SHAP explainability, score distribution |

Both notebooks are available as `.qmd`, `.ipynb`, `.pdf`, and `.html` where applicable. Raw data is not included.

---

## Team

| Member | Contribution |
|--------|-------------|
| **Thomas Beck** | EDA, modeling pipeline, feature engineering, model tournament, ensembles, SHAP analysis |
| **Max Ridgeway** | Ran the modeling notebook end-to-end, debugged the CatBoost wrapper, rewrote the SHAP block for reliable execution, added navigational TOC |
| **Astha KC** | Built the final sponsor presentation and owned the delivery narrative; calibration sanity checks |
