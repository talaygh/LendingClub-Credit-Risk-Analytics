# Loan Default Risk: Beating Lending Club's Pricing

> A data science approach to credit risk on $7.5B+ of consumer loans across an 11-year window (2007–2018).

**Author:** Talay Kamali · Data Analyst → Data Scientist
**Stack:** Python · Pandas · Scikit-learn · XGBoost · SHAP · Tableau · Power BI
**Data:** [LendingClub Accepted Loans, 2007–2018Q4](https://www.kaggle.com/datasets/wordsforthewise/lending-club) — 2.26M loans, 151 features
**Live dashboard:** [Tableau Public](https://public.tableau.com/app/profile/talay.kamali/vizzes)

---

## TL;DR

I built an XGBoost credit risk model on 2.26M Lending Club loans. **AUC 0.73** with isotonic-calibrated probabilities.

The model separates **16x risk** between the safest and riskiest deciles of borrowers (3% vs 53% default rate). The top 2 deciles capture **37% of all defaults**.

But the more interesting finding is what the simulation revealed: **Lending Club's grade-pricing is already well-calibrated at the portfolio level.** The 13% average interest rate covers default losses. A model-driven approval threshold barely improves global profit.

The real value is **sub-segment risk targeting**. Within every grade, the model identifies which loan purposes carry residual risk above the grade average — exactly where a model-driven price adjustment would matter in production.

---

## The story behind it

Lending Club priced every loan from 2007 to 2018 using a letter-grade system from A to G that mapped to interest rate. Grade A loans default ~6%; Grade G loans default ~49%. The system works directionally.

But within each grade, default rates vary 2–3x by loan purpose, geography, and debt load. A Grade A small-business loan defaults at 12% while a Grade A car loan defaults at 5% — both paying the same interest rate.

This project asks: **could a model have priced these loans more accurately than the grade system did, and where does it actually add value?**

---

## Project structure

```
loan-default-risk/
├── notebooks/
│   ├── 01_business_context.ipynb        # Framing and data overview
│   ├── 02_portfolio_analysis.ipynb      # Default patterns, mispricing heatmap
│   ├── 03_feature_engineering.ipynb     # Leakage handling, feature design, splits
│   ├── 04_modeling.ipynb                # LogReg, RF, XGBoost, SHAP, calibration
│   └── 05_business_impact.ipynb         # Risk decile analysis, segment mispricing
├── tableau/
│   └── Loan-Default-Risk-Dashboard.twbx # 5-page interactive Tableau dashboard
├── powerbi/
│   └── Loan-Default-Risk-Dashboard.pbix # 5-page interactive Power BI report
├── outputs/
│   ├── figures/                         # 10+ saved charts (PNG)
│   ├── loan_predictions.csv             # 59K test-set predictions
│   ├── xgb_model.pkl                    # Trained, calibrated model
│   └── columns_to_drop.json             # Feature audit results
├── data/                                # (gitignored — Kaggle dataset)
└── requirements.txt
```

Each notebook is self-contained: re-loads from raw CSV, applies the same 500K random sample with a fixed seed, and produces results that reconcile across the project.

---

## Interactive dashboards

The same 5-page analysis is built in **both Tableau and Power BI** to demonstrate fluency across the two leading BI platforms. Each tool tells the same story with its own native interactions.

### Pages (both versions)

1. **Portfolio Health** — KPIs (Total Originated, Default Rate, AUC, Risk Separation), vintage trends, grade-level default gradient, purpose risk profile (DTI vs default, sized by volume)
2. **Risk Decile Lab** — interactive what-if simulation with Approval Threshold and LGD parameters; cumulative defaults captured curve
3. **Segment Mispricing** — Grade × Purpose heatmap exposing where pricing doesn't match risk; pricing gap chart (charged rate vs actual default by grade)
4. **Loan Explorer** — drillable detail table by Grade → Sub Grade → Purpose
5. **Methodology** — model details, limitations, calibration plot (predicted vs actual default by decile)

The Tableau version is published live on [Tableau Public](https://public.tableau.com/app/profile/talay.kamali/vizzes). The Power BI version (`.pbix`) is in the `/powerbi` folder of this repo — clone and open in Power BI Desktop to interact with it.

---

## Key findings

### From the portfolio analysis (notebook 02)

- **Default rate ramps cleanly 6% → 49% across grades A–G** — the grade system is directionally correct
- **Within-grade variance is wide** — Grade A small-business loans default at 12% while Grade A car loans default at 5%, both at the same rate
- **Late-cycle loosening is visible in vintage data** — 2015–2017 cohorts default at 23% vs 12% for 2010–2011 cohorts, even as Lending Club tripled origination volume
- **DTI is the cleanest numeric signal** — default rate doubles from 15% (DTI<10) to 30% (DTI>40)
- **Geography matters** — Mississippi defaults at 27.5%, Oregon at 13.6%, both ignored by the grade

### From the model (notebook 04)

- **XGBoost wins:** AUC 0.7329 (vs 0.7223 LogReg, 0.7220 Random Forest) on the held-out test set
- **Calibrated via isotonic regression** — Brier score improved 30% post-calibration without sacrificing ranking quality
- **SHAP confirms expected drivers:** sub_grade, grade, term, DTI, issue_year, loan_to_income — the variables a credit analyst would expect, in the right direction
- **107 engineered features** including DTI tiers, FICO bands, credit history length, loan-to-income ratios, missingness flags

### From the impact analysis (notebook 05)

- **Risk separation works:** 16x spread between safest decile (3%) and riskiest decile (53%)
- **Top 2 risk deciles capture 37% of all defaults** despite being only 20% of loans
- **Threshold-based approval gains are marginal** — Lending Club's grade-pricing already covers losses at the portfolio level
- **Model adds value at sub-segment level** — 4 grade × purpose combinations show meaningful (>1pp) residual risk above grade average

---

## Methodology highlights

What separates this from a standard Kaggle notebook:

1. **Stratified train/test split balanced on default class AND vintage year** — both sets statistically representative of every issue year
2. **Imputation fit on training only, applied to both** — no test-set leakage in the fill values
3. **Encoding fit on training only** — target encoding for state uses smoothing toward the global mean for small states
4. **Grade and sub_grade decoded back to letter format (A, A1, B3, G5)** — preserves interpretability for business users in the BI layer
5. **Leakage column audit** — 23 post-default fields (`recoveries`, `total_rec_*`, `last_pymnt_*`) explicitly identified and dropped
6. **Class imbalance handled separately from probability calibration** — `scale_pos_weight` for ranking, isotonic regression for probabilities
7. **Three models compared on AUC, average precision, and Brier score** — not just accuracy, which is misleading on imbalanced data

---

## Limitations and honest caveats

- **500K random sample, not the full 2.26M** — preserved statistical structure across grades, vintages, states, and purposes; same pipeline scales to full dataset
- **Stratified random split, not forward-time** — production deployment would use a forward-time holdout to simulate vintage drift; mentioned for transparency
- **Loss severity assumed at 0.75** — Lending Club's actual recoveries vary; sensitivity analysis would refine this in production
- **`funded_amnt` and `funded_amnt_inv` are highly collinear** — survives in feature set; tree models handle it natively, but a future iteration would drop one for cleaner SHAP
- **Time value of money not discounted** — interest earned over 3-5 years treated as nominal; conservative on the model's claimed lift

---

## Reproducibility

```bash
# Clone the repo
git clone https://github.com/talaygh/loan-default-risk
cd loan-default-risk

# Install dependencies
pip install -r requirements.txt

# Download the dataset
# https://www.kaggle.com/datasets/wordsforthewise/lending-club
# Place accepted_2007_to_2018Q4.csv in ./data/

# Run notebooks in order
jupyter notebook notebooks/
```

Each notebook uses random seed `42` and the same 500K-row sample, so results are reproducible.

---

## Connect

[LinkedIn](https://www.linkedin.com/in/talaykamali) · [GitHub](https://github.com/talaygh) · [Tableau Public](https://public.tableau.com/app/profile/talay.kamali/vizzes) · [Email](mailto:talayapps@gmail.com)

If you found this useful — or have feedback on how I'd improve it — I'd love to hear from you.
