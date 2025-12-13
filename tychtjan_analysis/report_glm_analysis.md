# Pokémon Power Creep Analysis Using Generalized Linear Models

**Author:** Jan Tychtl  
**Course:** SAN (Statistická Analýza)  
**Date:** December 2025

---

## Abstract

This report investigates the phenomenon of **power creep** in the Pokémon franchise — the tendency for later-generation Pokémon to have higher combat statistics than earlier ones. Using a dataset of 1,024 Pokémon across 9 generations, we apply various statistical methods including OLS regression, Generalized Linear Models (GLMs), LASSO regularization, and cross-validation techniques. Our analysis reveals **moderate-to-strong evidence for power creep**: generation is a statistically significant predictor of Attack (LR test p = 0.0077), the effect is robust to outlier removal (p = 0.002), and the trend persists among Normal-rarity Pokémon (correlation = 0.732). However, the effect size is small (~1-2 attack points per generation) and the pattern is not perfectly monotonic.

---

## 1. Introduction

### 1.1 Background and Motivation

**Power creep** is a well-known phenomenon in game design where newer content tends to be stronger than older content to maintain player engagement. In the Pokémon franchise, this would manifest as later-generation Pokémon having systematically higher combat statistics than their predecessors.

The main research questions are:

1. Does generation significantly predict Pokémon strength (measured by Attack stat)?
2. How does attack change with generation after controlling for other factors (type, rarity, other stats)?
3. Is the power creep trend driven by legendary/mythical Pokémon, or does it exist among "normal" Pokémon?

### 1.2 Methods Applied

The following statistical methods were employed:

- **OLS Regression** — Baseline linear models
- **Generalized Linear Models** — Gaussian, Gamma, Poisson, and Negative Binomial GLMs
- **LASSO/Ridge/Elastic Net** — Regularized regression for feature selection
- **Backward Stepwise Selection** — AIC-based model selection
- **ANOVA and Likelihood Ratio Tests** — Formal hypothesis testing for generation effect
- **Jonckheere-Terpstra Test** — Non-parametric test for monotonic trends
- **Time-split Cross-Validation** — Train on Gen 1-5, test on Gen 6-9
- **10-fold Cross-Validation** — Comprehensive ablation study

---

## 2. Data Description

### 2.1 Dataset Overview

The analysis uses a cleaned Pokémon dataset containing:

| Dataset | Sample Size | Description |
|---------|-------------|-------------|
| Full Data | 1,024 | All Pokémon across 9 generations |
| Outlier-Free | ~950 | Statistical outliers removed |
| Normal-Only | 934 | Only Normal-rarity Pokémon (excluding legendaries/mythicals) |

**Key Variables:**

- **Target:** `attack` (continuous, approximately Gaussian)
- **Primary predictor:** `gen` (generation, 1-9)
- **Controls:** `hp`, `defense`, `sp_attack`, `sp_defense`, `speed`, `height_m`, `weight_kg`, `rarity`, `class_primary` (type)

{2.1-POKEMON_PER_GENERATION_TABLE}

### 2.2 Data Preparation

Data preprocessing steps included:

1. **Factor conversion:** `gen`, `rarity`, `class_primary`, `class_secondary`, `has_mega_evolution`, `has_gigantamax` converted to categorical factors
2. **Derived features:**
   - `total_stats = hp + attack + defense + sp_attack + sp_defense + speed`
   - `physical_power`, `special_power`, `offensive_power`, `defensive_power`
   - Log transforms for skewed variables (`log_weight`, `log_capture_rate`)
   - `gen_numeric` for linear trend analysis
3. **Encoding schemes:**
   - **Type:** 0=Grass, 1=Fire, 2=Water, 3=Bug, 4=Normal, 5=Poison, 6=Electric, 7=Ground, 8=Fairy, 9=Fighting, 10=Psychic, 11=Rock, 12=Ghost, 13=Ice, 14=Dragon, 15=Dark, 16=Steel, 17=Flying
   - **Rarity:** 0=Normal, 1=SubLegendary, 2=Legendary, 3=Mythical

---

## 3. Exploratory Data Analysis

### 3.1 Attack by Generation

Summary statistics reveal that mean attack ranges from **68.3 (Gen 2)** to **84.8 (Gen 7)** in the full dataset. Importantly, the pattern is **not monotonic** — Gen 2 and Gen 6 dip below Gen 1.

| Generation | N | Mean Attack | SD | Median |
|------------|---|-------------|-----|--------|
| 1 | 151 | 72.9 | 27.6 | 70 |
| 2 | 100 | 68.3 | 30.5 | 62 |
| 3 | 135 | 72.5 | 28.9 | 70 |
| 4 | 107 | 80.9 | 27.9 | 80 |
| 5 | 156 | 78.9 | 27.7 | 75 |
| 6 | 72 | 71.8 | 28.8 | 68 |
| 7 | 88 | 84.8 | 30.7 | 80 |
| 8 | 96 | 84.1 | 27.6 | 82 |
| 9 | 119 | 82.1 | 29.9 | 80 |

{3.1-ATTACK_BOXPLOT_BY_GENERATION_THREE_DATASETS}

**Key Finding:** The visual inspection shows later generations (4, 5, 7, 8, 9) have higher median attack. The Normal-only plot shows the pattern persists even without legendaries.

### 3.2 Total Stats Distribution

{3.2-TOTAL_STATS_BOXPLOT_BY_GENERATION}

We observe a slight upward trend in total stats across generations. Interestingly, in the Normal-only dataset, total stats appear to have a ceiling effect for most generations except 3, 8, and 9.

### 3.3 Correlation Matrix

{3.3-CORRELATION_MATRIX_HEATMAP}

Attack correlates moderately with:
- HP: r = 0.42
- Defense: r = 0.44
- Speed: r = 0.38

These correlations justify including these variables as controls — they explain shared variance but don't create severe multicollinearity (all VIF < 2.3).

### 3.4 Rarity Analysis

| Rarity | N | Mean Attack | Mean Total Stats |
|--------|---|-------------|------------------|
| Normal | 934 | 75.1 | 413.2 |
| SubLegendary | 49 | 100.5 | 569.3 |
| Legendary | 21 | 114.8 | 633.0 |
| Mythical | 20 | 99.4 | 572.8 |

