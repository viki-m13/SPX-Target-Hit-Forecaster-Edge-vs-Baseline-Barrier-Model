**SPX Target-Hit Forecaster — Edge vs Baseline Barrier Model**

This model is a **barrier-hitting forecaster** for SPX (^GSPC). It predicts when the index is likely to **rally at least a fixed percentage (e.g. +3%) within a future window (e.g. 10 trading days)** and only fires in favorable regimes.

The engine:

* Uses daily **high/low/close** prices from yfinance.
* Builds **labels** based on whether the **future max high** hits a target level (e.g. +3% above today’s close within N days).
* Uses a **regime filter** combining:

  * **Convex momentum** (quadratic trend + curvature in log price),
  * **ATR-based target sizing**,
  * **30-day realized volatility**,
  * **RSI band** (e.g. 40–70),
  * **Position in 60-day range**.
* Tunes **(lambda_convex, ATR_MULT, VOL_MIN)** via **walk-forward CV** (no leakage), optimizing **precision** and then **coverage**.

---

**Baseline vs Model (edge definition)**

* **Baseline**
  The baseline is the **unconditional probability** that SPX hits the target (e.g. **+3% within 10 trading days**) starting from any day, with **no filters**:

  > baseline = P( hit +3% within 10d | all days )

* **Model**
  The model only makes a prediction when its regime filter turns on (`predict_flag = True`). The **model hit rate** is:

  > model = P( hit target (≥ +3%) | model fires )

* **Lift / Edge vs baseline**
  The **edge** is the **absolute lift in hit rate** over the baseline:

  > **lift_vs_baseline = model − baseline**

  This tells you how much better the **conditional regime-selected days** are than **random days**, for the exact same target (+3% / 10 days).

---

**Example run — +3% target within 10 trading days**

* **Baseline:** P(hit +3.0% within 10d | all days) = **25.33%**
* **Model:** P(hit target (≥ 3.0%) | model fires) = **46.25%**
* **Coverage:** fraction of days with prediction = **6.34%**
* **Lift vs baseline (absolute):** **+20.92 percentage points**

Interpretation:

* If you blindly assume **“SPX will hit +3% within 10 days”** starting from any random day, you’d be right about **25%** of the time.
* If you only make that call on days when this model fires, you’re right about **46%** of the time.
* That’s an **edge of ~21 percentage points in hit probability**, with the model firing on **~6% of days**, i.e. a **selective high-conviction signal** rather than a daily forecast.
