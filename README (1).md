# BallIQ — NFL Tight End Draft Analytics
### TAMIDS Student Data Challenge 2025

> **Identify undervalued TE prospects using NFL Combine data, college football production, and NFL career outcomes.**

[![Python](https://img.shields.io/badge/Python-3.9+-blue)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## Overview

This notebook is the full analytics engine behind the BallIQ draft intelligence platform. It walks through a four-stage pipeline — descriptive, diagnostic, predictive, and prescriptive — focused on the NFL Tight End position. The end result is a ranked 2026 draft board that flags undervalued prospects using a surplus value framework built on top of two machine learning models.

The core argument: NFL teams systematically over-weight combine athleticism and under-weight college production efficiency when evaluating tight end prospects. This creates a repeatable market inefficiency — especially in rounds 2–4 — that a data-driven approach can exploit.

---

## Repository Structure

```
te_draft_analysis_fixed.ipynb   ← Main notebook (run this)
Copy_of_TAMIDS_Combine_Data.xlsx
Copy_of_TE_CFB_Stats_2004-2025.xlsx
Copy_of_TE_NFL_STATS_2000_2024.xlsx
```

---

## Data

All data was provided by the TAMIDS competition organizers. Three Excel files are required:

| File | Description |
|------|-------------|
| `Copy_of_TAMIDS_Combine_Data.xlsx` | NFL Combine measurements for TEs (2000–2025) plus the 2026 prospect class. Sheets: `2000-2025 TE Data`, `2021 TE Data`, `2026 TE Data`. |
| `Copy_of_TE_CFB_Stats_2004-2025.xlsx` | College football receiving stats by player-season (11,167 rows). Includes receptions, yards, TDs, usage rate, and PPA per play. |
| `Copy_of_TE_NFL_STATS_2000_2024.xlsx` | NFL regular season receiving stats by player-season (2,668 rows). Includes EPA, yards, TDs, targets, PPR, target share, and entry year. |

Place all three files in the same directory as the notebook before running.

---

## Notebook Structure

### Section 1 — Data Loading & Cleaning
Loads all three data sources and handles several non-trivial preprocessing steps:
- **Height parsing**: Combine heights are stored as Excel date serials (e.g. `2026-06-04` = 6'4"). A custom parser extracts feet and inches and converts to total inches.
- **Draft string parsing**: Draft outcomes are stored as free-text strings (e.g. `"New England Patriots / 2nd round / 42nd pick / 2004"`). Regex extracts round, pick number, year, and team.
- **Name normalization**: All player names are lowercased, stripped of punctuation, and hyphen-normalized to enable cross-dataset matching. This process introduces ~15–20% merge attrition.
- **CFB aggregation**: Player-seasons are aggregated to three windows — career totals, peak single season, and final season — to capture trajectory in addition to volume.
- **NFL early-career window**: Outcomes are measured across years 0–2 post-draft entry only, using regular season data.
- **Success label**: Binary target = top 40% of early-career receiving EPA among drafted TEs (threshold ≈ 16.1 EPA; hit rate ≈ 39.9%).

---

### Section 2 — Descriptive Analytics
Establishes baseline distributions and trends across combine metrics and college production.

**Figures produced:**
- `fig1_combine_distributions.png` — Histograms comparing 6 combine metrics (40-yd, vertical, broad jump, 3-cone, shuttle, weight) for drafted vs. undrafted TEs. The 40-yd dash shows the sharpest separation; bench press the least.
- `fig2_draft_round_epa.png` — Draft volume by round (bar) and median/mean early-career EPA by round (line + bar). EPA drops sharply from R1 to R2, then plateaus through R4 — the basis for the surplus-value strategy.
- `fig3_cfb_trends.png` — Median college TE receiving yards, receptions, and TDs by season (2004–2025). All three have risen with the spread offense era, requiring era-relative comparisons.
- Combine summary table (printed) — Median combine metrics for drafted vs. undrafted TEs with percentage differences.

---

### Section 3 — Diagnostic Analytics
Identifies which factors actually explain NFL success, controlling for draft capital.

**Figures produced:**
- `fig5_correlation_heatmap.png` — Correlation matrix of all 21 features (8 combine + 13 college production) against early-career EPA. College production features (Career Yds, Peak Yds, Avg PPA) show the strongest positive correlations. Speed metrics (40-yd, 3-cone) show moderate negative correlations (lower = faster = better).

---

### Section 4 — Predictive Modeling
Trains and validates two models.

**Model 1 — Draft Pick Regression (Random Forest Regressor)**
- Target: actual draft pick number
- Purpose: predict where a prospect's profile implies they *should* be drafted
- Architecture: 300 trees, max depth 8
- Validation: 5-fold cross-validated MAE ≈ 48.7 picks (~1 round accuracy)

**Model 2 — Success Classifier (Random Forest Classifier)**
- Target: binary success label (top 40% early-career EPA)
- Three models compared: Logistic Regression (AUC 0.593), Random Forest (AUC 0.616 ± 0.078), Gradient Boosting (AUC 0.551)
- Random Forest selected as final model

**Figures produced:**
- `fig6_draft_capital_epa.png` — Draft pick number vs. early-career EPA scatter, annotating notable hits (Kittle, Gronkowski, Kelce). Confirms high variance within every pick range.
- `fig7_cfb_vs_nfl_epa.png` — Career college yards vs. early NFL EPA, colored by draft round.
- `fig8_model_comparison.png` — 5-fold AUC bar chart with error bars + ROC curve for the final Random Forest.
- `fig9_feature_importance.png` — Mean decrease in impurity by feature, color-coded (gold = college production, blue = combine). Career Rec, Career Yds, and Avg Usage rank above the 40-yd dash.
- `fig10_permutation_importance.png` — Permutation importance with standard deviation bars (SHAP proxy), confirming feature rankings from Fig 9.

---

### Section 5 — Prescriptive Analytics: Surplus Value & 2026 Recommendations
Converts model outputs into actionable draft intelligence.

**Surplus Value formula:**
```
Surplus Value = Predicted Pick (from Model 1) − Actual Pick
```
A positive surplus means the player was drafted later than their athletic/production profile warranted.

**Quadrant classification:**
- **Undervalued Hit**: Surplus > 0 AND Early EPA ≥ threshold → target archetype
- **Expected Hit**: Surplus ≤ 0 AND Early EPA ≥ threshold
- **Undervalued Miss**: Surplus > 0 AND Early EPA < threshold
- **Expected Miss**: Surplus ≤ 0 AND Early EPA < threshold

**2026 draft board:** Each prospect receives a composite score (60% success probability + 40% inverse predicted pick). Prospects outside Round 1 with ≥ 40% success probability are flagged as primary surplus-value targets.

**Figures produced:**
- `fig11_surplus_quadrant.png` — Surplus value quadrant chart for all 253 historical TEs.
- `fig12_2026_draft_board.png` — Top 20 prospects by composite score, colored by predicted draft tier.
- `fig13_value_picks.png` — Value pick recommendations: high-probability TEs projected outside Round 1.

---

## Key Findings

1. **College production dominates combine athleticism.** Career receiving yards, peak season yards, and PPA are the top three predictors of early-career NFL success. The 40-yd dash is the best combine metric but ranks fourth overall.
2. **The Rounds 3–4 value window is real.** Hit rates in rounds 3–4 (~38–40%) are nearly comparable to Round 2 at dramatically lower draft capital cost. A first-round pick on a TE buys only a ~15–17 percentage point increase in hit probability.
3. **PPA is the best single efficiency metric.** It normalizes for snap count and offensive system context, making it the most portable metric for cross-program comparisons.
4. **Draft pick number alone is a poor predictor.** High variance in EPA exists within every pick range — the market price of a prospect frequently diverges from their underlying value.

---

## Installation

```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost shap openpyxl
```

Then open and run `te_draft_analysis_fixed.ipynb` top to bottom. All figures are saved as `.png` files in the working directory.

---

## Supplementary Materials

- **Live dashboard**: [balliq.org](https://balliq.org) — interactive draft board with filters, surplus value explorer, and downloadable 2026 rankings
- **Visualization source code**: [github.com/Babji65/nfl-viz](https://github.com/Babji65/nfl-viz)
- **Output files**: `2026_te_draft_board.csv`, `historical_te_surplus.csv`

---

## Citation

> BallIQ Team. *NFL Tight End Draft Analytics.* TAMIDS Student Data Challenge, 2025.

---

*All data provided by TAMIDS competition organizers. All libraries used are open-source.*