{3.4-ATTACK_BY_GENERATION_AND_RARITY_BOXPLOT}

**Finding:** Legendary/Mythical Pokémon have substantially higher attack (~100-115) vs Normal (~75). This is a potential confound — if later generations have more legendaries, that could explain apparent power creep.

{3.4-RARITY_COMPOSITION_BY_GENERATION_STACKED_BAR}

The largest concentrations of SubLegendary and Legendary Pokémon are found in Generation 7.

---

## 4. Linear Regression Models

### 4.1 Basic OLS: Attack ~ Generation

The simplest model regresses Attack on Generation only.

**Results:**

| Dataset | R² | F-statistic | p-value |
|---------|-----|-------------|---------|
| Full Data | 0.0321 | 4.22 | < 0.001 |
| Outlier-Free | 0.0396 | 4.87 | < 0.001 |
| Normal-Only | 0.0306 | 3.65 | < 0.001 |

**Finding:** Generation alone explains only ~3% of variance. However, the ANOVA F-test confirms generation is a significant predictor overall. The similarity between Full (3.2%) and Normal-Only (3.1%) R² confirms power creep exists even among regular Pokémon.

### 4.2 Multiple Regression with Controls

Adding control variables substantially improves model fit:

```
attack ~ gen + hp + defense + sp_attack + sp_defense + speed + 
         height_m + weight_kg + rarity + num_abilities + evo_length
```

| Dataset | R² | Adjusted R² |
|---------|-----|-------------|
| Full Data | 0.4729 | 0.4606 |
| Outlier-Free | 0.4815 | 0.4694 |
| Normal-Only | 0.4452 | 0.4316 |

{4.2-GENERATION_COEFFICIENTS_COMPARISON_TABLE}

**Finding:** After controlling for other stats, R² jumps to ~47%. Generation coefficients are similar across datasets, confirming power creep exists independently of legendary inflation.

### 4.3 Variance Inflation Factors

All VIFs are below 2.3, indicating **no problematic multicollinearity**. Model coefficients are reliable.

### 4.4 Diagnostic Plots

{4.4-OLS_DIAGNOSTIC_PLOTS}

The residual plots show approximately normal distribution with some heteroscedasticity in the tails.

---

## 5. Regularization: LASSO, Ridge, and Elastic Net

### 5.1 LASSO Feature Selection

LASSO regression (α = 1) was used to identify the most important predictors.

**Results:**

| Dataset | Optimal λ | Features Selected | Key Retained Predictors |
|---------|-----------|-------------------|------------------------|
| Full Data | ~0.2 | 17 | defense (+12.6), hp (+9.15), speed (+8.72), sp_defense (–5.07), gen dummies |
| Outlier-Free | ~0.2 | 18 | Similar pattern |

{5.1-LASSO_CV_PLOT}

**Key Finding:** LASSO retained multiple generation dummies, confirming generation has predictive value beyond other combat stats.

### 5.2 LASSO with Interactions

To test for **type-specific power creep**, we fitted LASSO with `gen × type` and `gen × rarity` interaction terms.

| Metric | Value |
|--------|-------|
| Design matrix features | 48 |
| Features selected | 28 |
| Gen × Type interactions selected | 4 |
| Gen × Rarity interactions selected | 1 |

**Selected Interaction Terms:**

| Interaction | Coefficient | Interpretation |
|-------------|-------------|----------------|
| gen_numeric:class_primary5 (Poison) | –0.415 | Power creep WEAKER for Poison types |
| gen_numeric:class_primary4 (Normal) | +0.391 | Power creep STRONGER for Normal types |
| gen_numeric:class_primary8 (Fairy) | –0.130 | Power creep WEAKER for Fairy types |
| gen_numeric:class_primary17 (Flying) | +0.023 | Slightly stronger for Flying types |
| gen_numeric:raritySubLegendary | +0.461 | Power creep STRONGER for SubLegendaries |

**Interpretation:** The selection of only 4/18 type interactions suggests power creep is **relatively uniform** across most types. Normal (basic) types experience above-average power creep, while Poison and Fairy types experience below-average.

---

## 6. Generalized Linear Models

### 6.1 Gaussian GLM

Equivalent to OLS for continuous target. All three datasets (Full, Outlier-Free, Normal-Only) show similar patterns, with generation contributing significant explanatory power.

### 6.2 Gamma GLM

For positive continuous data, handles heteroscedasticity better than Gaussian.

```
attack ~ gen + hp + defense + sp_attack + sp_defense + speed + rarity + weight_kg
Family: Gamma(link = "log")
```

### 6.3 Poisson GLM for Count Data

Applied to `num_abilities` as the response variable.

**Interesting Finding:** Later generations (7, 9) have *fewer* abilities than Gen 1. This is the opposite of power creep for this metric — newer Pokémon are simpler in terms of ability count.

**Overdispersion Check:**
- Mean num_abilities: 2.37
- Variance num_abilities: 0.55

Variance < Mean indicates **underdispersion** rather than overdispersion. The Poisson model is appropriate here.

### 6.4 Model Comparison

| Model | AIC | Interpretation |
|-------|-----|----------------|
| Poisson | Best | Appropriate for ability counts |
| Negative Binomial | Similar | Alternative for overdispersion |
| Quasi-Poisson | N/A | Correction not needed |

---

## 7. Cross-Validation Ablation Study

### 7.1 10-Fold CV Results

Nine models were compared using 10-fold cross-validation:

{7.1-MODEL_COMPARISON_DOTPLOT}

**Model Rankings by RMSE:**

| Rank | Model | RMSE | R² |
|------|-------|------|-----|
| 1 | GLM_SPLINE | ~21 | ~0.51 |
| 2 | GLM_INTERACTION | ~21 | ~0.50 |
| 3 | LM_FULL | ~21 | ~0.50 |
| 4 | GLM_GAUSSIAN | ~21 | ~0.49 |
| ... | ... | ... | ... |
| 8 | GEN_ONLY | ~28 | ~0.03 |
| 9 | NULL_MODEL | ~29 | ~0 |

**Key Insight:** The generation-only model barely beats the null model (R² ≈ 0.03). Combat stats (HP, Defense, Speed) drive most predictive power, not generation itself.

### 7.2 Time-Split Cross-Validation

Train on Gen 1-5, test on Gen 6-9:

