# Diamond Valuation & Offer Optimization  
**Enova Technical Assessment — by Nikola Stojkovski**

---

## Overview
This project simulates a diamond business scenario where I must **replenish inventory and maximize expected profit** under strict budget and rejection constraints.  
The analysis uses machine learning, regression, and exploratory data analysis to estimate optimal offer prices for diamonds currently for sale.

**Goal:**  
- Maximize expected profit while minimizing risk  
- Stay within a **$5,000,000** budget  
- Account for **offer rejections**, which still count against the budget  

---

## Datasets
Two datasets were provided:  
- **Training.csv:** Historical data on diamond sales, including offer prices, retail prices, and diamond attributes  
- **Offers.csv:** Current diamonds available for purchase, with attributes but no prices  

Both datasets contain detailed attributes such as:
`Carat, Clarity, Color, Cut, Depth %, Table %, Shape, Certification, Country, Conflict Status, Vendor`

---

## Data Cleaning & Preprocessing
Key steps:
- Handled extensive missing data (notably 90% unknown `Conflict_Diamond` in Offers dataset)
- Converted ordinal variables (`Cut`, `Polish`, `Symmetry`, `Clarity`) into numeric scales
- Corrected typos and inconsistencies in categorical fields (`Shape`, `Color`)
- Consolidated sparse `Color` categories (e.g., fancy/light/very light into one group)
- Extracted `Length`, `Width`, and `Depth` from measurements and engineered features:
  - `Volume = Length × Width × Depth`
  - `Length/Width Ratio = Length ÷ Width`
  - `Depth % = Depth / ((Length + Width) / 2) × 100`
- Encoded categorical variables with dummy variables
- Replaced missing `Depth` with calculated values and filled empty categorical fields

---

## Exploratory Data Analysis (EDA)
The EDA phase uncovered several insights:

### Vendor Pricing and ROI
- **Vendor 1** offered the cheapest diamonds; **Vendor 2** priced the highest.  
- However, **Vendor 2’s diamonds yielded the highest ROI**, suggesting *undercharging*.  
- **Vendor 4** showed the lowest ROI (≈20% below others), implying *overcharging*.

### Vendor Product Mix
- **Vendor 2** primarily sold larger, higher-end diamonds.  
- **Vendor 1** focused on smaller, budget-friendly stones.

### Conflict Diamonds
- Conflict diamonds only appeared in **Angola**, **DR Congo**, and **Zimbabwe**.  
- All diamonds from **Australia**, **Botswana**, **Canada**, **Russia**, and **South Africa** were safely marked as non-conflict.

### Feature Correlations
- `Carat` was the primary driver of both offer and retail price.  
- `Conflict_Diamond` correlated negatively (-0.34) with log retail price → significant feature.  
- Strong multicollinearity between volume-based metrics and carat.

---

## Regression Analysis
An **OLS regression** was used to study feature relationships and test for nonlinearity.

**Results:**
- Initial model: Adjusted R² = **0.843**
- Adding **quadratic and cubic terms** for `Carat` improved fit dramatically to **0.930**
- Strong evidence of a **cubic relationship between Carat and Retail Price**
- `Conflict_Diamond` remained statistically significant — cannot be dropped

**Interpretation:**  
Price increases nonlinearly with carat weight due to rarity effects — larger diamonds are exponentially rarer and thus disproportionately more valuable.

---

## Model Building
Four separate boosted tree regression models were trained to predict `Ln(Price)` and `Ln(Retail)`:

| Model | Conflict Status | Target | Algorithm | R² (Test) |
|--------|------------------|----------|------------|------------|
| 1 | Known | Ln(Price) | XGBoost | 0.982 |
| 2 | Missing | Ln(Price) | XGBoost | 0.978 |
| 3 | Known | Ln(Retail) | XGBoost | 0.981 |
| 4 | Missing | Ln(Retail) | XGBoost | 0.862 |

**Notes:**
- Models 1–3 performed excellently (R² > 0.98)
- Model 4 suffered from missing `Conflict_Diamond` data  
- Attempted to impute/predict conflict status, but limited improvement  
- XGBoost chosen for speed and performance after comparing LightGBM and CatBoost

---

## Offer Strategy
Using the best-performing model predictions:
- Applied a **30% safety buffer** to mitigate rejection risk (model slightly underestimates price)
- Sorted diamonds by **predicted ROI**
- Iteratively selected diamonds until hitting the **$5M budget**
- Generated optimized offer sheet maximizing expected profit under constraints

---

## Key Findings
- **Vendor 4** appears to overcharge (~20% lower ROI)
- **Vendor 2** likely undercharges despite higher sticker prices — focuses on larger diamonds
- **Carat–Price relationship is cubic**, reflecting rarity scaling
- **Conflict status** has a significant negative effect on retail price
- Missing conflict data is the primary barrier to further performance gains

---

## Tools & Libraries
**Python:** pandas, NumPy, scikit-learn, XGBoost, matplotlib, seaborn, statsmodels  
**Techniques:** Feature engineering, OLS regression, EDA, cross-validation, hyperparameter tuning

---

## Key Takeaways
- Accurate valuation in imperfect data environments requires a mix of domain knowledge, regression intuition, and robust ML methods.
- Conflict diamond data quality is a major risk factor in pricing models.
- Predictive modeling can outperform intuition when grounded in strong EDA and statistical validation.

---
