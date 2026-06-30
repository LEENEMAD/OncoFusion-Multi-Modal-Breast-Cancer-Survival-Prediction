[README.md](https://github.com/user-attachments/files/29512954/README.md)
# OncoFusion: Multi-Modal Breast Cancer Survival Prediction

A bioinformatics and clinical-AI pipeline that predicts breast cancer patient
survival by combining **transcriptomic gene-expression data** with **clinical
patient data**, then comparing biological, clinical, and fused (late-fusion)
modelling strategies. Built on the METABRIC cohort and designed around
interpretability (explainable AI) rather than raw accuracy alone.

> Course project — *Bioinformatics, Spring 2026*
> Author: **Leen Emad Jarad AlHwitat**

---

## Overview

Breast cancer is biologically heterogeneous: two patients with the same
clinical diagnosis can have very different survival outcomes. This project
investigates **which data modality best predicts survival** and whether
combining them helps. Specifically, it asks:

- Can transcriptomic gene-expression data predict patient survival?
- Do unsupervised molecular subgroups (clusters) relate to survival behaviour?
- How does biological data compare to clinical data for prediction?
- Does fusing clinical + transcriptomic predictions improve performance?

The workflow uses PCA and K-Means to explore transcriptomic structure, trains
four ML models on clinical, cluster-specific, and fused inputs, and applies
SHAP / EBM explainability to interpret the best models.

## Dataset

**METABRIC** (Molecular Taxonomy of Breast Cancer International Consortium) —
gene expression and clinical data for ~1,905 breast cancer patients from the
UK and Canada.

- Source: [Kaggle — Breast Cancer Gene Expression Profiles (METABRIC)](https://www.kaggle.com/datasets/raghadalharbi/breast-cancer-gene-expression-profiles-metabric)
- Expected file: `METABRIC_RNA_Mutation.csv` (place in the project root)
- ~1,905 samples · hundreds of gene-expression features (high-dimensional) ·
  multiple clinical variables (age at diagnosis, tumour size, lymph nodes,
  PAM50 + Claudin-low subtype, overall survival status, etc.)

> **Note:** the dataset is not included in this repository. Download it from
> Kaggle and place the CSV next to the notebook.

## Pipeline

1. **Data loading** — gene expression, clinical info, subtype annotations, survival labels
2. **Exploratory Data Analysis (EDA)** — distributions, missingness, class balance, subtype patterns
3. **Preprocessing** — stratified 80/20 split, median/most-frequent imputation, ordinal encoding, `StandardScaler`
4. **Feature selection** — `SelectKBest` (ANOVA F-test), 662 → top **150** genes
5. **PCA** — exploratory transcriptomic visualisation only (~85% variance retained; *not* used as model input)
6. **Clustering** — K-Means on gene expression (k chosen via Elbow + Silhouette → **3 clusters**)
7. **Cluster-based survival prediction** — a separate model per transcriptomic cluster
8. **Clinical survival prediction** — models trained on clinical features alone
9. **Late fusion** — meta-models trained on the combined clinical + cluster prediction probabilities
10. **Explainable AI (XAI)** — SHAP (late-fusion XGBoost) + EBM global feature importance

All models use **Stratified 5-Fold Cross-Validation** and **Grid Search**
hyperparameter tuning. Reusable helper functions standardise training,
evaluation, and visualisation across every stage.

## Models

Four classifiers are compared across all three input strategies:

| Model | Role |
|-------|------|
| Decision Tree | Transparent non-linear baseline |
| XGBoost | High-performance gradient boosting (suited to high-dimensional data) |
| Logistic Regression | Efficient linear baseline |
| Explainable Boosting Machine (EBM) | Interpretable, competitive performance |

**Evaluation metrics:** ROC-AUC and F1 (primary, due to class imbalance),
plus accuracy, precision, recall, and cross-validation stability.

## Key Results

- **Clinical EBM was the best and most reliable model** — test ROC-AUC **0.867**,
  CV ROC-AUC **0.852**, F1 **0.738**. The small test/CV gap indicates good generalisation.
- **Clinical models outperformed transcriptomic cluster-based models.** The best
  cluster model (Cluster 0 EBM) reached only ROC-AUC ~0.626.
- **Late fusion** (best: EBM, test ROC-AUC **0.821**, F1 **0.707**) did **not** beat
  the best clinical model; XAI confirms the clinical signal dominated the fused prediction.
- Transcriptomic clusters were **biologically meaningful** (differing survival rates
  and PAM50 composition) but **not strong standalone survival predictors** — PCA showed
  heavy overlap between living/deceased patients along the dominant components.
- The most influential clinical features were `overall_survival_months`, `cohort`,
  and `age_at_diagnosis`.

**Takeaway:** clinical data was the most direct predictor of survival, while
transcriptomic data was most valuable for understanding molecular heterogeneity.

## Tech Stack

- Python · Jupyter Notebook
- `pandas`, `numpy`, `matplotlib`, `seaborn`
- `scikit-learn` (preprocessing, feature selection, PCA, K-Means, models, CV, metrics)
- `xgboost`
- `interpret` (Explainable Boosting Machine)
- `shap`

## Getting Started

```bash
# 1. Install dependencies
pip install pandas numpy matplotlib seaborn scikit-learn xgboost interpret shap

# 2. Download METABRIC_RNA_Mutation.csv from Kaggle (link above)
#    and place it in the project root

# 3. Launch the notebook
jupyter notebook Leen_Alhwitat_Bioinfo.ipynb
```

Run the cells top to bottom — each stage feeds the next.

## Repository Contents

| File | Description |
|------|-------------|
| `Leen_Alhwitat_Bioinfo.ipynb` | Full analysis pipeline (101 cells) |
| `Leen_Alhwitat_Bioinfo.docx` | Detailed written report (problem definition, decisions, results, evaluation, reflection) |
| `README.md` | This file |

## Limitations & Future Work

- **No external validation** — tested only on a held-out METABRIC split; not yet
  clinically deployable.
- **Small per-cluster samples + class imbalance** (especially Cluster 1) made some
  cluster models unstable/degenerate.
- ANOVA F-test scores genes individually, ignoring gene–gene interactions and pathways.
- Future directions: pathway- or knowledge-based feature selection, embedding feature
  selection inside the CV loop to fully prevent leakage, larger/external cohorts, and
  bias screening before any clinical use.

## Ethical Note

This is a **research and educational** project. The models are not validated for
clinical decision-making and should not be used to inform real patient care.