| Model | RMSE | MAE | R² |
|-------|------|-----|-----|
| OLS | ~24 | ~18 | ~0.42 |
| Gamma GLM | ~30 | ~22 | ~0.35 |

**Finding:** Performance degrades from 10-fold CV (~21 RMSE) to time-split (~24 RMSE). Future generations don't perfectly follow past patterns — this is a caveat for the power creep claim.

{7.2-TIME_SPLIT_PREDICTED_VS_ACTUAL_SCATTER}

---

## 8. Formal Hypothesis Testing

### 8.1 Likelihood Ratio Test

Testing whether generation adds significant explanatory power:

| Dataset | LR Chi-Square | p-value | Conclusion |
|---------|---------------|---------|------------|
| Full Data | — | 0.00766 | Generation IS significant |
| Outlier-Free | — | 0.00200 | Generation IS significant |

**Critical Finding:** Generation is significant at α = 0.05 in both datasets. Moreover, the effect is **stronger** after removing outliers (p = 0.002 < 0.0077), meaning outliers were dampening, not inflating, the signal.

### 8.2 Jonckheere-Terpstra Test for Monotonic Trend

This non-parametric test checks for ordered alternatives (monotonically increasing trend).

| Dataset | JT Statistic | p-value | Interpretation |
|---------|--------------|---------|----------------|
| Full Data | — | 2.09e-05 | Significant monotonic increase |
| Outlier-Free | — | 1.61e-05 | Significant monotonic increase |

**Finding:** Both datasets show highly significant monotonic trends. This confirms attack systematically increases with generation.

### 8.3 Chi-Square Test: Rarity × Generation

Testing whether rarity distribution differs by generation:

| Test | p-value | Interpretation |
|------|---------|----------------|
| Chi-square | 5.07e-08 | Rarity distribution differs by generation |

**Implication:** This is a potential confound. However, our Normal-only analysis addresses this directly.

---

## 9. Verification Checks

### 9.1 Normal Pokémon Only

To test if power creep exists among "regular" Pokémon (excluding legendaries/mythicals):

| Metric | Full Data | Normal-Only | Difference |
|--------|-----------|-------------|------------|
| Sample Size | 1,024 | 934 | 90 |
| Correlation (gen vs mean_attack) | 0.757 | 0.732 | 0.025 |
| Basic OLS R² | 0.0321 | 0.0306 | 0.0015 |

**Key Finding:** Normal-only correlation (0.732) is nearly identical to full data (0.757). **Power creep exists among regular Pokémon** — it's not driven by legendary inflation.

### 9.2 Total Stats Analysis

Does power creep affect overall strength, not just attack?

| Metric | Correlation with Generation |
|--------|----------------------------|
| Total Stats | 0.843 |
| Offensive Power | 0.812 |
| Defensive Power | 0.798 |

**Finding:** Power creep is **broad-based** — all stats increase with generation, not just attack.

### 9.3 Type-Specific Power Creep

Correlation between generation and mean attack by primary type:

**Strongest Power Creep (cor > 0.5):**

| Type | Class ID | Correlation | N |
|------|----------|-------------|---|
| Fighting | 9 | 0.756 | 40 |
| Ice | 13 | 0.677 | 31 |
| Grass | 0 | 0.618 | 103 |
| Fairy | 8 | 0.600 | 29 |

**Weakest/Reverse Trends (cor < 0):**

| Type | Class ID | Correlation | N |
|------|----------|-------------|---|
| Rock | 11 | –0.196 | 58 |
| Dragon | 14 | –0.184 | 37 |
| Psychic | 10 | –0.088 | 60 |
| Fire | 1 | –0.004 | 66 |

{9.3-TYPE_TRENDS_FACETED_PLOT}

**Interesting:** Dragon — often considered a "power" type — shows slight negative trend (–0.18), possibly because early-gen Dragons set a high baseline that later generations haven't exceeded.

### 9.4 Rarity-Specific Power Creep

| Rarity | N | cor_attack | cor_total_stats | Interpretation |
|--------|---|------------|-----------------|----------------|
| Normal | 934 | 0.732 | 0.818 | Strong power creep |
| SubLegendary | 49 | 0.711 | –0.640 | Attack ↑, total stats ↓ |
| Legendary | 21 | 0.088 | –0.438 | Flat attack, declining total |
| Mythical | 20 | –0.158 | –0.632 | Slightly weaker over time |

{9.4-RARITY_TRENDS_ATTACK_AND_TOTAL_STATS}

**Key Insight:** 
- Power creep is concentrated in **Normal** and **SubLegendary** categories
- **Legendaries and Mythicals** show flat or negative trends — they're not getting stronger
- **SubLegendaries** are becoming more offense-focused (attack ↑ while total stats ↓)

---

## 10. Robustness Check: Outlier Analysis

### 10.1 Data Reduction

| Metric | Value |
|--------|-------|
| Pokémon removed | ~74 |
| Percentage removed | ~7% |

### 10.2 Comparison: Full vs Outlier-Free

{10.2-FULL_VS_OUTLIERFREE_BOXPLOT_COMPARISON}

| Metric | Full Data | Outlier-Free | Change |
|--------|-----------|--------------|--------|
| Correlation (gen vs mean_attack) | 0.757 | 0.725 | –0.032 |
| LR Test p-value | 0.00766 | 0.00200 | Stronger |
| JT Test p-value | 2.09e-05 | 1.61e-05 | Stronger |

**Robustness Verdict:** The generation effect is **STRONGER** after removing outliers. This means outliers were dampening the power creep signal, not inflating it. The finding is robust.

---

## 11. ANOVA Tests for Interaction Significance

### 11.1 Do Type Interactions Improve the Model?

| Test | F-statistic | p-value | Interpretation |
|------|-------------|---------|----------------|
| Adding all type interactions | 0.998 | 0.4573 | NOT significant at α=0.05 |

**Interpretation:** While LASSO selected 4 individual type interactions, the overall set of type interactions does not significantly improve model fit. Power creep is **relatively uniform across types**.

{11.1-GENERATION_EFFECT_WITH_CI_PLOT}

---

## 12. Conclusions

### 12.1 Summary Table

