# Power Outage Severity Analysis

**What characteristics are associated with higher severity major power outages in the continental United States?**

UCSD recently experienced a campus-wide power outage, which made me curious about what drives major outages and why some last hours while others last weeks. Using a dataset of 1,533 major outage events across the continental U.S. from 2000 to 2016, I investigated what characteristics are associated with higher severity outages — from cause and climate conditions to a state's urban-rural population distribution. The dataset originates from [Purdue University's LASCI Research Data](https://engineering.purdue.edu/LASCI/research-data/outages) and combines outage event details with regional climate, economic, and demographic variables, giving enough dimensions to explore severity from multiple angles and build predictive models. This project also connects to my upcoming internship at Southern California Edison (SCE), where I'll be working in grid performance within system planning and engineering.

---

## Introduction

Major power outages carry significant consequences for public safety, economic activity, and infrastructure reliability. Understanding what makes certain outages more severe than others — and whether severity can be predicted early — is valuable for utilities, emergency planners, and policymakers.

This analysis defines **outage severity** primarily through **outage duration**, with events in the top 25% of duration (≥ 48 hours) classified as "severe." The dataset contains 1,533 rows (after removing Alaska, which is not part of the continental U.S.) and 56 columns spanning outage event details, regional climate conditions, electricity pricing, customer demographics, and state-level economic indicators.

Key columns relevant to this analysis include:

| Column | Description |
|---|---|
| `YEAR`, `MONTH` | When the outage occurred |
| `U.S._STATE`, `POSTAL.CODE` | Location of the outage |
| `NERC.REGION` | North American Electric Reliability Corporation region |
| `CLIMATE.REGION`, `CLIMATE.CATEGORY` | U.S. Climate region and episode classification (warm/cold/normal) |
| `ANOMALY.LEVEL` | El Niño/La Niña oceanic index for the period |
| `CAUSE.CATEGORY` | High-level cause of the outage (e.g., severe weather, intentional attack) |
| `OUTAGE.DURATION` | Duration in minutes |
| `CUSTOMERS.AFFECTED` | Number of customers affected |
| `POPDEN_URBAN`, `POPDEN_RURAL` | Population density in urban and rural areas (persons per sq mile) |
| `TOTAL.CUSTOMERS` | Total customers served in the state |
| `POPULATION`, `POPPCT_URBAN` | State population and urbanization rate |
| `PC.REALGSP.STATE` | Per capita real gross state product |

---

## Data Cleaning and Exploratory Data Analysis

### Cleaning Steps

1. **Header extraction:** The raw Excel file contains metadata rows above the actual data. The first 5 rows were skipped to extract the correct header, and the units row was dropped.
2. **Datetime combination:** `OUTAGE.START.DATE` and `OUTAGE.START.TIME` were combined into a single `OUTAGE.START` datetime column. The same was done for restoration date and time to create `OUTAGE.RESTORATION`. The original four columns were then dropped.
3. **Type conversion:** Numeric columns such as `OUTAGE.DURATION`, `CUSTOMERS.AFFECTED`, and `DEMAND.LOSS.MW` were coerced to proper numeric types.
4. **Alaska removal:** Alaska was dropped because the dataset specifies continental U.S. only, and Alaska's extreme urban-rural density ratio (4,506×) would distort the analysis.
5. **Feature engineering:** An urban-rural **density gap** was created (`POPDEN_URBAN / POPDEN_RURAL`), measuring how concentrated a state's population is in urban cores versus rural areas. States were binned into terciles: Low Gap (more uniform population), Medium Gap, and High Gap (urban-rural divide).
6. **Missing value treatment:** `OUTAGE.DURATION` has 58 missing values (3.7%), `CUSTOMERS.AFFECTED` has 443 (28.9%), and `DEMAND.LOSS.MW` has 705 (46.0%). These were left as `NaN`; missingness is explored in Step 3.

### Univariate Analysis

- The distribution of `OUTAGE.DURATION` is heavily right-skewed: the median is 11.7 hours, but the mean is 43.8 hours, with a maximum of 75.5 days.
- **Severe weather** accounts for roughly half of all outages (763 of 1,533 events), followed by intentional attacks (418) and system operability disruptions (127).

### Bivariate Analysis

- A scatter plot of the urban-rural density gap vs. outage duration (colored by cause) reveals that outages in high density-gap states tend to cluster at shorter durations, while low-gap states spread across both short and long durations.
- A box plot of customers affected by density tier shows that low-gap states affect more customers per event (median ~79K) while high-gap states affect far fewer (median ~4.9K).

### Interesting Aggregates

| Cause ↓ \ Density Gap → | Low Gap (more uniform) | Medium Gap | High Gap (urban-rural divide) |
|---|---|---|---|
| Severe Weather | 38.7 hrs | 41.9 hrs | 42.0 hrs |
| Intentional Attack | 1.5 hrs | 0.3 hrs | 1.2 hrs |
| Fuel Supply Emergency | 132.8 hrs | 48.0 hrs | 312.0 hrs |
| Equipment Failure | 3.0 hrs | 3.6 hrs | 7.5 hrs |
| System Disruption | 3.6 hrs | 3.7 hrs | 3.3 hrs |
| Public Appeal | 18.2 hrs | 8.3 hrs | 5.0 hrs |
| Islanding | 3.2 hrs | 0.5 hrs | 1.3 hrs |

*(Median outage duration in hours by cause category and density gap tier)*

The pivot table reveals that severe weather duration is fairly consistent across density tiers (~39–42 hrs), but fuel supply emergencies vary wildly — lasting over 300 hours in high-gap states. Public appeal outages show the clearest trend: longer in low-gap states (18 hrs) and shorter in high-gap states (5 hrs), suggesting concentrated populations are easier to manage during voluntary conservation events.

---

## Assessment of Missingness

### NMAR Analysis

`CUSTOMERS.AFFECTED` (28.9% missing) is likely **NMAR** — the value tends to be missing *because* the true customer impact is small. Events with negligible customer impact (especially intentional attacks like minor vandalism) simply don't get that field recorded by utilities. The missingness depends on the unobserved value itself: if the true number of affected customers is very low, there's less incentive or requirement to formally assess and report it.

Additional data that could make this MAR: utility company reporting thresholds or DOE filing requirements that specify when customer counts must be included.

### Missingness Dependency

Permutation tests were conducted on the missingness of `CUSTOMERS.AFFECTED`:

- **Depends on `CAUSE.CATEGORY`** (TVD = 0.558, p = 0.0): The distribution of cause categories looks completely different between missing and non-missing rows. Intentional attacks are massively overrepresented in the missing group — these small-scale vandalism events rarely get formal customer impact assessments.

- **Does NOT depend on `COM.PERCEN`** (p = 0.985): The share of commercial electricity consumption in a state has no relationship to whether customer counts are reported. The distributions when missing vs. not missing are nearly identical.

---

## Hypothesis Testing

**Question:** Do states with a high urban-rural density gap have significantly shorter outages than states with a low density gap?

- **Null hypothesis:** The mean outage duration is the same for high density-gap states and low density-gap states. Any observed difference is due to random chance.
- **Alternative hypothesis:** Low density-gap states (more uniform population) have a higher mean outage duration than high density-gap states.
- **Test statistic:** Difference in group means (low gap mean − high gap mean).
- **Significance level:** α = 0.05

**Result:** The observed difference was 906 minutes (15.1 hours), with a p-value of 0.002. We reject the null hypothesis — states with a more uniform population distribution experience significantly longer outages. This suggests that concentrated urban populations enable faster restoration, while spread-out populations require crews to cover wider areas, extending recovery time.

---

## Framing a Prediction Problem

**Prediction task:** Given characteristics known at the time of an outage, can we predict whether it will be **severe** (lasting 48+ hours, the top 25% of duration)?

This is a **binary classification** problem.

- **Target variable:** `SEVERE` (1 if duration ≥ 2,880 minutes, 0 otherwise)
- **Evaluation metric:** F1 score — chosen because the classes are imbalanced (75% not severe, 25% severe). Accuracy alone would reward a model that always predicts "not severe" (75% accuracy). F1 balances precision and recall for the minority class.
- **No data leakage:** All features used are known at the time of the outage — no restoration time, duration, or customer impact information is included as a feature.

---

## Baseline Model

**Algorithm:** Decision Tree Classifier

**Features (4):**
- `CAUSE.CATEGORY` (nominal) — one-hot encoded
- `CLIMATE.CATEGORY` (nominal) — one-hot encoded
- `NERC.REGION` (nominal) — one-hot encoded
- `MONTH` (ordinal) — standardized

These are deliberately basic: the three categorical features capture the what, where, and when of the outage at a high level, with no engineered features or hyperparameter tuning.

**Results:**
- Accuracy: 0.744
- F1 Score: 0.443

The baseline barely outperforms the naive strategy of always predicting "not severe" (75.4% accuracy). It identifies severe outages at only 44% recall, meaning it misses more than half of them.

---

## Final Model

**Algorithm:** Random Forest Classifier with GridSearchCV (5-fold cross-validation)

**Features (11):** All baseline features plus:
- `ANOMALY.LEVEL` (numeric) — El Niño/La Niña index; extreme climate episodes drive severe weather
- `POPULATION` (numeric) — larger states have different grid resilience characteristics
- `POPPCT_URBAN` (numeric) — urban vs. rural infrastructure differences
- `TOTAL.CUSTOMERS` (numeric) — grid size as a proxy for utility resources
- `TOTAL.PRICE` (numeric) — electricity market conditions
- `PC.REALGSP.STATE` (numeric) — state wealth may affect grid investment and restoration speed
- `DENSITY_GAP` (numeric, engineered) — our urban-rural density ratio from EDA, which showed a significant relationship with outage duration

**Best hyperparameters:** `max_depth=20`, `min_samples_split=2`, `n_estimators=200`

**Results:**
- Accuracy: 0.827
- F1 Score: 0.536

The final model improves F1 by **0.09** and accuracy by **8 percentage points** over the baseline. The improvement comes from both the additional features (especially the density gap and climate features that our EDA identified as meaningful) and the Random Forest's ability to capture nonlinear interactions between features. The deep trees (`max_depth=20`) suggest severity is driven by complex feature interactions rather than simple rules.

---

## Fairness Analysis

**Question:** Does the model perform equally across climate categories? Specifically, is the model fair for outages occurring in **cold** vs. **warm** climate conditions?

This matters because if the model underperforms in one climate category, utilities in those regions would receive less accurate severity predictions — an equity concern for emergency planning.

- **Null hypothesis:** The model's F1 score is the same for cold and warm climate regions. Any observed difference is due to random chance.
- **Alternative hypothesis:** The model's F1 score differs between cold and warm climate regions.
- **Test statistic:** Difference in F1 scores (cold − warm)
- **Significance level:** α = 0.05

**Result:**
- F1 (cold climate): 0.621
- F1 (warm climate): 0.615
- Observed difference: 0.005
- Permutation test p-value: 0.999

**Conclusion:** We fail to reject the null hypothesis. The model performs virtually identically across cold and warm climate categories (F1 difference of just 0.005, p = 0.999). There is no evidence of unfairness — the model is equally useful for predicting outage severity regardless of climate conditions.
