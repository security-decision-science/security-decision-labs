# FAIR Risk Quantification – Model Reference

This document explains the core math and assumptions used by the FAIR Risk Quantification Tool.  
The goal is **transparency**: no hidden scores, no opaque multipliers.

---

## 1. Core Variables and Units

All calculations are built from three main parts:

- **TEF** – Threat Event Frequency  
  - Unit: `events per year`  
  - Interpretation: how many times per year relevant threat events **reach the asset** and attempt the described adverse outcome.
  - Input as percentiles: **P10, P50, P90**.

- **Susceptibility**  
  - Unit: `%` (0–100)  
  - Interpretation: conditional probability that a threat event **succeeds in causing the primary loss event**, given that it occurs.  
  - Input as percentiles: **P10, P50, P90**.

- **Loss Magnitude (LM)** – per-event loss  
  - Unit: currency per loss event (e.g. USD/event).  
  - Decomposed into **six loss forms**:
    - Productivity
    - Response
    - Replacement
    - Fines & Judgments
    - Competitive Advantage
    - Reputation  
  - Each loss form is modeled as a distribution over currency amounts.

- **SLEF – Secondary Loss Event Frequency**  
  - Unit: `%` (0–100)  
  - Interpretation: probability that **secondary losses occur**, *given that the primary loss event has occurred*.
  - Input as percentiles: **P10, P50, P90**.

From these we derive:

- **LEF – Loss Event Frequency**  
  - Unit: `events per year`  
  - Interpretation: expected number of **actual loss events** per year.

- **ALE – Annual Loss Expectancy**  
  - Unit: currency per year (e.g. USD/year)  
  - Interpretation: expected annual loss for the scenario.

---

## 2. Core Equations

### 2.1 From TEF and Susceptibility to LEF

Let's use the median values as the intuitive explanation; the simulation uses full distributions.

- **Median LEF:**

$$
\text{LEF}_{50} = \text{TEF}_{50} \times \frac{\text{Susceptibility}_{50}}{100}$$

- **In the Monte Carlo engine**, for each simulation \( i \):

$$
\text{LEF}_i = \text{TEF}_i \times \frac{\text{Susceptibility}_i}{100}$$

Where:

- $\text{TEF}_i \ge 0$
- $0 \le \text{Susceptibility}_i \le 100$

This guarantees $\text{LEF}_i \ge 0$.

---

### 2.2 Primary & Secondary Loss Magnitude

Per loss event, we compute:

- **Primary loss** (always evaluated for each event):

$$
\text{Primary}_i = 
  \text{Productivity}_i +
  \text{Response}_i +
  \text{Replacement}_i
$$

- **Secondary loss** (only realized when secondary losses occur):

$$
\text{Secondary}_i =
  \text{Fines}_i +
  \text{CompetitiveAdv}_i +
  \text{Reputation}_i
$$

All loss-form samples are by construction **non-negative**:

$$
\text{Productivity}_i, \dots, \text{Reputation}_i \ge 0
$$

---

### 2.3 SLEF and Secondary Loss Gating

SLEF is the **conditional probability** that secondary loss occurs, given that the primary loss event has occurred.

We sample SLEF as a percentage:

$$
0 \le \text{SLEF}_i \le 100
$$

Then convert to a probability:

$$
p_{\text{SLEF}, i} = \frac{\text{SLEF}_i}{100}
$$

The effective Single Loss Event Exposure (i.e. LM per event) is:

$$
\text{LM}_i = \text{Primary}_i + \text{Secondary}_i \times p_{\text{SLEF}, i}
$$

Special cases:

- If **SLEF = 0**  
  then $p_{\text{SLEF}, i} = 0$  
  and secondary terms drop out; only Primary contributes.
- If **there are no secondary losses** (all fines/competitive/reputation medians are zero), SLEF is not required and may be set to 0.

Because SLEF is conditional on the primary event and bounded in $[0,100]\%$, it **cannot cause a probability inconsistency** (secondary losses never exceed the probability of the precipitating primary loss event).

---

### 2.4 From LEF and LM to ALE

For each simulation $i$:

$$
\text{ALE}_i = \text{LEF}_i \times \text{LM}_i
$$

- Units: `(events/year) × (currency/event) = currency/year`
- Non-negativity: both factors are non-negative, so ALE is non-negative.

From the sampled ALE distribution we then compute summary statistics:

- Mean (expected annual loss)
- Percentiles (P10, P50, P90, P95, P99)

---

## 3. Distributions & Constraints

### 3.1 TEF

- Modeled as:
  - Poisson, optionally Zero-Inflated Poisson, or
  - Lognormal (for over-dispersed counts).
- Inputs: P10, P50, P90 (all ≥ 0).
- Constraint: **TEF ≥ 0**.

### 3.2 Susceptibility & SLEF

- Modeled via Beta-PERT on the range [0, 100].
- Inputs: P10, P50, P90 with:
  - $0 \le \text{P10} \le \text{P50} \le \text{P90} \le 100$
- These constraints guarantee that probabilities remain within valid bounds.

### 3.3 Loss Forms

- Each loss form is modeled as either:
  - Lognormal, or
  - Zero-inflated lognormal when P10 = 0 < P50.
- Inputs: P10, P50, P90 with P10 ≤ P50 ≤ P90 and all ≥ 0.
- When P50 = 0, the loss form is treated as always 0.

The zero-inflated path ensures we can represent “often zero, sometimes large” behavior without generating absurdly heavy tails.

---

## 4. No Hidden Scores

This engine **does not** use proprietary scores or arbitrary multipliers.

- There is **no risk score to money** magic.
- Everything is derived from:
  - TEF (events/year)
  - Susceptibility (%)
  - SLEF (%)
  - Loss Magnitude (currency/event across six explicit loss forms)

Monte Carlo simulation simply propagates the uncertainty in these inputs to distributions over LEF, LM, and ALE.

---

## 5. Versioning

The model is versioned at the math/engine level. Examples of changes:

- **v0.1** – Basic FAIR engine: TEF, Susceptibility, Primary/Secondary losses, SLEF, simple lognormals.
- **v0.2** – Added Zero-Inflated Poisson TEF option.
- **v0.3** – Added zero-inflated loss forms when P10 = 0; clarified LM vs SLEE naming.
- **v0.4** – Documented constraints; added unit tests for SLEF and Susceptibility edge cases.

Future changes to the math will be recorded here.