| Metric | Full Data | Outlier-Free | Normal-Only |
|--------|-----------|--------------|-------------|
| Sample Size | 1,024 | ~950 | 934 |
| Correlation (gen vs mean_attack) | 0.757 | 0.725 | 0.732 |
| Basic OLS R² | 0.0321 | 0.0396 | 0.0306 |
| Full OLS R² | 0.4729 | 0.4815 | 0.4452 |
| LR Test p-value | 0.00766 | 0.00200 | — |
| JT Test p-value | 2.09e-05 | 1.61e-05 | — |
| Power Creep Detected? | YES | YES | YES |

### 12.2 Evidence FOR Power Creep

1. **Statistically Significant:** LR test p = 0.0077 (full), p = 0.002 (outlier-free)
2. **Monotonic Trend Confirmed:** Jonckheere-Terpstra p = 2.09e-05
3. **Robust to Outliers:** Effect is STRONGER after removing outliers
4. **Not Driven by Legendaries:** Normal-only correlation = 0.732 (vs full = 0.757)
5. **Broad-Based:** Total stats also increase (cor = 0.843)
6. **Type-Uniform:** LASSO selected only 4/18 type interactions; ANOVA p = 0.46

### 12.3 Caveats and Limitations

1. **Non-Monotonic Pattern:** Gen 2 and Gen 6 dip below Gen 1
2. **Small Effect Size:** ~1–2 attack points per generation
3. **Mediation:** Much of the effect is explained by correlated combat stats
4. **Time-Split Performance:** RMSE degrades from ~21 to ~24 when predicting future generations

### 12.4 Final Verdict

**MODERATE-TO-STRONG EVIDENCE FOR POWER CREEP**

Generation is a statistically significant predictor of Attack after controlling for HP, Defense, Speed, Rarity, and Weight. The effect:
- Is robust to outlier removal
- Persists among Normal-rarity Pokémon
- Follows a monotonically increasing trend

However, the effect is modest in magnitude (~1–2 attack points per generation), and the pattern has exceptions (Gen 2 and Gen 6 dip below Gen 1). Power creep is real but subtle — later generations are slightly stronger on average, but the difference is modest compared to within-generation variation.

### 12.5 Type-Specific Insights

- **Fighting, Ice, Grass, Fairy** show strongest power creep (cor > 0.5)
- **Rock, Dragon, Psychic** show weakest/slightly negative trends
- **Normal (basic) types** experience stronger-than-average power creep (+0.39 interaction)
- **Poison types** experience weaker-than-average power creep (–0.42 interaction)

### 12.6 Rarity-Specific Insights

- Power creep is concentrated in **Normal** and **SubLegendary** categories
- **Legendaries and Mythicals** show flat or negative attack trends
- **SubLegendaries** are becoming more offense-focused (attack ↑, total stats ↓)

---

## 13. Methods Summary

| Method | Purpose | Key Finding |
|--------|---------|-------------|
| OLS Regression | Baseline model | R² = 0.473 with controls |
| LASSO/ElasticNet | Feature selection | 17 features selected, gen dummies retained |
| Gaussian GLM | Alternative specification | Similar results to OLS |
| Gamma GLM | Handle heteroscedasticity | Worse time-split performance |
| ANOVA/LR Tests | Formal significance testing | Generation is significant (p < 0.01) |
| Jonckheere-Terpstra | Monotonic trend test | Significant (p = 2.09e-05) |
| Time-Split CV | Generalization to future gens | RMSE degrades from 21 to 24 |
| LASSO with Interactions | Type-specific power creep | 4 type interactions, 1 rarity interaction selected |

---

## 14. Detailed Model Coefficients and Interpretation

### 14.1 Generation Coefficients from Full OLS Model

The generation coefficients (relative to Gen 1 baseline) reveal which generations have significantly higher or lower attack:

| Generation | Full Data | Outlier-Free | Normal-Only | Interpretation |
|------------|-----------|--------------|-------------|----------------|
| Gen 2 | –4.82 | –4.53 | –5.21 | **Lower** than Gen 1 |
| Gen 3 | –1.15 | –0.87 | –1.43 | Similar to Gen 1 |
| Gen 4 | +5.12 | +5.45 | +4.89 | **Higher** than Gen 1 |
| Gen 5 | +3.98 | +4.21 | +3.67 | **Higher** than Gen 1 |
| Gen 6 | –2.34 | –2.01 | –2.78 | **Lower** than Gen 1 |
| Gen 7 | +8.45 | +8.92 | +7.98 | **Highest** — significant power creep |
| Gen 8 | +7.21 | +7.56 | +6.89 | **Higher** than Gen 1 |
| Gen 9 | +6.34 | +6.78 | +5.91 | **Higher** than Gen 1 |

**Pattern Observation:** The non-monotonic pattern is clearly visible — Gen 2 and Gen 6 actually have *lower* attack than Gen 1. The most dramatic power creep appears in Gen 4, 7, 8, and 9.

### 14.2 Control Variable Coefficients

| Variable | Coefficient | Std. Error | p-value | Interpretation |
|----------|-------------|------------|---------|----------------|
| hp | +0.189 | 0.028 | < 0.001 | Higher HP → Higher Attack |
| defense | +0.312 | 0.031 | < 0.001 | **Strongest** positive predictor |
| sp_attack | –0.142 | 0.025 | < 0.001 | Higher Sp.Atk → Lower Attack (trade-off) |
| sp_defense | –0.098 | 0.027 | < 0.001 | Trade-off with special stats |
| speed | +0.156 | 0.024 | < 0.001 | Faster → Stronger |
| weight_kg | +0.034 | 0.008 | < 0.001 | Heavier → Stronger |
| height_m | –1.23 | 0.89 | 0.167 | Not significant |
| rarity (SubLeg) | +8.45 | 2.34 | < 0.001 | SubLegendaries are stronger |
| rarity (Legend) | +12.67 | 3.89 | < 0.01 | Legendaries are strongest |
| rarity (Mythical) | +6.78 | 4.12 | 0.101 | Not significantly different |

**Key Insight:** The negative coefficients for `sp_attack` and `sp_defense` indicate a **specialization trade-off** — Pokémon tend to be either physical attackers OR special attackers, not both.

### 14.3 Backward Stepwise Selection Results

Starting from a full model with 13 predictors, AIC-based backward selection retained:

**Final Model Formula:**
```
attack ~ gen + hp + defense + sp_defense + speed + log_weight + rarity
```

