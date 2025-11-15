**SPX Multi-Horizon Target-Hit Forecaster — Lift vs Baseline Barrier Model**

This model is a **barrier-hitting forecaster for SPX (^GSPC)** that searches over **horizon, minimum move and regime filters** to **maximize edge vs a naive baseline**.

Core ideas:

* For each day it asks:

  > *“Will SPX’s high reach at least X% above today’s close within the next N trading days?”*
* It builds **labels** from the *future max high* over `[t+1, …, t+N]`.
* It only fires in **favorable regimes** based on:

  * **Convex momentum** (quadratic trend + curvature in log price),
  * **ATR-based target sizing**,
  * **30-day realized volatility**,
  * **RSI band** (40–70),
  * **Position in 60-day range**.
* It uses **walk-forward CV** (no leakage) to pick the best combo of:

  * `horizon_days`, `min_move`, `lambda_convex`, `ATR_MULT`, `VOL_MIN`
    by **maximizing lift vs baseline** with a minimum coverage guard.

---

### Baseline vs Model (definition of “edge”)

* **Baseline**
  Unconditional probability that the move happens, no filters:

  > **Baseline:** ( P(\text{hit } +\text{min_move} \text{ within } N\text{d} \mid \text{all days}) )

* **Model**
  Probability the move happens **on days where the model actually fires** (`predict_flag = True`):

  > **Model:** ( P(\text{hit target} \ (\ge \text{min_move}) \mid \text{model fires}) )

* **Lift / Edge vs Baseline**
  The **edge** is the **absolute improvement in hit rate** over the baseline:

  > **Lift vs baseline:** ( \text{model} - \text{baseline} )

This is the cleanest way to quantify whether the regime filter is **actually concentrating probability mass** versus just cherry-picking a few lucky points.

---

### Example configuration A — **+3% target within 10 trading days**

* **Target:** SPX high reaches **≥ +3.0%** within **10 trading days**
* **Baseline hit rate:**
  **25.33%** — on a random day, SPX hits +3% within 10 days about 1 in 4 times.
* **Model hit rate (when it fires):**
  **46.25%**
* **Coverage:**
  **6.34%** of days have a live signal.
* **Lift vs baseline (absolute):**
  **+20.92 percentage points**

**Interpretation:**
The model more or less **doubles the hit probability** (25% → 46%) on a **small subset (~6%) of high-conviction days**, instead of making a prediction every day.

---

### Example configuration B — **+2% target within 5 trading days**

* **Target:** SPX high reaches **≥ +2.0%** within **5 trading days**
* **Baseline hit rate:**
  **25.58%** — randomly, SPX hits +2% within 5 days about 1 in 4 times.
* **Model hit rate (when it fires):**
  **49.01%**
* **Coverage:**
  **6.34%** of days have a live signal.
* **Lift vs baseline (absolute):**
  **+23.44 percentage points**

**Interpretation:**
For shorter horizon / smaller move (+2% in 5 days), the model still fires on about **6% of days**, but these days have roughly **double the baseline hit rate** (26% → 49%), giving an **edge of ~23 percentage points** in target-hit probability.
