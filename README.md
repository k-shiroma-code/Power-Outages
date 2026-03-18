## Introduction
**By: Kyle Shiroma and Marcus De Ramos**

Power outages hit millions of Americans every year and mess with daily life, businesses, even important infrastructure. But not all outages are the same. Some only last a couple hours, while others drag on for days. Figuring out what actually makes an outage more severe could help utility companies prepare better and use their resources more efficiently.

Question: **What characteristics are associated with more severe major power outages, and can we predict whether an outage will be severe?**

The dataset comes from Purdue University and contains **1,534 major power outage events** across the continental United States from 2000 to 2016. The columns most relevant to our analysis are:

| Column | Description |
|---|---|
| `OUTAGE.DURATION` | How long the outage lasted (in minutes) |
| `CAUSE.CATEGORY` | General cause of the outage (e.g., severe weather, intentional attack) |
| `NERC.REGION` | Which power grid reliability region the outage occurred in |
| `ANOMALY.LEVEL` | How much hotter or colder it was than usual (in °C) |
| `MONTH` | Month the outage occurred |
| `CUSTOMERS.AFFECTED` | Number of customers who lost power |
| `DEMAND.LOSS.MW` | Megawatts of electricity demand lost |
| `POPDEN_URBAN` | Urban population density |
| `POPDEN_RURAL` | Rural population density |
| `TOTAL.CUSTOMERS` | Total customers served in the state |

---

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

We performed the following cleaning steps:

1. **Dropped irrelevant columns**: Removed 33 columns not needed for our analysis like `POSTAL.CODE`, `HURRICANE.NAMES`, individual sector sales/prices/customer counts.
2. **Converted columns to numeric types**: Columns like `OUTAGE.DURATION`, `CUSTOMERS.AFFECTED`, and `DEMAND.LOSS.MW` were stored as objects, so we converted them to numeric values, coercing invalid entries to `NaN`.
3. **Combined date and time columns**: Merged `OUTAGE.START.DATE` + `OUTAGE.START.TIME` into a single `OUTAGE.START` datetime column, and did the same for restoration. Then dropped the original four columns.
4. **Cast `YEAR` and `MONTH` to integers** for cleaner analysis.
5. **Removed Alaska** to focus on the continental U.S.
6. **Feature engineered new columns**:
   - `DENSITY_GAP`: Ratio of urban to rural population density, showing how much denser cities are compared to rural areas.
   - `DENSITY_TIER`: Split states into 3 tiers (Low Gap, Medium Gap, High Gap) based on `DENSITY_GAP` using quantile cuts.
   - `SEVERE`: Binary label 1 if the outage duration is in the top 25% (≥ 2,880 minutes / 48 hours), 0 otherwise.

Here is the head of the cleaned DataFrame:


   
| OBS | YEAR | MONTH | U.S._STATE | NERC.REGION | ANOMALY.LEVEL | CLIMATE.CATEGORY | CAUSE.CATEGORY | OUTAGE.DURATION | CUSTOMERS.AFFECTED | DENSITY_TIER | SEVERE |
|-----|------|-------|------------|-------------|---------------|------------------|----------------|-----------------|--------------------|--------------|--------|
| 1 | 2011 | 7 | Minnesota | MRO | -0.3 | normal | severe weather | 3060 | 70000 | Low Gap | 1 |
| 2 | 2014 | 5 | Minnesota | MRO | -0.1 | normal | intentional attack | 1 | NaN | Low Gap | 0 |
| 3 | 2010 | 10 | Minnesota | MRO | -1.5 | cold | severe weather | 3000 | 70000 | Low Gap | 1 |
| 4 | 2012 | 6 | Minnesota | MRO | -0.1 | normal | severe weather | 2550 | 68200 | Low Gap | 0 |
| 5 | 2015 | 7 | Minnesota | MRO | 1.2 | warm | severe weather | 1740 | 250000 | Low Gap | 0 |





### Univariate Analysis

<iframe src="assets/fig_uni1.html" width="1000" height="450" frameborder="0"></iframe>

Most outages are relatively short, with the majority lasting under 2,000 minutes. However, there are a few extreme outages that last much longer, creating a right skewed distribution.

<iframe src="assets/fig_uni2.html" width="1000" height="450" frameborder="0"></iframe>