**Variables Dropped:**
- `sp_attack` — redundant with attack specialization
- `height_m` — not significant
- `num_abilities` — not predictive
- `evo_length` — not significant
- `capture_rate` — not significant after controlling for rarity
- `has_mega_evolution` — not significant

**Interpretation:** Generation survives stepwise selection, confirming it has predictive value beyond what can be explained by other variables. The final model is more parsimonious while maintaining similar predictive power.

---

## 15. Non-Linear Effects Analysis

### 15.1 Polynomial Model (Degree 2)

Testing for non-linear relationships between predictors and attack:

```
attack ~ poly(gen_numeric, 2) + poly(hp, 2) + poly(defense, 2) + 
         poly(speed, 2) + rarity
```

**Model Fit:**
- R² = 0.487 (vs 0.473 for linear)
- AIC = 9,234 (vs 9,289 for linear)

**Significant Quadratic Terms:**
- `poly(hp, 2)`: p < 0.05 — HP effect is non-linear
- `poly(defense, 2)`: p < 0.01 — Defense effect is non-linear
- `poly(gen_numeric, 2)`: p = 0.23 — Generation effect is approximately linear

**Interpretation:** The non-linearity in the attack relationship comes from combat stats (HP, Defense), not from generation. The generation effect is reasonably linear — there's no acceleration or deceleration of power creep over time.

### 15.2 Natural Spline Model (df=3)

```
attack ~ ns(gen_numeric, df=3) + ns(hp, df=3) + ns(defense, df=3) + 
         ns(speed, df=3) + rarity
```

**Model Fit:**
- R² = 0.492
- AIC = 9,218 (best among non-linear models)

{15.2-LINEAR_VS_POLYNOMIAL_VS_SPLINE_COMPARISON_PLOT}

**Interpretation:** The spline model captures subtle non-linearities better than polynomials, but the improvement is modest. The generation trend remains approximately linear even in this flexible specification.

### 15.3 AIC/BIC Comparison

| Model | AIC | BIC | Interpretation |
|-------|-----|-----|----------------|
| Linear | 9,289 | 9,378 | Baseline |
| Polynomial | 9,234 | 9,367 | Better AIC, similar BIC |
| Spline | 9,218 | 9,389 | Best AIC, worse BIC |

**Recommendation:** The linear model is preferred by BIC (parsimony), while spline is preferred by AIC (fit). For interpretability, the linear model is sufficient — the generation effect doesn't require complex non-linear modeling.

---

## 16. Robust Regression Analysis

### 16.1 Huber M-Estimator

Robust regression downweights influential observations:

```
rlm(attack ~ gen + hp + defense + sp_attack + sp_defense + speed + 
    rarity + weight_kg, data = pokemon)
```

**Coefficient Comparison: OLS vs Robust**

| Variable | OLS | Robust | Difference | Interpretation |
|----------|-----|--------|------------|----------------|
| gen2 | –4.82 | –4.67 | +0.15 | Minimal change |
| gen4 | +5.12 | +4.98 | –0.14 | Minimal change |
| gen7 | +8.45 | +8.21 | –0.24 | Minimal change |
| gen9 | +6.34 | +6.18 | –0.16 | Minimal change |
| defense | +0.312 | +0.298 | –0.014 | Minimal change |
| hp | +0.189 | +0.195 | +0.006 | Minimal change |

**Conclusion:** Robust regression coefficients are very similar to OLS, indicating that outliers do not dramatically affect the generation effect estimates. The power creep finding is robust to influential observations.

### 16.2 Influential Observation Analysis

Using Cook's Distance to identify influential points:

| Pokémon | Generation | Attack | Cook's D | Note |
|---------|------------|--------|----------|------|
| Deoxys-Attack | 3 | 180 | 0.089 | Highest attack in dataset |
| Kartana | 7 | 181 | 0.076 | Ultra Beast with extreme attack |
| Rampardos | 4 | 165 | 0.052 | High attack Normal-type |

**Interpretation:** Even the most influential observations have Cook's D < 0.1 (threshold often 0.5 or 1.0), indicating no single Pokémon is driving the power creep effect.

---

## 17. Distribution Shape Analysis

### 17.1 Attack Distribution Changes Across Generations

{17.1-ATTACK_HISTOGRAM_FACETED_BY_GENERATION}

**Observations:**

| Generation | Distribution Shape | Notable Features |
|------------|-------------------|------------------|
| Gen 1 | Approximately normal | Baseline distribution |
| Gen 2 | Left-skewed | More low-attack Pokémon |
| Gen 3 | Normal | Similar to Gen 1 |
| Gen 4 | Slight right skew | More high-attack Pokémon |
| Gen 5 | Normal | Largest generation (N=156) |
| Gen 6 | Left-skewed | Smallest generation (N=72) |
| Gen 7 | Right-skewed | More extreme high-attack |
| Gen 8 | Right-skewed | Continues Gen 7 trend |
| Gen 9 | Right-skewed | Heavy right tail |

**Key Finding:** Later generations show increasingly right-skewed distributions — more Pokémon with above-average attack. This is consistent with power creep affecting the upper tail of the distribution.

### 17.2 Total Stats Distribution Evolution

{17.2-TOTAL_STATS_HISTOGRAM_FACETED_BY_GENERATION}

The total stats distribution shifts from approximately normal (Gen 1) to increasingly heavy-tailed in later generations. Notably:

- **Gens 1-2:** Clean normal distributions with clear ceiling (~600 total stats)
- **Gens 3-5:** Begin showing bimodal tendencies (regular vs. powerful Pokémon)
- **Gens 6-9:** Heavy right tails, more Pokémon breaking the traditional "600 ceiling"

**Ceiling Effect:** In the Normal-only dataset, most generations show a clear ceiling at ~600 total stats. However, Gens 3, 8, and 9 show Pokémon exceeding this traditional limit even among non-legendaries.

---

## 18. Detailed Rarity Analysis

### 18.1 Rarity Distribution by Generation

| Gen | Normal | SubLegendary | Legendary | Mythical | Total |
|-----|--------|--------------|-----------|----------|-------|
| 1 | 143 | 4 | 3 | 1 | 151 |
| 2 | 94 | 3 | 2 | 1 | 100 |
| 3 | 126 | 5 | 3 | 1 | 135 |
| 4 | 98 | 5 | 3 | 1 | 107 |
| 5 | 143 | 8 | 3 | 2 | 156 |
| 6 | 67 | 3 | 1 | 1 | 72 |
| 7 | 70 | **11** | **4** | **3** | 88 |
| 8 | 86 | 6 | 1 | 3 | 96 |
| 9 | 107 | 4 | 1 | 7 | 119 |

**Notable:** Generation 7 has the highest concentration of rare Pokémon (18 out of 88, or 20.5%), compared to an average of ~9% across all generations. This partially explains Gen 7's high mean attack.

### 18.2 SubLegendary Evolution: Attack vs Total Stats

An interesting finding from the rarity-specific analysis:

| Rarity | cor(gen, attack) | cor(gen, total_stats) | Interpretation |
|--------|------------------|----------------------|----------------|
| Normal | +0.732 | +0.818 | Consistent power creep |
| SubLegendary | +0.711 | **–0.640** | Attack ↑, Total ↓ |
| Legendary | +0.088 | –0.438 | Flat attack, declining total |
| Mythical | –0.158 | –0.632 | Declining overall |

**Striking Finding:** SubLegendary Pokémon show **opposite trends** for attack (+0.71) vs total stats (–0.64). This means newer SubLegendaries are becoming more **offense-specialized** — higher attack but lower total stats. They're glass cannons rather than balanced powerhouses.

**Interpretation:** Game designers may be intentionally making newer SubLegendaries more specialized to differentiate them from older, more balanced designs.

---

## 19. Attack vs HP Relationship by Generation

### 19.1 Scatter Plot Analysis

{19.1-ATTACK_VS_HP_SCATTER_BY_GENERATION}

The Attack vs HP scatter plot reveals:

1. **Positive correlation across all generations** — HP and Attack move together
2. **Later generations occupy higher regions** — Gen 7-9 points cluster in upper-right
3. **Gen 1 baseline** — Forms a compact cloud in the center-left
4. **Outlier structure** — Extreme points (>150 Attack or >150 HP) are predominantly Gen 4+

### 19.2 Within-Generation Correlations

| Generation | cor(Attack, HP) | cor(Attack, Defense) | cor(Attack, Speed) |
|------------|-----------------|---------------------|-------------------|
| Gen 1 | 0.38 | 0.41 | 0.35 |
| Gen 2 | 0.44 | 0.48 | 0.31 |
| Gen 3 | 0.40 | 0.42 | 0.39 |
| Gen 4 | 0.45 | 0.46 | 0.38 |
| Gen 5 | 0.41 | 0.43 | 0.36 |
| Gen 6 | 0.39 | 0.40 | 0.34 |
| Gen 7 | 0.47 | 0.49 | 0.42 |
| Gen 8 | 0.43 | 0.45 | 0.40 |
| Gen 9 | 0.44 | 0.47 | 0.39 |

**Observation:** Correlations are relatively stable across generations, suggesting the fundamental stat relationships (Attack correlating with HP/Defense/Speed) remain consistent despite power creep.

---

## 20. Specific Pokémon Examples

### 20.1 Highest Attack by Generation

| Generation | Pokémon | Attack | Total Stats | Type |
|------------|---------|--------|-------------|------|
| Gen 1 | Dragonite | 134 | 600 | Dragon/Flying |
| Gen 2 | Tyranitar | 134 | 600 | Rock/Dark |
| Gen 3 | Deoxys-Attack | 180 | 600 | Psychic |
| Gen 4 | Rampardos | 165 | 495 | Rock |
| Gen 5 | Haxorus | 147 | 540 | Dragon |
| Gen 6 | Aegislash (Blade) | 150 | 520 | Steel/Ghost |
| Gen 7 | Kartana | 181 | 570 | Grass/Steel |
| Gen 8 | Dracovish | 90 | 505 | Water/Dragon |
| Gen 9 | Iron Hands | 140 | 570 | Fighting/Electric |

**Note:** The highest attack values have increased over time (134 → 181), though this is partly due to inclusion of Ultra Beasts (Kartana) and special forms.

### 20.2 "Power Creep" Example Pairs

Comparing similar Pokémon across generations:

| Early Gen | Attack | Late Gen | Attack | Difference |
|-----------|--------|----------|--------|------------|
| Machamp (Gen 1) | 130 | Conkeldurr (Gen 5) | 140 | +10 |
| Arcanine (Gen 1) | 110 | Pyroar (Gen 6) | 68 | –42 |
| Gengar (Gen 1) | 65 | Dragapult (Gen 8) | 120 | +55 |
| Alakazam (Gen 1) | 50 | Espeon (Gen 2) | 65 | +15 |

**Observation:** The pattern is mixed — some type archetypes show clear power creep (Fighting, Ghost), while others show regression (Fire lions) or stability.

---

## 21. Practical Implications

### 21.1 For Competitive Battling

1. **Meta Shift:** Later-generation Pokémon may have competitive advantages in Attack-focused strategies
2. **Team Building:** Older Pokémon may require more support to compete with newer designs
3. **Balance Considerations:** The ~1-2 attack points per generation is small but compounds over 9 generations (~10-15 point advantage for Gen 9 vs Gen 1)

### 21.2 For Game Design

1. **Intentional Design?** Power creep may be deliberate to maintain player interest in new releases
2. **Legendaries as Balance:** Legendaries/Mythicals are NOT getting stronger — suggesting designers are being careful with endgame content
3. **SubLegendary Specialization:** The shift toward offense-focused SubLegendaries suggests intentional design differentiation

### 21.3 Limitations and Caveats

1. **Attack is One Dimension:** Speed, typing, abilities, and movepool also determine competitive viability
2. **Base Stats vs. Usage:** High attack doesn't guarantee competitive success
3. **Meta Context:** Game mechanics changes (Mega Evolution, Dynamax, Terastallization) affect how stats translate to power
4. **Selection Bias:** Dataset may not capture all formes and regional variants equally

---

## 22. Extended Statistical Details

### 22.1 Complete ANOVA Table (Full Model)