Severe weather is by far the leading cause of outages, followed by intentional attacks.

### Bivariate Analysis

<iframe src="assets/fig_bi1.html" width="1000" height="550" frameborder="0"></iframe>

This box plot shows outage duration across density tiers and cause categories. Severe weather consistently produces the longest outages regardless of density tier.

### Interesting Aggregates

| CAUSE.CATEGORY | Low Gap | Medium Gap | High Gap |
|---|---|---|---|
| equipment failure | 376.0 | 224.0 | 202.0 |
| fuel supply emergency | 7565.5 | 12240.0 | 1080.0 |
| intentional attack | 1.0 | 90.0 | 36.5 |
| islanding | 168.0 | 97.0 | NaN |
| public appeal | 1468.0 | 360.0 | 2685.0 |
| severe weather | 2880.0 | 2691.0 | 3059.5 |
| system operability disruption | 348.0 | 275.0 | 231.0 |

This pivot table shows the median outage duration in minutes for each cause across density tiers. Severe weather outages consistently have the longest durations across all tiers, while intentional attacks tend to be very short, especially in Low Gap states.

## Assessment of Missingness

### MNAR Analysis

We think the column `DEMAND.LOSS.MW` (megawatts of electricity demand lost) is MNAR (Missing Not at Random). It’s missing in about 46% of the rows, which is way higher than anything else in the dataset. A likely reason is that when outages cause very little demand loss, utilities don’t bother measuring or reporting it, so the data is missing because the value itself is small. That means the missingness depends on the actual unobserved value.

To make this MAR instead, we’d need more context like data on each utility’s reporting policies. For example, if utilities only report demand loss above a certain threshold, then the missingness could be explained by that rule rather than the value itself. In that case, it would be MAR.

### Missingness Dependency

We analyzed whether the missingness of `CUSTOMERS.AFFECTED` (28.9% missing) depends on other columns.

**Test 1: Missingness depends on `CAUSE.CATEGORY` (p = 0.0)**

We ran a permutation test using Total Variation Distance (TVD) as the test statistic, comparing the distribution of `CAUSE.CATEGORY` when `CUSTOMERS.AFFECTED` is missing vs. not missing. The observed TVD was 0.5581, and none of the 1,000 permutations produced a TVD that large, giving a p-value of 0.0. We reject the null at α = 0.05 The missingness of `CUSTOMERS.AFFECTED` depends on `CAUSE.CATEGORY`. This makes sense because certain causes like intentional attacks are far more likely to have missing customer counts.

<iframe src="assets/fig_perm1.html" width="1000" height="450" frameborder="0"></iframe>

**Test 2: Missingness does not depend on `COM.PERCEN`** (p = 0.986)

We ran a permutation test using the absolute difference in means of `COM.PERCEN` between rows where `CUSTOMERS.AFFECTED` is missing vs. not. The observed difference was just 0.0085, and the p-value was 0.986. We fail to reject the null at α = 0.05 The missingness of `CUSTOMERS.AFFECTED` does not depend on `COM.PERCEN`.

<iframe src="assets/fig_perm_com.html" width="1000" height="450" frameborder="0"></iframe>

---

## Hypothesis Testing

**Question:** Do states with a low urban rural density gap have significantly longer outages than states with a high density gap?

- **Null Hypothesis:** The mean outage duration is the same for Low Gap and High Gap states. Any observed difference is due to random chance.
- **Alternative Hypothesis:** Low Gap states have a higher mean outage duration than High Gap states.
- **Test Statistic:** Difference in mean duration (Low Gap − High Gap). We chose this because we are comparing a numerical quantity (duration) across two groups, and we have a directional hypothesis.
- **Significance Level:** α = 0.05

The observed difference was 906 minutes (~15 hours), with Low Gap states having longer outages on average. After 10,000 permutations, the p-value was 0.0007.

Since 0.0007 < 0.05, we reject the null hypothesis. There is strong evidence suggesting that states with more evenly spread populations (Low Gap) tend to experience longer outage durations than states with large urban-rural density divides (High Gap). This could be because repairs in evenly spread areas require crews to cover more ground rather than being concentrated in dense urban centers.

<iframe src="assets/fig_hyp.html" width="1000" height="450" frameborder="0"></iframe>

---

## Framing a Prediction Problem