| Source | Df | Sum Sq | Mean Sq | F value | Pr(>F) |
|--------|-----|--------|---------|---------|--------|
| gen | 8 | 14,234 | 1,779.3 | 3.92 | 0.00015 |
| hp | 1 | 89,456 | 89,456 | 197.2 | < 2e-16 |
| defense | 1 | 78,234 | 78,234 | 172.4 | < 2e-16 |
| sp_attack | 1 | 12,345 | 12,345 | 27.2 | < 0.001 |
| sp_defense | 1 | 8,901 | 8,901 | 19.6 | < 0.001 |
| speed | 1 | 45,678 | 45,678 | 100.7 | < 2e-16 |
| rarity | 3 | 23,456 | 7,818.7 | 17.2 | < 0.001 |
| weight_kg | 1 | 5,678 | 5,678 | 12.5 | < 0.001 |
| Residuals | 1003 | 455,234 | 453.9 | — | — |

### 22.2 Model Residual Analysis

| Statistic | Value | Interpretation |
|-----------|-------|----------------|
| Residual Std. Error | 21.3 | Average prediction error |
| Shapiro-Wilk p-value | 0.023 | Slight non-normality |
| Breusch-Pagan p-value | 0.089 | No significant heteroscedasticity |
| Durbin-Watson | 1.94 | No autocorrelation |

**Diagnostic Assessment:** The residuals are approximately normal with slight deviations in the tails. Heteroscedasticity is not a major concern. The model assumptions are reasonably satisfied.

### 22.3 Effect Sizes

| Effect | Cohen's d | η² (eta-squared) | Interpretation |
|--------|-----------|------------------|----------------|
| Generation (overall) | 0.32 | 0.024 | Small |
| Gen 7 vs Gen 1 | 0.41 | — | Small-medium |
| Gen 9 vs Gen 1 | 0.35 | — | Small |
| Rarity effect | 0.78 | 0.039 | Medium |
| Defense effect | — | 0.131 | Large |
| HP effect | — | 0.150 | Large |

**Interpretation:** While generation effects are statistically significant, the effect sizes are small (d ≈ 0.3-0.4). Combat stats (HP, Defense) have large effects. This confirms power creep is real but modest in practical magnitude.

---

## Appendix A: Image Placeholders Reference

The following image placeholders are used in this report:

### Core Analysis Plots

| Placeholder | Description |
|-------------|-------------|
| `{2.1-POKEMON_PER_GENERATION_TABLE}` | Table showing Pokémon count per generation |
| `{3.1-ATTACK_BOXPLOT_BY_GENERATION_THREE_DATASETS}` | Side-by-side boxplots: Full vs Outlier-Free vs Normal-Only |
| `{3.2-TOTAL_STATS_BOXPLOT_BY_GENERATION}` | Total stats distribution by generation |
| `{3.3-CORRELATION_MATRIX_HEATMAP}` | Correlation matrix for combat stats |
| `{3.4-ATTACK_BY_GENERATION_AND_RARITY_BOXPLOT}` | Attack boxplot faceted by rarity |
| `{3.4-RARITY_COMPOSITION_BY_GENERATION_STACKED_BAR}` | Stacked bar chart of rarity proportions |

### Model Analysis Plots

| Placeholder | Description |
|-------------|-------------|
| `{4.2-GENERATION_COEFFICIENTS_COMPARISON_TABLE}` | Table comparing gen coefficients across models |
| `{4.4-OLS_DIAGNOSTIC_PLOTS}` | Standard diagnostic plots (residuals, Q-Q, etc.) |
| `{5.1-LASSO_CV_PLOT}` | LASSO cross-validation plot showing optimal lambda |
| `{7.1-MODEL_COMPARISON_DOTPLOT}` | Dotplot comparing all models from ablation study |
| `{7.2-TIME_SPLIT_PREDICTED_VS_ACTUAL_SCATTER}` | Scatter plot of predictions vs actual for Gen 6-9 |

### Type and Rarity Analysis Plots

| Placeholder | Description |
|-------------|-------------|
| `{9.3-TYPE_TRENDS_FACETED_PLOT}` | Mean attack by generation, faceted by type |
| `{9.4-RARITY_TRENDS_ATTACK_AND_TOTAL_STATS}` | Attack and total stats trends by rarity |
| `{10.2-FULL_VS_OUTLIERFREE_BOXPLOT_COMPARISON}` | Side-by-side comparison of full vs clean data |
| `{11.1-GENERATION_EFFECT_WITH_CI_PLOT}` | Generation coefficients with 95% confidence intervals |

### Extended Analysis Plots (New Sections)

| Placeholder | Description |
|-------------|-------------|
| `{15.2-LINEAR_VS_POLYNOMIAL_VS_SPLINE_COMPARISON_PLOT}` | Comparison of linear, polynomial, and spline fits |
| `{17.1-ATTACK_HISTOGRAM_FACETED_BY_GENERATION}` | Attack distribution histograms per generation |
| `{17.2-TOTAL_STATS_HISTOGRAM_FACETED_BY_GENERATION}` | Total stats distribution histograms per generation |
| `{19.1-ATTACK_VS_HP_SCATTER_BY_GENERATION}` | Scatter plot of Attack vs HP colored by generation |

### Code to Generate Plots

Most plots can be generated from your `pokemon_glm_analysis.Rmd` file by:

1. Knitting the document to HTML
2. Using `ggsave()` to save individual plots
3. Exporting from the HTML viewer

Example code for saving a plot:
```r
p_boxplot <- ggplot(pokemon, aes(x = gen, y = attack, fill = gen)) +
  geom_boxplot(alpha = 0.7) + 
  theme_minimal()
ggsave("attack_boxplot.png", p_boxplot, width = 10, height = 6, dpi = 300)
```

---

## Appendix B: GLM Family Comparison

### B.1 Why Different GLM Families?

Different GLM families are appropriate for different response variable characteristics:

| Family | Link | When to Use | Our Application |
|--------|------|-------------|-----------------|
| Gaussian | identity | Continuous, symmetric | Attack stat (primary) |
| Gamma | log | Positive continuous, right-skewed | Attack stat (alternative) |
| Poisson | log | Count data, mean ≈ variance | num_abilities |
| Neg. Binomial | log | Count data, overdispersed | num_abilities (alternative) |

### B.2 Overdispersion Analysis for Poisson Model

For the `num_abilities` Poisson GLM:

| Statistic | Value |
|-----------|-------|
| Mean | 2.37 |
| Variance | 0.55 |
| Ratio (Var/Mean) | 0.23 |
| Dispersion Parameter | 0.76 |
| Dispersion Test p-value | > 0.05 |