We are predicting whether a power outage is severe, meaning it lasts in the top 25% of all outages (less than or equal to 2,880 minutes or 48 hours). This is a binary classification problem, where the target variable is SEVERE (1 = severe, 0 = not severe).

We chose this response variable because outage duration is the most direct measure of how disruptive an outage is, and identifying severe outages early could help utilities prioritize response efforts.

We evaluate our model using F1 score rather than accuracy. Because 75% of outages are not severe, a model that always predicts "not severe" would achieve 75% accuracy without learning anything useful. F1 score balances precision and recall, making it a better metric for imbalanced classes like ours.

All the features we use are available right when the outage starts. We don’t use things like duration, customers affected, or restoration time since those wouldn’t be known yet and would be data leakage.

---

## Baseline Model

Our baseline model is a **Logistic Regression** built in a single sklearn `Pipeline` with 4 features:

- `CAUSE.CATEGORY`: **nominal** (no natural order), one-hot encoded
- `NERC.REGION`: **nominal** (no natural order), one-hot encoded
- `ANOMALY.LEVEL`: **quantitative**, standardized with StandardScaler
- `MONTH`: **ordinal** (has natural order 1–12), standardized with StandardScaler

**Results:**

| | Accuracy | F1 Score |
|---|---|---|
| Training | 0.7869 | 0.5752 |
| Test | 0.7803 | 0.5379 |

The baseline model isn't that great. The accuracy is 78%, which is a little better than just always guessing "not sever" (75%), but the F1 score of 0.54 shows it's not that good at actually catching severe outages. The train and test scores are pretty close, so it performs similarly on train and test data, but it jsut doesn't have enough features or coplexity to really capture what makes an outage sever.

---

## Final Model

We improved the model by switching to a Random Forest Classifier and adding two new features on top of the baseline:

- `DENSITY_TIER`: **nominal**, one-hot encoded. This captures the urban rural population structure of the state. Our EDA and hypothesis test showed that density gap is strongly associated with outage duration, so it should help the model distinguish severe outages.
- `TOTAL.CUSTOMERS`: **quantitative**, standardized. States with more total customers have larger, more complex grids that may be harder to restore quickly, making this a useful signal for severity.

We used GridSearchCV with 5 fold cross-validation to tune the following hyperparameters:

- `n_estimators`: [50, 100, 200]
- `max_depth`: [5, 10, None]
- `min_samples_split`: [2, 5, 10]
- `max_features`: ['sqrt', 'log2']

The best hyperparameters were: `max_depth=10`, `max_features='sqrt'`, `min_samples_split=2`, `n_estimators=50`.

**Results:**

| | Accuracy | F1 Score |
|---|---|---|
| Training | 0.9208 | 0.8400 |
| Test | 0.8317 | 0.5920 |

The final model is better than the baseline in both accuracy (83.2% vs 78.0%) and F1 score (0.592 vs 0.538). There’s a bit of overfitting since the train and test F1 aren’t the same, but the test performance still improved by a solid amount. The added features and the Random Forest model help capture more complex patterns, which makes it better at identifying severe outages.

---

## Fairness Analysis

**Question:** Does our model perform worse for outages in "High Gap" states compared to "Low Gap" states?

- **Group X:** Low Gap (low urban rural density difference)
- **Group Y:** High Gap (high urban rural density difference)
- **Evaluation Metric:** Precision of the "Severe" class. What fraction of predicted severe outages are actually severe.
- **Null Hypothesis:** The model is fair. Its precision for Low Gap and High Gap states is roughly the same, and any difference is due to random chance.
- **Alternative Hypothesis:** The model is unfair. Its precision for Low Gap states is lower than its precision for High Gap states.
- **Test Statistic:** precision(High Gap) − precision(Low Gap)
- **Significance Level:** α = 0.05

**Results:** The precision for Low Gap was 0.6552 and for High Gap was 0.6667, giving an observed difference of just 0.0115. After 1,000 permutations, the p-value was 0.4870.

Since 0.4870 > 0.05, we fail to reject the null hypothesis. There is no significant evidence that the model performs unfairly between Low Gap and High Gap states. The small precision difference is consistent with random chance.

<iframe src="assets/fig_fair.html" width="1000" height="500" frameborder="0"></iframe>