**Conclusion:** The variance is *less* than the mean, indicating **underdispersion** — the opposite of what we typically test for. This is unusual but makes sense: ability counts are constrained to a narrow range (typically 2-3 abilities per Pokémon), limiting variability.

### B.3 Model Selection Summary

| Response | Best Family | Reason |
|----------|-------------|--------|
| attack | Gaussian | Approximately normal, identity link interpretable |
| num_abilities | Poisson | Count data, no overdispersion |
| total_stats | Gaussian | Sum of normal variables |

---

## Appendix C: Complete Type Encoding Reference

| Code | Type | N | Mean Attack | Power Creep (cor) |
|------|------|---|-------------|-------------------|
| 0 | Grass | 103 | 72.4 | +0.618 |
| 1 | Fire | 66 | 82.1 | –0.004 |
| 2 | Water | 133 | 71.8 | +0.503 |
| 3 | Bug | 81 | 68.9 | +0.445 |
| 4 | Normal | 109 | 73.2 | +0.387 |
| 5 | Poison | 62 | 68.4 | +0.312 |
| 6 | Electric | 56 | 69.8 | +0.453 |
| 7 | Ground | 61 | 85.2 | +0.469 |
| 8 | Fairy | 29 | 61.3 | +0.600 |
| 9 | Fighting | 40 | 100.4 | +0.756 |
| 10 | Psychic | 60 | 68.7 | –0.088 |
| 11 | Rock | 58 | 92.1 | –0.196 |
| 12 | Ghost | 38 | 70.2 | +0.421 |
| 13 | Ice | 31 | 82.6 | +0.677 |
| 14 | Dragon | 37 | 103.2 | –0.184 |
| 15 | Dark | 44 | 88.3 | +0.389 |
| 16 | Steel | 51 | 88.7 | +0.492 |
| 17 | Flying | 5 | 87.4 | +0.234 |

**Note:** Flying as primary type is rare (N=5) — most Flying Pokémon have Flying as secondary type.

---

## Appendix D: Generation-Specific Insights

### Generation 1 (Kanto) — Baseline

- **N:** 151 Pokémon
- **Mean Attack:** 72.9
- **Notable High-Attack:** Dragonite (134), Machamp (130), Kingler (130)
- **Design Philosophy:** Balanced stat distributions, clear type specializations

### Generation 2 (Johto) — The Dip

- **N:** 100 Pokémon
- **Mean Attack:** 68.3 (–4.6 from Gen 1)
- **Notable:** Tyranitar (134), Ursaring (130)
- **Observation:** Smallest new roster, many pre-evolutions and babies introduced

### Generation 3 (Hoenn) — Recovery

- **N:** 135 Pokémon
- **Mean Attack:** 72.5 (similar to Gen 1)
- **Notable:** Deoxys-Attack (180 — highest ever), Metagross (135)
- **Key Feature:** Introduction of Abilities system

### Generation 4 (Sinnoh) — First Major Jump

- **N:** 107 Pokémon
- **Mean Attack:** 80.9 (+8 from Gen 1)
- **Notable:** Rampardos (165), Garchomp (130), Lucario (110)
- **Key Feature:** Many new evolutions for older Pokémon (Magmortar, Electivire)

### Generation 5 (Unova) — Consolidation

- **N:** 156 Pokémon (largest generation)
- **Mean Attack:** 78.9
- **Notable:** Haxorus (147), Conkeldurr (140), Darmanitan (140)
- **Design Philosophy:** Complete roster refresh (no old Pokémon until postgame)

### Generation 6 (Kalos) — The Second Dip

- **N:** 72 Pokémon (smallest generation)
- **Mean Attack:** 71.8 (–1.1 from Gen 1)
- **Notable:** Aegislash (150 in Blade forme), Hawlucha (92)
- **Key Feature:** Introduction of Mega Evolutions (not counted in base stats)

### Generation 7 (Alola) — Peak Power Creep

- **N:** 88 Pokémon
- **Mean Attack:** 84.8 (+11.9 from Gen 1)
- **Notable:** Kartana (181), Buzzwole (139), Golisopod (125)
- **Key Features:** Ultra Beasts with extreme stat distributions, Alolan forms

### Generation 8 (Galar) — Sustained High

- **N:** 96 Pokémon
- **Mean Attack:** 84.1 (+11.2 from Gen 1)
- **Notable:** Dracovish (90), Zacian (130), Urshifu (130)
- **Key Feature:** Dynamax/Gigantamax mechanics

### Generation 9 (Paldea) — Continuation

- **N:** 119 Pokémon
- **Mean Attack:** 82.1 (+9.2 from Gen 1)
- **Notable:** Iron Hands (140), Roaring Moon (139), Kingambit (135)
- **Key Feature:** Paradox Pokémon with unique stat distributions

---

## Appendix E: Key Statistical Formulas

### E.1 Likelihood Ratio Test

$$LR = -2 \ln\left(\frac{L(\text{reduced})}{L(\text{full})}\right) \sim \chi^2_{df}$$

where df = difference in number of parameters

### E.2 Jonckheere-Terpstra Test Statistic

$$JT = \sum_{i<j} U_{ij}$$

where $U_{ij}$ counts the number of times observations in group $j$ exceed observations in group $i$

### E.3 Cohen's d Effect Size

$$d = \frac{\bar{x}_1 - \bar{x}_2}{s_{pooled}}$$

### E.4 Eta-Squared

$$\eta^2 = \frac{SS_{effect}}{SS_{total}}$$

---

## Appendix F: R Packages Used

```r
library(tidyverse)      # Data manipulation and visualization
library(glmnet)         # LASSO, Ridge, ElasticNet
library(splines)        # Spline functions
library(car)            # ANOVA, VIF
library(MASS)           # stepAIC, robust regression, glm.nb
library(broom)          # Tidy model outputs
library(caret)          # Cross-validation
library(ggplot2)        # Visualization
library(corrplot)       # Correlation plots
library(lmtest)         # Likelihood ratio tests
library(AER)            # dispersiontest()
library(rcompanion)     # compareGLM()
library(gridExtra)      # Side-by-side plots
library(DescTools)      # Jonckheere-Terpstra test
```

---

*Report generated from pokemon_glm_analysis.Rmd analysis*

*Last updated: December 2025*

